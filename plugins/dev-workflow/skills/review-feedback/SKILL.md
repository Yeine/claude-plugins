# Review PR Feedback

Fetch unresolved review feedback on a pull request, analyze each issue, and optionally fix them. Works with any reviewer — human, CodeRabbit, Copilot, or any other bot.

## Arguments

`$ARGUMENTS` optionally specifies a PR number and/or author filter:
- `/review-feedback` — all unresolved feedback on the current branch's PR
- `/review-feedback 123` — all unresolved feedback on PR #123
- `/review-feedback --author=coderabbitai` — filter to a specific reviewer
- `/review-feedback 123 --author=coderabbitai` — PR #123, specific reviewer only

## 1. Get PR Information

Parse `$ARGUMENTS`:
- Extract PR number if a bare number is present
- Extract author filter if `--author=` is present

If no PR number was provided, get the current branch's PR:
```bash
gh pr view --json number -q '.number'
```

Get repository info:
```bash
gh repo view --json nameWithOwner -q '.nameWithOwner'
```

Split `nameWithOwner` into OWNER and REPO.

## 2. Fetch Review Threads

Use the GitHub GraphQL API to get all review threads with full comment chains:

```bash
gh api graphql -f query='
query($owner: String!, $repo: String!, $pr: Int!) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $pr) {
      title
      url
      reviewThreads(first: 100) {
        nodes {
          id
          isResolved
          isOutdated
          path
          line
          startLine
          diffSide
          comments(first: 50) {
            nodes {
              body
              author { login }
              createdAt
            }
          }
        }
      }
      reviews(first: 50) {
        nodes {
          author { login }
          body
          state
          comments(first: 100) {
            nodes {
              path
              line
              body
            }
          }
        }
      }
    }
  }
}
' -f owner=OWNER -f repo=REPO -F pr=PR_NUMBER
```

Store each thread's `id` — it is needed later to reply and resolve.

Replace OWNER, REPO, and PR_NUMBER with actual values.

## 3. Filter and Organize

From the API response:

1. **Filter out resolved threads** — focus on `isResolved: false` only
2. **Filter out outdated threads** — skip `isOutdated: true` (code has changed since)
3. **Apply author filter** — if `--author` was provided, keep only comments from that author login. Otherwise keep all comments.
4. **Group by file** — organize remaining comments by their `path`

If no unresolved comments remain after filtering, report that and stop.

## 4. Analyze and Categorize

For each unresolved comment:

1. **Read the file** at the mentioned path
2. **Understand the context** around the mentioned line
3. **Categorize:**
   - **Critical** — security vulnerabilities, bugs, crashes, data loss
   - **Warning** — performance issues, missing validation, bad patterns
   - **Suggestion** — style, naming, documentation, minor improvements

## 5. Present Findings

```
## PR Feedback Review

**PR:** #[number] - [title]
**Unresolved Issues:** N
**Reviewers:** [list of authors with comment counts]

---

### Critical Issues

#### `path/to/file:42` (reviewer: @author)
**Issue:** [Reviewer's concern]
**Analysis:** [Your understanding after reading the code]
**Fix:** [Proposed solution]

---

### Warnings

#### `path/to/file:87` (reviewer: @author)
**Issue:** [Reviewer's concern]
**Analysis:** [Your understanding]
**Fix:** [Proposed solution]

---

### Suggestions

#### `path/to/file:123` (reviewer: @author)
**Issue:** [Reviewer's concern]
**Analysis:** [Your understanding]
**Fix:** [Proposed solution]
```

## 6. Ask for Action

After presenting, ask:

> Would you like me to:
> 1. Fix all issues, commit, reply, and resolve threads
> 2. Fix only critical issues, commit, reply, and resolve
> 3. Fix specific issues (list which ones), commit, reply, and resolve
> 4. Fix without committing or replying (local only)
> 5. Just explain the issues without fixing

## 7. Fix Issues

When fixing:

1. Read each file before modifying
2. Apply minimal changes to address the reviewer's concern
3. Verify the fix addresses the specific feedback
4. Do not over-engineer — fix only what was flagged
5. Track which thread IDs were addressed by each fix

## 8. Commit

After all fixes are applied:

1. Stage only the files that were modified
2. Create a commit with a message summarizing the feedback addressed:
   ```
   fix: address PR review feedback

   - [brief description of each fix]
   ```
3. Push to the remote branch so the changes appear on the PR

## 9. Reply and Resolve Threads

For each fixed issue, reply to the review thread explaining what was done, then resolve it.

**Reply to a thread:**
```bash
gh api graphql -f query='
mutation($threadId: ID!, $body: String!) {
  addPullRequestReviewThreadReply(input: {pullRequestReviewThreadId: $threadId, body: $body}) {
    comment { body }
  }
}
' -f threadId=THREAD_ID -f body="Fixed — [brief explanation of what was changed]"
```

**Resolve the thread:**
```bash
gh api graphql -f query='
mutation($threadId: ID!) {
  resolveReviewThread(input: {threadId: $threadId}) {
    thread { isResolved }
  }
}
' -f threadId=THREAD_ID
```

For issues that were analyzed but intentionally NOT fixed (e.g., disagree with the suggestion, or it's a false positive), reply with an explanation of why but do NOT resolve — leave those for the reviewer to decide.

## Rules

- Focus on unresolved comments — do not revisit already resolved issues
- Skip outdated comments — the code may have already changed
- Read surrounding code before suggesting fixes — understand context
- Be conservative — only fix what was specifically flagged
- Preserve the original author's intent and approach
- One issue at a time — do not combine unrelated changes
