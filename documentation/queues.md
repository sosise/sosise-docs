# Queues
## Introduction
While building your web application, you may have some tasks, such as parsing and storing an uploaded CSV file, that take too long to perform during a typical web request. Thankfully, Sosise allows you to easily create queued jobs that may be processed in the background. By moving time intensive tasks to a queue.

Sosise's queue configuration options are stored in your `src/config/queue.ts` configuration file. In this file, you will find connection configurations for redis.

> Please refer to docker-compose.yml if you need redis with persistent or non persistent storage.

## How it works
Queues consist of two components `Jobs` and `QueueWorkers`. A `job` can be pushed to some `queue` and a `queue` is then handeled by a worker.

For example you may push a send email job to a `email` queue, which then will be handeled by a worker which listens for `email` queue.

## How to put a job into `Queue`
Simple example:
```typescript
import Queue from 'sosise-core/build/Queue/Queue';
await Queue.add('email-processing-queue', { to: 'info@example.com' });
```
In this example a job will be added to `email-processing-queue` queue with some payload.

Second example:
```typescript
import Queue from 'sosise-core/build/Queue/Queue';
await Queue.add('email-processing-queue', { to: 'info@example.com' }, { delay: 1000, attempts: 3, backoff: { delay: 300, type: 'exponential' } });
```

In this example a job will be added to `email-processing-queue` queue with some payload. The job will be executed after a `1000 ms` delay. If it fails it will be retried `3 times` with exponential delay of `300 ms`, `600 ms`, `1200 ms`.

## How to handle a job in a queue
To handle a job in a queue, you'll have to create a queue worker:

```sh
./artisan make:queueworker SendEmailWorker
```

Self speaking example:
```typescript
import { Worker, QueueScheduler, Job } from 'bullmq';
import queueConfig from '../../../config/queue';
import Handler from '../../Exceptions/Handler';

export default class SendEmailWorker {
    /**
     * Queue name to listen
     */
    public queueName: string = 'email-processing-queue';

    /**
     * Redis connection data
     */
    private redisConnection: { host: string, port: number };

    /**
     * Constructor
     */
    constructor() {
        this.redisConnection = { host: queueConfig.redis.host, port: queueConfig.redis.port };
    }

    /**
     * Handle incoming job
     */
    private async handle(job: Job): Promise<void> {
        console.log('job received', job.id, job.data, job.attemptsMade);
    }

    /**
     * Listen queue, please do not delete this method
     */
    private async listen(): Promise<void> {
        // Instantiate queue scheduler
        const scheduler = new QueueScheduler(this.queueName, { connection: this.redisConnection });

        // Instantiate worker
        const myWorker = new Worker(this.queueName, async (job) => this.handle(job), { connection: this.redisConnection });

        // Report job exceptions
        myWorker.on('failed', async (job: Job, error) => {
            // Some exception occured during job handling
            // Report the exception
            const exceptionHandler = new Handler();
            exceptionHandler.reportCommandException(error);
        });
    }
}
```

## How to run the queue worker
```sh
./artisan queue:listen email-processing-queue
```

## List all jobs in a queue
```sh
./artisan queue:list
```

## Retry all failed jobs
```sh
./artisan queue:retry
```

## Flush (remove) all failed jobs
```sh
./artisan queue:flush
```