# n8n IMAP Inbox Organiser

An n8n workflow that automatically scans your email inbox, groups emails by sender domain, creates folders for each group, and moves emails into them — giving you a clean, organised inbox you can review and bulk-delete with ease.

## What it does

- Connects to any IMAP-compatible email provider
- Fetches emails from your INBOX
- Extracts the sender domain from each email
- Groups emails by domain (e.g. amazon.es, linkedin.com, paypal.co.uk)
- Creates a folder structure under `Organised/` for each domain group
- Moves all emails into their respective folders
- Falls back to COPY + DELETE if your mail server does not support the IMAP MOVE command

## Workflow structure

```
Manual Trigger
    └── Code node: Fetch emails via IMAP
        └── Code node: Extract sender domain
            └── Code node: Group emails by domain
                └── Loop Over Items (batch size 1)
                    └── Code node: Create IMAP folder
                        └── Code node: Move emails to folder
                            └── (back to loop)
```

## Requirements

- n8n (self-hosted, community edition or above)
- Any IMAP-compatible email account
- The following npm packages enabled via environment variable:

```
NODE_FUNCTION_ALLOW_EXTERNAL=imap,mailparser
```

Add this to your `.env` file or via your server management panel (e.g. EasyPanel) and restart n8n.

## Setup

### 1. Clone or import the workflow

Import the workflow JSON into your n8n instance via **Workflows → Import**.

### 2. Configure your IMAP credentials

In each of the three Code nodes that connect to your mail server, update the following values:

```js
const imap = new Imap({
  user: 'you@yourdomain.com',
  password: 'yourpassword',
  host: 'imap.yourprovider.com',
  port: 993,
  tls: true,
  tlsOptions: { rejectUnauthorized: false }
});
```

Common IMAP host values:

| Provider | Host |
|---|---|
| Gmail | imap.gmail.com |
| Outlook / Hotmail | outlook.office365.com |
| Yahoo | imap.mail.yahoo.com |
| Fastmail | imap.fastmail.com |
| cPanel / custom domain | mail.yourdomain.com |

### 3. Adjust the fetch slice size

By default the workflow fetches the 50 most recent emails. Once you have confirmed it works correctly, increase this in the fetch Code node:

```js
const slice = uids.slice(0, 50); // increase to 200, 500, etc.
```

### 4. Run the workflow

Click **Execute workflow** in n8n. The workflow will:
- Fetch your emails
- Group them by domain
- Create `Organised/` subfolders in your mailbox
- Move emails into their respective folders

Check your email client — you should see the `Organised/` folder tree populating in real time.

## Folder structure created in your mailbox

```
Organised/
    ├── amazon.es/
    ├── linkedin.com/
    ├── paypal.co.uk/
    ├── newsletter.domain.com/
    └── ...
```

## Notes

- The workflow is triggered **manually** — it does not run automatically in the background. Run it whenever your inbox needs organising.
- Emails are **moved**, not copied — your INBOX will be cleared as folders are populated.
- If a folder already exists, the workflow skips creation and moves emails directly.
- To process your full inbox, increase the slice size gradually and monitor for timeouts. If you hit runner timeout errors, add the following to your environment: `N8N_RUNNERS_HEARTBEAT_INTERVAL=60`

## Built with

- [n8n](https://n8n.io) — workflow automation
- [node-imap](https://github.com/mscdex/node-imap) — IMAP client for Node.js
- [mailparser](https://nodemailer.com/extras/mailparser/) — email header parsing
- Claude.ai - AI supported me with structure and coding

## License

MIT
