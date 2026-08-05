# HTTP Server

HTTP server limits are configurable in `sosise-core` 2.0.1 and later, transport
middlewares and graceful shutdown in 2.1.0 and later. They are read directly
from the environment when the application starts.

## Upgrade

```bash
npm install sosise-core@2.1.0
npm ls sosise-core --depth=0
```

Existing lockfiles remain on the previously resolved version until the
dependency is explicitly updated.

## Configuration

```dotenv
HTTP_JSON_BODY_LIMIT=10mb
HTTP_RAW_BODY_LIMIT=150mb
HTTP_REQUEST_TIMEOUT_MS=300000
HTTP_HEADERS_TIMEOUT_MS=60000
HTTP_SOCKET_TIMEOUT_MS=0
HTTP_SHUTDOWN_TIMEOUT_MS=10000

HTTP_COMPRESSION=true
HTTP_JSON_PARSER=true
HTTP_URLENCODED_PARSER=true
HTTP_RAW_PARSER=true
```

`HTTP_JSON_BODY_LIMIT`
: Maximum parsed JSON and URL-encoded request body size. The starter value is
  `10mb`.

`HTTP_RAW_BODY_LIMIT`
: Maximum raw request body size. The starter value is `150mb`.

`HTTP_REQUEST_TIMEOUT_MS`
: Time allowed to receive the complete incoming request. The starter value is
  `300000`.

`HTTP_HEADERS_TIMEOUT_MS`
: Time allowed to receive complete request headers. The starter value is
  `60000`.

`HTTP_SOCKET_TIMEOUT_MS`
: Established socket inactivity timeout. The starter value is `0`.

`HTTP_SHUTDOWN_TIMEOUT_MS`
: Grace period granted to in-flight requests during shutdown. The starter value
  is `10000`.

`HTTP_COMPRESSION`
: Registers global gzip response compression. The starter value is `true`.

`HTTP_JSON_PARSER`
: Registers the global `application/json` body parser. The starter value is
  `true`.

`HTTP_URLENCODED_PARSER`
: Registers the global `application/x-www-form-urlencoded` body parser. The
  starter value is `true`.

`HTTP_RAW_PARSER`
: Registers the global raw body parser. The starter value is `true`.

Middleware switches accept only `true` or `false`. Any other value, including a
blank one, fails startup validation rather than silently disabling a middleware
because of a typo.

Timeout values are milliseconds without units. Request and header timeouts
must be positive safe integers. Socket timeout may be `0`, which disables the
established-socket inactivity timeout. The header timeout cannot exceed the
request timeout.

When a timeout variable is omitted, Sosise preserves the default supplied by
the running Node.js version. A blank timeout variable is not the same as an
omitted variable and fails startup validation.

## Timeout Semantics

`HTTP_REQUEST_TIMEOUT_MS` limits how long Node waits to receive the request,
including its body. It does not impose a controller or service execution
deadline after the request has arrived.

`HTTP_HEADERS_TIMEOUT_MS` protects the server from clients that send request
headers too slowly. Node handles request and header timeout failures before
Express route handling and normally returns HTTP `408`.

`HTTP_SOCKET_TIMEOUT_MS` is an inactivity timeout for an established socket. A
nonzero value can terminate a silent long-running response even when the
application is still working. Use `0` only when a trusted reverse proxy and
endpoint-specific application deadlines provide the required protection.

For long-running work, prefer `202 Accepted` plus polling, a webhook, or a
queue. If a synchronous endpoint is required, align every timeout in the
request path:

```text
application deadline < reverse proxy response timeout < caller timeout
```

For SSE and other streaming responses, send heartbeats more frequently than
the smallest socket, proxy, or client inactivity timeout.

## Request Body Limit

`HTTP_JSON_BODY_LIMIT` uses Express size syntax such as `256kb`, `1mb`, or
`10mb`. It applies globally to:

- `application/json`
- `application/x-www-form-urlencoded`

It does not configure multipart file uploads. The raw-body parser has its own
limit, `HTTP_RAW_BODY_LIMIT`. Do not raise the JSON limit to support file
uploads; use streaming multipart handling or direct object storage instead.

The effective default changed in `sosise-core` 2.0.1 from Express's implicit
`100kb` limit to the Sosise fallback of `10mb`. Keep the configured value as
small as legitimate traffic permits and enforce an equal or stricter limit at
the edge proxy. Sosise parses the body before dynamic application middleware,
so throttling middleware does not prevent the body from being read first.

Express parser errors such as `entity.too.large` are passed to the
application's exception handler. Ensure a custom
`src/app/Exceptions/Handler.ts` preserves safe middleware status codes if
clients must receive HTTP `413`; otherwise a generic fallback can convert the
error to `500`.

## Transport Middlewares

By default Sosise registers gzip compression and the JSON, URL-encoded and raw
body parsers globally, before your routes. Every one of those parsers consumes
the request stream, so by the time a controller runs, the stream is already
drained. That is the right default for a JSON API and the wrong one for a few
specific workloads:

- **Streamed uploads.** A route-level multipart handler such as `busboy` or
  `multer` receives an already consumed stream. Set `HTTP_RAW_PARSER=false`.
- **Request proxying.** Forwarding the untouched request body upstream requires
  the raw stream. Set `HTTP_RAW_PARSER=false`.
- **Server-Sent Events and streaming responses.** Compression buffers the
  response and delays or breaks event delivery. Set `HTTP_COMPRESSION=false`.

Disable only what conflicts with the workload and register the middleware you
still need on the individual routes that need it:

```dotenv
HTTP_RAW_PARSER=false
```

```typescript
import Express from 'express';

// Register the parser only where the route actually wants a parsed body
router.post('/webhook', Express.raw({ type: 'application/octet-stream' }), controller.handleWebhook);
```

Applications that do not set these variables keep the historical behaviour,
so upgrading changes nothing until a switch is flipped.

## Graceful Shutdown

`Server` exposes a lifecycle beyond `run()`:

`start()`
: Resolves once the server actually listens and returns the Node.js HTTP
  server. A port that cannot be acquired rejects the promise instead of raising
  an unhandled `error` event.

`stop()`
: Stops accepting new connections, releases idle keep-alive sockets
  immediately, waits up to `HTTP_SHUTDOWN_TIMEOUT_MS` for in-flight requests to
  finish, then destroys whatever is left. It also releases the session-store
  resources the server owns, such as the file-store reap timer, the memory-store
  interval and the Redis session client.

The skeleton wires this to the process termination signals:

```typescript
const server = new Server();

// Shut down gracefully when the process manager asks the application to terminate
for (const signal of ['SIGTERM', 'SIGINT'] as const) {
    process.on(signal, () => {
        server.stop().then(
            () => process.exit(0),
            (error) => {
                console.error(error);
                process.exit(1);
            },
        );
    });
}

// A startup failure must terminate the process instead of becoming an unhandled rejection
server.run().catch((error) => {
    console.error(error);
    process.exit(1);
});
```

Without the idle-connection release, a plain `close()` waits for every
keep-alive client, so a container with live traffic would only exit on the
orchestrator's kill timeout. Keep `HTTP_SHUTDOWN_TIMEOUT_MS` below the
termination grace period of your orchestrator, for example Kubernetes
`terminationGracePeriodSeconds` or the Docker stop timeout.

## Reverse Proxy

The effective behavior is always determined by the shortest timeout or
smallest body limit in the client, proxy, and application chain. A typical
Nginx policy is:

```nginx
client_max_body_size 1m;
client_header_timeout 30s;
client_body_timeout 120s;

location /api/ {
    proxy_pass http://sosise:10000;
    proxy_connect_timeout 5s;
    proxy_send_timeout 120s;
    proxy_read_timeout 300s;
}
```

Nginx timeout directives are not exact equivalents of Node settings. In
particular, `proxy_read_timeout` controls inactivity while waiting for upstream
response bytes. Increase it only for routes that legitimately remain silent
for a long time rather than weakening the global proxy policy.

## Troubleshooting

If startup fails, verify that timeout values are decimal integers, are not
blank, and satisfy
`HTTP_HEADERS_TIMEOUT_MS <= HTTP_REQUEST_TIMEOUT_MS`, and that every middleware
switch is spelled exactly `true` or `false`.

If a controller receives an empty request body while a route-level parser is
registered, a global parser has already consumed the stream. Disable the
conflicting global parser rather than reordering middleware.

If the process does not exit on `SIGTERM`, check that the signal handlers in
`src/server.ts` call `server.stop()` and that `HTTP_SHUTDOWN_TIMEOUT_MS` is
shorter than the orchestrator's termination grace period.

If a request disconnects while application work continues, inspect the client
timeout, reverse proxy `proxy_read_timeout`, and socket inactivity timeout.
Increasing `HTTP_REQUEST_TIMEOUT_MS` will not extend controller execution
because that setting only governs receiving the incoming request.
