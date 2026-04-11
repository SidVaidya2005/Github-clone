# apnaGit — Custom VCS Internals

apnaGit is a simplified Git-like VCS built on top of the filesystem and AWS S3. It is invoked as a CLI from `backend/index.js` and operates entirely on the **current working directory** (`process.cwd()`), not on the backend source directory.

## Local storage layout

```
<project-dir>/
  .apnaGit/
    config.json          { "bucket": "<S3_BUCKET value at init time>" }
    staging/             flat directory — files copied here by `add`
    commits/
      <uuid-v4>/         one dir per commit
        <file1>          copied from staging
        <file2>
        commit.json      { "message": "...", "date": "<ISO string>" }
```

## Command internals

### `init`
- Creates `.apnaGit/` and `.apnaGit/commits/`
- Writes `config.json` with `{ bucket: process.env.S3_BUCKET }` — **only read at init time**. Changing `S3_BUCKET` after init has no effect; edit `config.json` manually.

### `add <file>`
- Reads `<file>` relative to `process.cwd()`
- Copies it flat into `.apnaGit/staging/` (directory structure is not preserved)

### `commit <message>`
- Generates a UUID v4 as the commit ID
- Creates `.apnaGit/commits/<uuid>/`
- Copies all files from `staging/` into the commit directory
- Writes `commit.json` with message + ISO date
- Does **not** clear the staging directory after committing

### `push`
- Reads bucket name from `.apnaGit/config.json`
- Uploads the entire `.apnaGit/commits/` tree to S3 using `aws-sdk` v2
- Uses AWS credentials from environment or `~/.aws/credentials`

### `pull`
- Downloads all objects from the S3 bucket into `.apnaGit/commits/`
- Overwrites local commits if keys match

### `revert <commitID>`
- Reads files from `.apnaGit/commits/<commitID>/`
- Copies them back to `process.cwd()` (restores working directory)
- `commit.json` inside the commit dir is also copied out — clean this up manually

## AWS configuration

`config/aws-config.js` hard-codes:
- Region: `ap-south-1`
- Bucket: `"insert_bucket_name"` (placeholder)

The effective bucket comes from `config.json` written at `init` time. The AWS config file region must be changed in source if you use a different region.

## Known limitations
- Staging is never cleared — files accumulate across commits unless manually removed
- No branching, merging, or diffing
- Directory structures are flattened on `add`
- `commit.json` bleeds into the working directory on `revert`
- No `.apnaGitignore` support
