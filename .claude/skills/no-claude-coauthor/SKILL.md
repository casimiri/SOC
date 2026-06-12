---
name: no-claude-coauthor
description: Use whenever creating a git commit in this project. Ensures commit messages do NOT include any Claude/Anthropic co-author trailer or "Generated with Claude Code" attribution. Invoke before running `git commit`.
---

# No Claude Co-Author

When committing in this repository, the commit message must be clean of any AI attribution.

## Rules

1. **Never** add a `Co-Authored-By: Claude ...` trailer to commit messages.
2. **Never** add `🤖 Generated with [Claude Code](...)` or any similar "generated with" line.
3. Do not add any `Co-Authored-By` line referencing Anthropic, Claude, or an AI model.
4. Write the commit message as if authored solely by the user. Keep the standard
   subject + body format, just without the AI trailers.

## How to apply

- Before running `git commit`, draft the message and remove any AI co-author or
  attribution trailer.
- Prefer `git commit -m "subject" -m "body"` so no trailer is appended.
- If you catch yourself about to append a `Co-Authored-By: Claude` line — stop and omit it.

## Verify after committing

Run a quick check and amend if a trailer slipped in:

```bash
git log -1 --pretty=%B | grep -iE 'co-authored-by:.*(claude|anthropic)|generated with' \
  && echo "FOUND AI trailer — amend the commit to remove it" \
  || echo "Clean: no AI attribution"
```

If the check finds a trailer, fix it with:

```bash
git commit --amend -m "<clean subject>" -m "<clean body>"
```
