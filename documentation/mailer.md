# Mailer
## Introduction
Sending email doesn't have to be complicated. Sosise provides a clean, simple email sending library.

Sosise's mailer configuration options are stored in your `src/config/mailer.ts` configuration file. In this file, you will find SMTP connection configuration.

> Sosise uses [Nodemailer](https://nodemailer.com/about/) library. Please refer to it for full documentation.

## Sending mail
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

When you have activated this option, all additional information will be logged to terminal.

```bash
2021-07-27 13:25:39 INFO  Mailer dryrun mode is active... No emails would be sent 
2021-07-27 13:25:39 INFO  Test SMTP account obtained successfully {
  user: 'l27iz4qt26x4b45z@ethereal.email',
  pass: '7nCJDgsjQqDrH7Eq1z',
  smtp: { host: 'smtp.ethereal.email', port: 587, secure: false },
  imap: { host: 'imap.ethereal.email', port: 993, secure: true },
  pop3: { host: 'pop3.ethereal.email', port: 995, secure: true },
  web: 'https://ethereal.email'
}
2021-07-27 13:25:39 INFO  Preparing to send fake email 
2021-07-27 13:25:40 INFO  Fake email was sent successfully { messageId: '<db335cd0-2f27-609b-da7c-5c576c34374e@server.com>' }
2021-07-27 13:25:40 INFO  Fake email link generated, https://ethereal.email/message/YP-08y0yk1T9bJDdYP-09efY8noJPhrkAAAAAW7nbvG1shb3n.ZXZvoR8tw 
```