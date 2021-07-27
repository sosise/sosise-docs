# Mailer
## Introduction
Sending email doesn't have to be complicated. Sosise provides a clean, simple email sending library.

Sosise's mailer configuration options are stored in your `src/config/mailer.ts` configuration file. In this file, you will find SMTP connection configuration.

> Sosise uses [Nodemailer](https://nodemailer.com/about/) library. Please refer to it for full documentation.

## Sending mail
Example
```typescript
import Mailer from 'sosise-core/build/Mailer/Mailer';
await Mailer.sendMail({
    from: "example@server.com",
    to: "billy@micr.com",
    subject: "Message title",
    text: "Plaintext version of the message",
    html: "<p>HTML version of the message</p>"
});
```

> Please refer to [Mail.Options](https://nodemailer.com/message/) to see a full list of all possible message options

## Sending mail with attachment
Example
```typescript
import Mailer from 'sosise-core/build/Mailer/Mailer';
await Mailer.sendMail({
    from: "example@server.com",
    to: "billy@micr.com",
    subject: "Message title",
    text: "Plaintext version of the message",
    html: "<p>HTML version of the message</p>",
    attachments: [
        {
            path: process.cwd() + '/hello.txt'
        },
    ]
});
```

> Please refer to [Attachments](https://nodemailer.com/message/attachments/) documentation to see all possible options how to attach files

## Send fake emails
Since `Sosise` uses `Nodemailer` there is a possibility to send fake emails, which can be used for development. Just set `dryrun` option in `src/config/mailer.ts` to `true`.