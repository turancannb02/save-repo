# Save Repo — GitHub README Updater for Apple Shortcuts

A simple **Apple Shortcut** that lets you update a GitHub repository's `README.md` directly from your iPhone.

The Shortcut uses the **GitHub REST API** to:

1. Fetch the current `README.md`
2. Retrieve its current `SHA`
3. Generate the new README content
4. Encode the content as Base64
5. Update the file through GitHub's Contents API
6. Create a new Git commit automatically

No Git client or computer is required.

## Features

- Works entirely from Apple Shortcuts
- Uses the official GitHub REST API
- Automatically retrieves the current file `SHA`
- Automatically Base64-encodes the README
- Updates the existing file instead of creating duplicates
- Creates a Git commit through the API
- Can be extended to update any text file in a repository

---

## How It Works

The workflow follows this structure:

```text
Input Text
    ↓
GitHub GET request
    ↓
Get existing README
    ↓
Extract current SHA
    ↓
Generate new README
    ↓
Base64 Encode
    ↓
GitHub PUT request
    ↓
New GitHub commit
```

The important part is that GitHub requires the **current SHA of the file** when updating an existing file.

The API request is made to:

```text
PUT https://api.github.com/repos/{OWNER}/{REPOSITORY}/contents/{PATH}
```

For example:

```text
https://api.github.com/repos/OWNER/REPOSITORY/contents/README.md
```

---

# Requirements

- iPhone or iPad
- Apple Shortcuts
- A GitHub account
- A GitHub Personal Access Token (PAT)
- A repository you have permission to modify

---

# 1. Create a GitHub Personal Access Token

Create a Personal Access Token in your GitHub account.

For a fine-grained token, give it access to the repository you want to modify.

The token needs permission to:

```text
Contents → Read and write
```

**Do not publish your token in the Shortcut or repository.**

A token should never be committed to GitHub.

---

# 2. Create the Shortcut

Create a new Shortcut and name it:

```text
Save Repo
```

The Shortcut consists of two GitHub API requests.

---

# 3. GET the Existing README

Add:

**Get Contents of URL**

Configure:

### Method

```text
GET
```

### URL

Use your repository's README endpoint:

```text
https://api.github.com/repos/OWNER/REPOSITORY/contents/README.md
```

For example:

```text
https://api.github.com/repos/OWNER/awesome-repos/contents/README.md
```

### Headers

Add:

```text
Accept: application/vnd.github+json
```

```text
Authorization: Bearer YOUR_GITHUB_TOKEN
```

```text
X-GitHub-Api-Version: 2026-03-10
```

Instead of typing the token directly, it is recommended to store it securely and pass it into the `Authorization` header dynamically.

The resulting response contains information similar to:

```json
{
  "name": "README.md",
  "path": "README.md",
  "sha": "abc123...",
  "content": "SGVsbG8gV29ybGQ..."
}
```

The important field here is:

```text
sha
```

We need this SHA for the update request.

---

# 4. Extract the SHA

Add:

**Get Value for `sha` in Dictionary**

The Dictionary should be the response from the GitHub GET request.

This gives you the current SHA of `README.md`.

Keep this value for the PUT request later.

---

# 5. Create the New README Content

Add a **Text** action containing the README you want to upload.

For example:

```markdown
# Awesome Repositories

## Repositories

- [owner/repository](https://github.com/owner/repository) - Description
```

You can also generate this text dynamically.

For example, the Shortcut can build the README from a list of repositories.

---

# 6. Encode the README as Base64

This is the most important step.

Add:

**Encode Text with Base64**

Pass the new README text into it.

### Important

Set:

```text
Line Breaks → None
```

Do **not** use:

```text
Every 76 Characters
```

GitHub expects the `content` field to contain valid Base64. Adding line breaks can result in:

```text
422 Unprocessable Entity
```

with:

```json
{
  "message": "content is not valid Base64"
}
```

So the correct configuration is:

```text
Encode Text
    ↓
Base64
    ↓
Line Breaks: None
```

---

# 7. Convert the Base64 Result to Text

Add:

**Get Text from Base64 Encoded**

This gives you the Base64 string as plain text.

This value will become the `content` field in the GitHub API request.

The flow is therefore:

```text
README Text
    ↓
Encode Text with Base64
    ↓
Get Text from Base64 Encoded
    ↓
Base64 String
```

---

# 8. Create the Request Dictionary

Add a **Dictionary** with three fields:

```text
message
content
sha
```

Configure them as follows:

```text
message → Update README
content → Base64 Encoded Text
sha     → SHA retrieved from GitHub
```

The resulting JSON is conceptually:

```json
{
  "message": "Update README",
  "content": "SGVsbG8gV29ybGQ=",
  "sha": "abc123..."
}
```

Do not manually write the Base64 content.

Use the output from the Base64 encoding action.

---

# 9. PUT the Updated README

Add another:

**Get Contents of URL**

This time configure:

### Method

```text
PUT
```

### URL

```text
https://api.github.com/repos/OWNER/REPOSITORY/contents/README.md
```

### Headers

```text
Accept: application/vnd.github+json
```

```text
Authorization: Bearer YOUR_GITHUB_TOKEN
```

```text
X-GitHub-Api-Version: 2026-03-10
```

### Request Body

Set:

```text
JSON
```

Then provide the Dictionary created in the previous step.

The final structure should be:

```text
Get Contents of URL
    Method: PUT

    Headers:
        Accept
        Authorization
        X-GitHub-Api-Version

    Request Body:
        {
            message
            content
            sha
        }
```

---

# 10. Run the Shortcut

Run the Shortcut.

If everything is configured correctly, GitHub will return a successful response containing the newly created commit.

You should then see a new commit in the repository:

```text
Update README
```

and the README will contain your newly generated content.

---

# Troubleshooting

## `422 — content is not valid Base64`

This is usually caused by the Base64 output being formatted incorrectly.

Check:

```text
Encode Text with Base64
    Line Breaks → None
```

Do not use:

```text
Every 76 Characters
```

Also make sure that the `content` field contains the **Base64-encoded output**, not the original README text.

---

## `422 — sha wasn't supplied`

Make sure the SHA comes from:

```text
GET README
    ↓
Get Value for "sha" in Dictionary
```

and that this value is passed into:

```text
sha
```

inside the PUT request body.

---

## `401 — Bad credentials`

Check your GitHub token.

Make sure:

- The token is still valid
- The token has access to the repository
- The `Authorization` header is exactly:

```text
Bearer YOUR_TOKEN
```

There must be a space between `Bearer` and the token.

---

## `403 — Forbidden`

The token probably doesn't have sufficient repository permissions.

For a fine-grained Personal Access Token, check:

```text
Repository access
```

and:

```text
Contents → Read and write
```

---

# Security

**Never commit your GitHub Personal Access Token.**

Do not put the token into:

- `README.md`
- screenshots
- GitHub Issues
- source files
- public Shortcut exports
- Git history

If you accidentally expose a token, revoke it immediately and generate a new one.

For a public Shortcut, the token should be supplied by the user rather than embedded in the Shortcut itself.

---

# Extending the Shortcut

The same technique can be used to update other files.

Change:

```text
contents/README.md
```

to another path, for example:

```text
contents/data.json
```

or:

```text
contents/config/settings.json
```

The general GitHub Contents API pattern is:

```text
GET
    ↓
Get SHA
    ↓
Generate file
    ↓
Base64 encode
    ↓
PUT
    ↓
Commit
```

This means the Shortcut can be adapted to automatically update:

- Markdown files
- JSON files
- configuration files
- documentation
- changelogs
- generated lists
- project metadata

---

# Example Use Case

For example, you could maintain an **Awesome Repositories** repository directly from your iPhone.

The Shortcut could take:

```text
Repository
Description
URL
```

and automatically generate:

```markdown
- [owner/repository](https://github.com/owner/repository) - Description
```

Then update the README through GitHub's API.

No computer or Git client is necessary.

---

# License

MIT License

Feel free to modify, extend, and redistribute the Shortcut.

---

## Credits

Built with:

- Apple Shortcuts
- GitHub REST API
- GitHub Contents API

The implementation uses GitHub's **Create or Update File Contents** endpoint.
