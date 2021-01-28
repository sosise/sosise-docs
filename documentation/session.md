# HTTP Session
## Introduction
Since HTTP driven applications are stateless, sessions provide a way to store information about the user across multiple requests. That user information is typically placed in a persistent store / backend that can be accessed from subsequent requests.

Sosise ships with a variety of session backends that are accessed through an expressive, unified API. Support for popular backends such as File, Memory, if you need more, please let me know.

## Configuration
Your application's session configuration file is stored at `src/config/session.ts`. Be sure to review the options available to you in this file. By default, Sosise is configured to use the file session driver, which will work well for many applications. 

## Retrieving All Session Data
If you would like to retrieve all the data in the session:

```typescript
import { Request, Response, NextFunction } from 'express';
import HttpResponse from 'sosise-core/build/Types/HttpResponse';

export default class IndexController {
    /**
     * Example method
     */
    public async index(request: Request, response: Response, next: NextFunction) {
        try {
            console.log(request.session);
        } catch (error) {
            next(error);
        }
    }
}
```

## Determining If An Item Exists In The Session
To determine if an item is present in the session:

```typescript
if (request.session['users']) {
    console.log(request.session['users']);
}
```

## Storing Data
To store data in the session:

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

## Deleting Data
To remove a piece of data from the session:

```typescript
request.session['users'] = null;
```

## Regenerating The Session ID
Regenerating the session ID is often done in order to prevent malicious users from exploiting a session fixation attack on your application.

```typescript
request.session.regenerate(() => {
    return response.send('Session was regenerated');
});
```