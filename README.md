# Google Bridge

**Let your AI assistant create and edit real Google Docs, without a server, without per-user API keys, and without paying another platform.**

Google Bridge is a GitHub repository that works like a mailbox between an AI assistant (Claude, or any AI that can write to GitHub) and Google Workspace. The assistant drops a request in the mailbox, GitHub Actions does the work in Google Drive and Google Docs using one company credential, and drops the result back. Every operation is recorded in the git history: who asked for what, when, and what happened.

We built this to run a real content agency: AI-drafted articles land as Google Docs in client folders, client comments come back to the AI, and revisions go out the same way. It is the delivery layer of our content pipeline, in production with real clients. Now it is open for anyone to use.

## The problem this solves

Your team already uses an AI assistant on a subscription plan. But that assistant cannot deliver work where your business actually lives: Google Drive and Google Docs. Your current options:

1. **Pay an integration platform** for every action your AI takes.
2. **Host your own MCP server**, which means infrastructure, maintenance, and a developer.
3. **Give every employee Google API credentials**, which your IT team will (correctly) refuse.

Google Bridge is option 4: a repo you copy once, configure in about 20 minutes, and never host. GitHub Actions is the server. A single Google service account is the credential, stored as a repo secret the AI never sees.

## How it works

```
AI assistant                GitHub repo                  Google Workspace
    |                           |                              |
    |-- commit request.json --> |  requests/                   |
    |                           |-- Action triggers ---------> |
    |                           |   (service account)          |
    |                           | <------- API response ------ |
    |                           |  results/                    |
    | <-- read result.json ---- |                              |
```

1. The AI commits a JSON file to `requests/` (for example: "create a doc named X in folder Y with this content").
2. The push triggers the GitHub Action, which executes the operation against Google APIs using the service account key stored as a secret.
3. The Action commits the outcome to `results/` under the same filename.
4. The AI polls `results/` and reports back to you with the document link.

Round trip: about 60 to 90 seconds. Cost: zero (GitHub Actions free tier covers thousands of operations per month, and the service account is free).

## Operations

| Operation | What it does |
|---|---|
| `create_doc` | Creates a Google Doc from HTML in a Drive folder |
| `update_doc` | Replaces the full body of an existing Doc |
| `replace_text` | Surgical find and replace (preserves comments and formatting) |
| `read_doc` | Exports the Doc body as plain text |
| `copy_doc` | Copies a Doc to another folder (approved-draft handoff) |
| `create_folder` | Creates a Drive folder |
| `list_comments` | Reads all comments and replies on a Doc |
| `reply_comment` / `reply_comments` | Replies to comments, optionally resolving them |
| `generate_images` | Generates images with Gemini and uploads them to Drive (optional, needs `GEMINI_API_KEY`) |
| `insert_images` | Inserts images into a Doc at placeholder positions |

Request formats for every operation: [docs/operations.md](docs/operations.md).

## Setup (about 20 minutes, once)

**Requirements:** a GitHub account, a Google Workspace with a shared drive (the service account can only create files in shared drives), and an AI assistant that can call the GitHub API.

1. **Copy this repo.** Click **Use this template** (top right) and create your own private repository. The workflow comes with the copy and is active immediately. Your secrets stay yours: templates never copy secrets.
2. **Create a Google service account.** Follow [docs/setup-google-cloud.md](docs/setup-google-cloud.md). You will end up with a JSON key file. This is the only technical step, and the guide assumes zero Google Cloud experience.
3. **Add the secret.** In your new repo: Settings, Secrets and variables, Actions, New repository secret. Name: `GOOGLE_SA_KEY`. Value: the full content of the JSON key file.
4. **Share your Drive folder.** Share the shared-drive folder where you want documents created with the service account's email address (it looks like `name@project.iam.gserviceaccount.com`) as a Content manager.
5. **Connect your AI.** Follow [docs/claude-setup.md](docs/claude-setup.md): create a fine-grained GitHub token scoped to this one repo, and paste the starter instructions from [skill/SKILL.md](skill/SKILL.md) into your assistant.

Then ask your assistant for a document. It will show up in your Drive.

## Security model

- The Google credential lives only as a GitHub Actions secret. The AI never sees it, and it never appears in the chat, the repo files, or the git history.
- The AI's GitHub token is fine-grained and scoped to this single repository. Worst case if leaked: someone can file doc requests in your bridge, all of them logged in git.
- The workflow only executes its fixed set of operations. Request files are data, never code: nothing in a request can run commands, reach other APIs, or touch other repos.
- One shared token is fine for most teams. If you ever want per-person attribution, tokens are cheap: issue one per person and the git history does the rest. Optional, not required.
- The service account only reaches the folders you explicitly share with it. Nothing else in your Drive is visible.
- Every operation is a commit: full audit trail for free.
- The workflow never interpolates untrusted input into shell commands.

## FAQ

**What does it cost?** Nothing. One operation takes about a minute of GitHub Actions time, and the free tier gives private repos 2,000 minutes a month, so roughly 2,000 operations before you would pay a cent. A content team does not get close. Google service accounts are free. The optional image generation uses your own Gemini API key at Gemini prices.

**Does it work with assistants other than Claude?** Yes. Any AI that can make HTTP calls to the GitHub API (or commit to a repo) can use the bridge. The request format is plain JSON.

**Why not just use an MCP server?** MCP is great if you can host a server and manage OAuth for every user. The bridge exists for teams that cannot or do not want to host anything. It trades some latency (about a minute per operation) for zero infrastructure and a git audit trail.

**Can it create files in My Drive?** No. Google service accounts can only create files in shared drives. It can edit and comment on My Drive files that are shared with it.

**What is the catch?** Latency and scope. Each operation takes about 60 to 90 seconds because it rides on GitHub Actions, and operations run one at a time (serialized on purpose, so results never race). For a team drafting and delivering documents this is irrelevant; for high-volume automation or interactive editing it is not the right tool. And it covers Google Docs and Drive today: no Sheets or Slides operations yet.

## License

MIT. Built and battle-tested at [TJ Digital](https://github.com/TJ-Digital-LLC).
