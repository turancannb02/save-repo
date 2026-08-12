<p align="center">
  <img src="save-repo.png" alt="Save Repo Shortcut" width="480">
</p>

# Save Repo

**Save Repo** is an Apple Shortcut that lets you update a file in a GitHub repository directly from your iPhone using the GitHub REST API.

It is designed as a lightweight, Git-free workflow for quickly updating `README.md` or other text files from Apple Shortcuts.

> **Security:** The published Shortcut does **not** contain a GitHub Personal Access Token. The token is represented by a placeholder and must be provided by each user.

## What It Does

The Shortcut:

1. Builds the GitHub API URL
2. Authenticates with a user-provided GitHub Personal Access Token
3. Fetches the existing file from GitHub
4. Extracts the file's current `SHA`
5. Generates the new file content
6. Encodes the content as Base64
7. Sends a `PUT` request to GitHub
8. Creates a new commit with the updated file

### Workflow

```text
New File Content
       ↓
GET existing file
       ↓
Extract SHA
       ↓
Base64 Encode
       ↓
Build JSON request body
       ↓
PUT updated file
       ↓
GitHub Commit
```

---

# Requirements

- An iPhone or iPad
- Apple Shortcuts
- A GitHub account
- A GitHub Personal Access Token
- A repository where you have permission to modify the target file

---

# Installation

Download the included:

```text
Save Repo.shortcut
```

and import it into Apple Shortcuts.

The Shortcut is intentionally distributed without a real GitHub token.

---

# Configuration

The published Shortcut uses placeholders for repository information and authentication.

You will see values similar to:

```text
https://api.github.com/repos/{OWNER}/{REPOSITORY}/contents/README.md
```

and:

```text
[YOUR_TOKEN]
```

Replace these with your own values.

## 1. Repository URL

Set:

```text
https://api.github.com/repos/{OWNER}/{REPOSITORY}/contents/{FILE_PATH}
```

For example:

```text
https://api.github.com/repos/octocat/hello-world/contents/README.md
```

You can use any file path supported by the GitHub Contents API, not only `README.md`.

---

## 2. GitHub Personal Access Token

Create your own GitHub Personal Access Token and use it in the Shortcut.

For a fine-grained token, grant access only to the repository you want to modify and give it the minimum required permission:

```text
Repository permissions
└── Contents
    └── Read and write
```

The Shortcut expects the token in the authorization header as:

```text
Bearer YOUR_TOKEN
```

### Never share your token

Do **not**:

- Commit your token to GitHub
- Put your token in `README.md`
- Share screenshots containing your token
- Upload a Shortcut containing your real token
- Publish your token in an issue or discussion

If a token is accidentally exposed, revoke it immediately and create a new one.

---

# How the Shortcut Works

## Step 1 — GET the Existing File

The Shortcut sends a `GET` request to:

```text
https://api.github.com/repos/{OWNER}/{REPOSITORY}/contents/{FILE_PATH}
```

Headers:

```text
Accept: application/vnd.github+json
Authorization: Bearer YOUR_TOKEN
X-GitHub-Api-Version: 2026-03-10
```

GitHub returns metadata for the file, including its current SHA.

Example response:

```json
{
  "name": "README.md",
  "path": "README.md",
  "sha": "abc123...",
  "content": "SGVsbG8gV29ybGQ..."
}
```

The `sha` is required when updating an existing file.

---

## Step 2 — Extract the SHA

The Shortcut gets:

```text
sha
```

from the response Dictionary.

This value is passed into the final update request.

Without the current SHA, GitHub will reject an update to an existing file.

---

## Step 3 — Generate the New Content

The Shortcut creates the new file content using a Text action.

For example:

```markdown
# Awesome Repositories

## Repositories

- [owner/repository](https://github.com/owner/repository) - Description
```

The content can also be generated dynamically from other Shortcut actions.

---

## Step 4 — Base64 Encode the Content

GitHub's Contents API expects the file content in Base64.

The Shortcut uses:

```text
Encode Text with Base64
```

### Important: Disable line breaks

Set:

```text
Line Breaks → None
```

Do **not** use:

```text
Every 76 Characters
```

Line breaks in the Base64 output can cause GitHub to return:

```text
422
content is not valid Base64
```

The working flow is:

```text
README Text
    ↓
Encode Text with Base64
    ↓
Line Breaks: None
```

---

## Step 5 — Convert the Base64 Output to Text

The Shortcut then uses:

```text
Get Text from Base64 Encoded
```

This produces the Base64 string that is sent as the `content` field in the GitHub request.

---

## Step 6 — Build the Request Body

The Shortcut creates a Dictionary containing:

```text
message
content
sha
```

For example:

```json
{
  "message": "Update README",
  "content": "SGVsbG8gV29ybGQ=",
  "sha": "abc123..."
}
```

Where:

- `message` is the Git commit message
- `content` is the Base64-encoded file content
- `sha` is the SHA retrieved from the GET request

---

## Step 7 — PUT the Updated File

The Shortcut sends a `PUT` request to the same Contents API endpoint:

```text
https://api.github.com/repos/{OWNER}/{REPOSITORY}/contents/{FILE_PATH}
```

with:

```text
Accept: application/vnd.github+json
Authorization: Bearer YOUR_TOKEN
X-GitHub-Api-Version: 2026-03-10
```

and the JSON request body:

```json
{
  "message": "Update README",
  "content": "BASE64_CONTENT",
  "sha": "CURRENT_FILE_SHA"
}
```

GitHub then updates the file and creates a new commit.

---

# Troubleshooting

## `422 — content is not valid Base64`

Make sure the Base64 action is configured as:

```text
Encode Text with Base64
Line Breaks → None
```

Also verify that the `content` field contains the Base64 output and not the original text.

---

## `422 — sha wasn't supplied`

Make sure the `sha` value comes from the GET request:

```text
GET file
   ↓
Get Value for "sha" in Dictionary
   ↓
sha in PUT request
```

---

## `401 — Bad credentials`

Check:

- Your token is valid
- The token has access to the repository
- The Authorization header uses the correct format

```text
Authorization: Bearer YOUR_TOKEN
```

There must be a space between `Bearer` and the token.

---

## `403 — Forbidden`

Check the permissions of your Personal Access Token.

For a fine-grained token, make sure:

```text
Repository access
    ↓
Target repository
    ↓
Contents: Read and write
```

---

# Extending the Shortcut

Although this project uses `README.md` as the example, the same workflow can update other files.

For example:

```text
contents/data.json
contents/changelog.md
contents/config/settings.json
contents/docs/example.md
```

The general pattern is:

```text
GET
 ↓
Get SHA
 ↓
Generate content
 ↓
Base64 Encode
 ↓
PUT
 ↓
Commit
```

This can be used for:

- Markdown files
- JSON files
- Documentation
- Changelogs
- Generated lists
- Configuration files
- Project metadata
- Other text-based files

---

# Why Apple Shortcuts?

This project demonstrates that Apple's Shortcuts app can act as a lightweight GitHub client using HTTP requests.

It can be useful when you want to:

- Quickly update a repository from your phone
- Automate small GitHub updates
- Generate documentation from mobile data
- Build simple mobile GitHub workflows
- Experiment with REST APIs without writing a full application

No local Git installation is required.

---

# Repository Structure

```text
save-repo/
├── README.md
├── Save Repo.shortcut
└── LICENSE
```

The included `.shortcut` file is the actual Apple Shortcut used to perform the workflow.

---

# Security Notice

This repository intentionally contains **no real GitHub credentials**.

The distributed Shortcut uses:

```text
[YOUR_TOKEN]
```

as a placeholder.

Users must provide their own GitHub Personal Access Token.

For security, use the smallest repository scope and permissions necessary for your use case.

---

# License

MIT License

See [`LICENSE`](LICENSE) for details.

---

## Built With

- [Apple Shortcuts](https://support.apple.com/guide/shortcuts/welcome/ios)
- GitHub REST API
- GitHub Contents API

The Shortcut uses GitHub's **Create or Update File Contents** endpoint.
