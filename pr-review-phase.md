# PR Review Phase (Batch Loop)

## Purpose
Define the batch-loop PR review phase for Node.js Jira-driven work items. This phase retrieves PR diffs, audits changes for best practices/security, pauses for Human-in-the-Loop (HITL) review, then posts inline review comments and sets the PR review status.

## Input Mappings
- `pr_number` = `{{create_code[item.index].pr_number}}`

## Execution Actions
1. **Fetch PR changes and diff data from GitHub**
   - Retrieve PR metadata, changed files, and unified diff/patch for each changed file.
   - Data needed for auditing and inline comments:
     - file path
     - patch (unified diff hunks)
     - line positions (for inline comments)
     - base/head SHAs

2. **Audit code patterns against Node.js best practices and security rules**
   - Best practices checks (examples):
     - Avoid blocking I/O on request paths.
     - Prefer async/await and proper error handling.
     - Validate/normalize user input; avoid unsafe deserialization.
     - Ensure secrets are not logged.
     - Avoid `eval`, `Function`, unsafe regex, command injection (`child_process.exec` with user input).
     - Use parameterized queries for DB.
     - Enforce HTTP security headers and safe CORS.
   - Security checks (examples):
     - OWASP Top 10 style checks for injection, SSRF, path traversal.
     - Dependency vulnerability awareness (if lockfiles present).
     - Ensure authZ checks exist for privileged actions.

3. **Pause for HITL review**
   - Present findings (grouped by file) and allow reviewer to decide which comments to post.
   - Reviewer chooses `review_status`:
     - `APPROVED` if no changes required.
     - `CHANGES_REQUESTED` if at least one blocking issue exists.

4. **Post inline comments on the GitHub PR and set review status**
   - Create a PR review with:
     - `event`: `APPROVE` or `REQUEST_CHANGES`
     - `comments`: inline comments referencing file + diff position

## Outputs Captured
- `review_status`: `APPROVED` or `CHANGES_REQUESTED`

## API Notes (Implementation Hints)
GitHub REST endpoints (typical):
- Get PR: `GET /repos/{owner}/{repo}/pulls/{pull_number}`
- List files: `GET /repos/{owner}/{repo}/pulls/{pull_number}/files`
- Create review: `POST /repos/{owner}/{repo}/pulls/{pull_number}/reviews`

If using `gh` CLI:
- `gh pr view <pr_number> --json files,additions,deletions,title,body,headRefName,baseRefName`
- `gh api repos/<owner>/<repo>/pulls/<pr_number>/files`

> Inline comment placement requires `position` within the diff; use the `patch` hunks to compute the correct position.
