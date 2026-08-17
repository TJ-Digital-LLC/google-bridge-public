# Operations reference

Every request is one JSON file committed to `requests/`. The filename must be unique (use a timestamp plus a slug, like `2026-08-17-1432-create-report.json`). The result appears in `results/` under the same filename.

Every result includes `"operation"` and `"status"` (`ok`, `partial`, or `error`). Errors include an `"error"` message and, for API failures, an `"http_status"`.

## create_doc

Creates a Google Doc from HTML in a Drive folder. HTML gives you headings, bold, lists, links and tables for free.

```json
{
  "operation": "create_doc",
  "name": "My document title",
  "folder_id": "<Drive folder ID>",
  "html_base64": "<base64 of the HTML body>"
}
```

Result: `{"status": "ok", "id": "...", "webViewLink": "...", "name": "..."}`

## update_doc

Replaces the entire body of an existing Doc. Use only for docs that have no comments yet; for reviewed docs use `replace_text` so comments survive.

```json
{
  "operation": "update_doc",
  "doc_id": "<Doc ID>",
  "html_base64": "<base64 of the new HTML body>"
}
```

## replace_text

Surgical find and replace via the Docs API. Preserves comments and formatting. Match is exact and case sensitive.

```json
{
  "operation": "replace_text",
  "doc_id": "<Doc ID>",
  "replacements": [
    {"find": "<exact existing text>", "replace": "<new text>"}
  ]
}
```

Result reports `occurrences_changed` per replacement. Zero means the exact text was not found: read the doc first and copy the text verbatim.

## read_doc

Exports the Doc body as plain text, base64-encoded in the result. Use it to see the current state before editing.

```json
{"operation": "read_doc", "doc_id": "<Doc ID>"}
```

## copy_doc

Copies a Doc, optionally to another folder and with a new name. Typical use: moving an approved draft from a working folder to the client-facing folder. The destination must be in a shared drive.

```json
{
  "operation": "copy_doc",
  "doc_id": "<Doc ID>",
  "dest_folder_id": "<folder ID, optional>",
  "name": "<new name, optional>"
}
```

## create_folder

```json
{
  "operation": "create_folder",
  "name": "New folder name",
  "parent_id": "<parent folder ID>"
}
```

## list_comments

Reads up to 100 comments on a Doc, with authors, quoted text, resolution state, and replies.

```json
{"operation": "list_comments", "doc_id": "<Doc ID>"}
```

## reply_comment / reply_comments

Reply to one comment, or to several in a batch, optionally resolving them.

```json
{
  "operation": "reply_comments",
  "doc_id": "<Doc ID>",
  "replies": [
    {"comment_id": "<id>", "reply_text": "Done, updated the intro.", "resolve": true}
  ]
}
```

## generate_images (optional, needs GEMINI_API_KEY secret)

Generates up to 8 images with Gemini and uploads them to a Drive folder. Each image gets a `direct_url` usable by `insert_images`.

**Privacy note:** by default each image is set to "anyone with the link can view" (`"link_readable": true`). This is required for `insert_images`, because the Docs API fetches the image from a public URL. If you do not need doc insertion, pass `"link_readable": false` and the images stay private to the shared folder.

```json
{
  "operation": "generate_images",
  "folder_id": "<Drive folder ID>",
  "subfolder": "my-article",
  "slug": "my-article",
  "prompts": [
    {"name": "hero", "prompt": "<image prompt>", "aspect_ratio": "16:9"}
  ]
}
```

## insert_images

Replaces placeholder strings in a Doc with inline images (up to 12 per request). Put placeholders like `[IMAGE: hero]` in the doc body when creating it, then map them.

```json
{
  "operation": "insert_images",
  "doc_id": "<Doc ID>",
  "mappings": [
    {"placeholder": "[IMAGE: hero]", "image_url": "<direct_url from generate_images>"}
  ]
}
```
