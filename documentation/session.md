# HTTP Session Management

## Introduction
In HTTP-driven applications, where requests and responses are stateless, sessions provide a way to store information about the user across multiple requests. Sessions are essential for maintaining user-specific data and allowing users to be recognized across interactions with the application.

Sosise comes with a flexible and unified session API that supports various session backends. The session data is typically stored in a persistent store or backend that can be accessed and modified from subsequent requests.

## Configuration
The session configuration file for your application is located at `src/config/session.ts`. This file allows you to define the session driver and configure its options. By default, Sosise is set up to use the file session driver, which works well for many applications. However, you can switch to other supported backends, such as Memory, or request additional backends if needed.

> Note: Sessions are disabled by default. Make sure to enable them in the `src/config/session.ts` file before using them in your application.

## Retrieving All Session Data
To retrieve all the data stored in the session, access the `request.session` object. It contains the entire session data:

```typescript
import { Request, Response, NextFunction } from 'express';

export default class IndexController {
    public async index(request: Request, response: Response, next: NextFunction) {
        try {
            console.log(request.session);
        } catch (error) {
            next(error);
        }
    }
}
```

## Checking If an Item Exists in the Session
You can check if a specific item exists in the session using standard JavaScript syntax:

```typescript
if (request.session['users']) {
    console.log(request.session['users']);
}
```

## Storing Data in the Session
To store data in the session, simply assign the data to the appropriate session key:

```typescript
request.session['users'] = [
    {
        id: 1,
        name: 'Sharon'
    },
    {
        id: 2,
        name: 'Asif'
    }
];
```

## Deleting Data from the Session
If you want to remove a piece of data from the session, you can set it to `null`:

```typescript
request.session['users'] = null;
```

## Regenerating the Session ID
Regenerating the session ID is a useful security measure to prevent session fixation attacks on your application. You can do this by calling the `regenerate` method on the session object:

```typescript
request.session.regenerate(() => {
    return response.send('Session was regenerated');
});
```

By regenerating the session ID, you invalidate the current session and create a new one. This helps prevent attackers from exploiting session vulnerabilities.

## Conclusion
Session management is crucial for maintaining stateful behavior in HTTP applications. With Sosise's session API and the various session backends, you can easily manage user-specific data and enhance the security and functionality of your web application. Make sure to choose the appropriate session driver and configure it according to your application's needs in the `src/config/session.ts` file.