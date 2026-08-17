# Google Cloud setup: create the service account

This is the only technical step of the bridge setup. It takes about 10 minutes and you only do it once. No code involved, just clicking through the Google Cloud console.

A service account is a robot Google identity that belongs to your company, not to any person. The bridge uses it to create and edit documents. It can only touch the folders you explicitly share with it.

## 1. Create a project

1. Go to [console.cloud.google.com](https://console.cloud.google.com) and sign in with your Google Workspace admin account.
2. Click the project selector (top bar), then **New project**.
3. Name it something like `ai-docs-bridge` and click **Create**. Select it.

## 2. Enable the APIs

1. In the left menu: **APIs and Services**, then **Library**.
2. Search for **Google Drive API**, open it, click **Enable**.
3. Search for **Google Docs API**, open it, click **Enable**.

## 3. Create the service account

1. Left menu: **IAM and Admin**, then **Service accounts**.
2. Click **Create service account**.
3. Name: `docs-bot` (or anything you like). Skip the optional permission steps. Click **Done**.
4. You will see an email like `docs-bot@ai-docs-bridge.iam.gserviceaccount.com`. **Copy it, you will need it to share folders.**

## 4. Download the key

1. Click the service account you just created, then the **Keys** tab.
2. **Add key**, **Create new key**, type **JSON**, **Create**.
3. A `.json` file downloads. This file is the credential. Treat it like a password: do not email it, do not put it in a shared drive, do not commit it to any repo.

## 5. Put the key in GitHub

1. In your bridge repo: **Settings**, **Secrets and variables**, **Actions**, **New repository secret**.
2. Name: `GOOGLE_SA_KEY`.
3. Value: open the JSON file in a text editor, copy everything, paste it.
4. Save. You can now delete the downloaded JSON file from your computer.

## 6. Share your Drive folder with the robot

1. In Google Drive, go to the **shared drive** folder where you want the AI to deliver documents. It must be a shared drive: service accounts cannot create files in My Drive.
2. Share it with the service account email from step 3, role **Content manager**.

Done. The bridge can now create documents in that folder and nowhere else.

## Troubleshooting

- **Result says 403 or "insufficient permissions":** the folder is not shared with the service account email, or it is a My Drive folder instead of a shared drive.
- **Result says 404:** the folder or doc ID in the request is wrong. IDs are the long string in the Drive URL.
- **The Action fails immediately with a JSON error:** the `GOOGLE_SA_KEY` secret is not the full JSON file content. Re-paste it completely, including the braces.
