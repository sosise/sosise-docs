# Mailer

## Introduction

Sending emails can be made easy with Sosise's clean and simple email sending library.

## Configuration

The mailer configuration options for Sosise are located in the `src/config/mailer.ts` configuration file. In this file, you will find the SMTP connection configuration.

> Sosise uses [Nodemailer](https://nodemailer.com/about/) library. For a full documentation reference, please visit the Nodemailer documentation.

## Sending Mail

You can send emails using the `Mailer` class provided by Sosise. The `sendMail` method allows you to send plain text and HTML emails.

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

> For a full list of all possible message options, please refer to [Mail.Options](https://nodemailer.com/message/).

## Sending Mail with Attachment

You can also send emails with attachments by including the `attachments` property in the options.

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

> For more information about attaching files, please refer to the [Attachments](https://nodemailer.com/message/attachments/) documentation.

## Send Fake Emails

Sosise, using Nodemailer, provides an option to send fake emails, which is useful during development and testing. To enable this feature, set the `dryrun` option in `src/config/mailer.ts` to `true`.

When this option is active, emails won't actually be sent, but instead, additional information will be logged to the terminal.

Example log output for a fake email:

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

Enabling this option allows you to test and verify your email functionality without actually sending emails to real recipients.