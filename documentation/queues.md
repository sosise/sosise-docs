# Queues

## Introduction

While building your web application, you may encounter time-intensive tasks, such as parsing and storing uploaded CSV files, that take too long to perform during a typical web request. Sosise allows you to easily create queued jobs that can be processed in the background, freeing up your web server to handle more requests. These queued jobs can be moved to a queue where they can be processed asynchronously.

Sosise's queue configuration options are stored in your `src/config/queue.ts` configuration file. In this file, you will find connection configurations for Redis.

> If you need Redis with persistent or non-persistent storage, please refer to `docker-compose.yml`.
> Sosise uses the [BullMQ](https://docs.bullmq.io) library for queue management. For detailed documentation, refer to the BullMQ documentation.

## How It Works

Queues consist of two components: `Jobs` and `QueueWorkers`. A `Job` can be pushed to a specific `Queue`, and a `QueueWorker` is responsible for handling jobs from that queue.

For example, you may push a "send email" job to an `email` queue, which will be handled by a worker specifically listening for the `email` queue.

## How to Put a Job into a Queue

Simple example:

```typescript
import Queue from 'sosise-core/build/Queue/Queue';

await Queue.add('email-processing-queue', { to: 'info@example.com' });
```

In this example, a job will be added to the `email-processing-queue` with the specified payload.

Second example:

```typescript
import Queue from 'sosise-core/build/Queue/Queue';

await Queue.add('email-processing-queue', { to: 'info@example.com' }, { delay: 1000, attempts: 3, backoff: { delay: 300, type: 'exponential' } });
```

In this example, a job will be added to the `email-processing-queue` with the specified payload. The job will be executed after a `1000 ms` delay. If it fails, it will be retried `3 times` with an exponential delay of `300 ms`, `600 ms`, and `1200 ms`.

## How to Handle a Job in a Queue

To handle a job in a queue, you need to create a queue worker:

```sh
./artisan make:queueworker SendEmailWorker
```

Example of a queue worker:

```typescript
import { Worker, QueueScheduler, Job } from 'bullmq';
import queueConfig from '../../../config/queue';
import Handler from '../../Exceptions/Handler';

export default class SendEmailWorker {
    // ... (constructor and other methods)

    /**
     * Handle incoming job
     */
    private async handle(job: Job): Promise<void> {
        console.log('job received', job.id, job.data, job.attemptsMade);
    }

    /**
     * Listen to the queue (please do not delete this method)
     */
    private async listen(): Promise<void> {
        // Instantiate queue scheduler
        const scheduler = new QueueScheduler(this.queueName, { connection: this.redisConnection });

        // Instantiate worker
        const myWorker = new Worker(this.queueName, async (job) => this.handle(job), { connection: this.redisConnection });

        // Report job exceptions
        myWorker.on('failed', async (job: Job, error) => {
            // Some exception occurred during job handling
            // Report the exception
            const exceptionHandler = new Handler();
            exceptionHandler.reportCommandException(error);
        });
    }
}
```

## How to Run the Queue Worker

```sh
./artisan queue:listen email-processing-queue
```

## List All Jobs in a Specific Queue

```sh
./artisan queue:list email-processing-queue
```

## Retry All Failed Jobs from a Specific Queue

```sh
./artisan queue:retry email-processing-queue
```

## Flush (Remove) All Failed Jobs from a Specific Queue

```sh
./artisan queue:flush email-processing-queue
```