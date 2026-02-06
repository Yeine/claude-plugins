# Resolve Merge Conflicts
1. Run `git diff --name-only --diff-filter=U` to list conflicted files
2. Read each conflicted file fully before editing
3. Use targeted line-specific edits (never replace_all on files with conflict markers)
4. After each file edit, re-read to verify zero conflict markers remain
5. Run `git diff --check` to confirm no markers left
6. Stage resolved files with `git add`
