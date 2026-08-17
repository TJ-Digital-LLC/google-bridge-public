# Connect your AI assistant

The bridge is AI-agnostic: anything that can call the GitHub API can use it. This guide shows the Claude setup, which is what we run in production.

## 1. Create a GitHub token for the assistant

1. GitHub: **Settings** (your profile), **Developer settings**, **Personal access tokens**, **Fine-grained tokens**, **Generate new token**.
2. Name: `bridge-access`. Expiration: your call (we use 6 months and rotate).
3. **Repository access:** "Only select repositories", pick your bridge repo. Nothing else.
4. **Permissions:** Repository permissions, **Contents: Read and write**. Nothing else.
5. Generate and copy the token.

This token can only touch the bridge repo. It cannot see any other repository, and every use is logged as a commit.

## 2. Set up a Claude Project

1. In Claude (Team or Pro), create a Project, for example "Docs Bridge".
2. In the project instructions, paste the content of [../skill/SKILL.md](../skill/SKILL.md) and fill in the three placeholders at the top:
   - `{BRIDGE_REPO}`: your repo, like `your-org/google-bridge`
   - `{GITHUB_TOKEN}`: the token from step 1
   - `{DEFAULT_FOLDER_ID}`: the Drive folder ID where docs should land by default (the long string in the folder URL)

If your Claude plan supports Skills, you can instead install the same file as a skill and keep only the three config lines in the project instructions.

## 3. Test it

In a new chat in that project, ask:

> Create a Google Doc called "Bridge test" with a short hello-world paragraph.

Within about 90 seconds the assistant should reply with a Google Doc link, and the doc should be in your Drive folder. If it is not, check the **Actions** tab of your bridge repo: the run log will say exactly what failed, and the troubleshooting section of [setup-google-cloud.md](setup-google-cloud.md) covers the common cases.

## Notes for other assistants

Any agent that can do these two things can use the bridge:

1. `PUT /repos/{repo}/contents/requests/{id}.json` with the base64-encoded request JSON (this is the "submit" step).
2. Poll `GET /repos/{repo}/contents/results/{id}.json` until it exists (this is the "wait for result" step).

The request and result formats are documented in [operations.md](operations.md).
