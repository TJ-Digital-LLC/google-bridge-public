# Google Bridge starter skill

Paste this whole file into your Claude Project instructions (or install it as a skill) and fill in the three config lines.

---

## Config

- BRIDGE_REPO: `{BRIDGE_REPO}` (example: `your-org/google-bridge`)
- GITHUB_TOKEN: `{GITHUB_TOKEN}` (fine-grained, Contents read/write, this repo only)
- DEFAULT_FOLDER_ID: `{DEFAULT_FOLDER_ID}` (Drive folder where docs land unless the user says otherwise)

## What you can do

You can create and edit real Google Docs for the user through the Google Bridge: a GitHub repo where you commit operation requests and a GitHub Action executes them in Google Drive/Docs. Available operations: create_doc, update_doc, replace_text, read_doc, copy_doc, create_folder, list_comments, reply_comment, reply_comments, generate_images, insert_images. Full request formats: `docs/operations.md` in the bridge repo.

## How to execute an operation

1. **Build the request JSON** for the operation (see formats below or fetch `docs/operations.md` from the repo).
2. **Pick a unique filename:** `YYYY-MM-DD-HHMM-<short-slug>.json` (current date and time, so retries never collide).
3. **Submit** with the GitHub Contents API:
   - `PUT https://api.github.com/repos/{BRIDGE_REPO}/contents/requests/<filename>`
   - Headers: `Authorization: Bearer {GITHUB_TOKEN}`, `Accept: application/vnd.github+json`
   - Body: `{"message": "bridge: <operation> <slug>", "content": "<base64 of the request JSON>"}`
4. **Poll for the result** in separate short calls: `GET https://api.github.com/repos/{BRIDGE_REPO}/contents/results/<filename>` with the same headers. A 404 means it is still running. Wait about 20 to 25 seconds between checks, up to 20 attempts (the Action usually takes 60 to 90 seconds).
5. **Decode the result** (`content` field is base64) and report to the user: on success, give them the `webViewLink` as a clickable link; on error, quote the error message and suggest the likely fix (folder not shared, wrong ID, exact text not found).

## Rules

- One operation per request file. Never overwrite an existing request.
- For `create_doc`, write clean semantic HTML (h1/h2, p, ul, strong, a). Base64-encode it.
- To edit a doc a human has touched, `read_doc` first, then `replace_text` with verbatim text. Never `update_doc` over a doc that has comments.
- If the user does not name a destination folder, use DEFAULT_FOLDER_ID.
- If polling runs out of attempts, tell the user to check the Actions tab of the bridge repo; do not silently retry the same operation.
- Never show GITHUB_TOKEN in your replies.

## Example: create a doc

Request file `2026-08-17-1500-hello.json`:

```json
{
  "operation": "create_doc",
  "name": "Bridge test",
  "folder_id": "{DEFAULT_FOLDER_ID}",
  "html_base64": "PGgxPkhlbGxvPC9oMT48cD5GaXJzdCBkb2MgdmlhIHRoZSBicmlkZ2UuPC9wPg=="
}
```

Expected result: `{"operation": "create_doc", "status": "ok", "id": "...", "webViewLink": "https://docs.google.com/..."}`
