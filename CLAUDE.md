# Lexicon Flashcards — Claude Code Instructions

## GitHub Push Process

**Every time the user says "push to GitHub" or "push the updates", follow ALL steps below without being asked.**

### Step 1 — Commit & Push

```bash
git add <changed files>          # stage specific files, never git add -A blindly
git commit -m "type: message"    # follow Conventional Commits (feat/fix/docs/refactor)
git push origin main
```

### Step 2 — Create GitHub Issue(s)

`gh` CLI is NOT installed. Use `curl` + the stored OAuth token from git credentials.

**Get the token:**
```bash
git credential fill <<'EOF'
protocol=https
host=github.com
EOF
# Copy the `password=` value — that is the token
```

**Create an issue:**
```bash
GITHUB_TOKEN="<token>"
REPO="suraj-0023/Flashcards_app"

curl -s -X POST "https://api.github.com/repos/${REPO}/issues" \
  -H "Authorization: token ${GITHUB_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "<issue title matching the commit>",
    "body": "<markdown body — include summary, what changed, and the commit SHA>"
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('number'), d.get('html_url'))"
```

**Issue body template:**
```markdown
## Summary
- Bullet points of what changed and why

## Changes
| Area | Detail |
|---|---|
| File/feature | What was done |

## Commit
<SHA>

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### Step 3 — Close the Issue

```bash
curl -s -X PATCH "https://api.github.com/repos/${REPO}/issues/<NUMBER>" \
  -H "Authorization: token ${GITHUB_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"state": "closed"}' \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('state'), d.get('html_url'))"
```

### One-shot reference (copy-paste skeleton)

```bash
GITHUB_TOKEN=$(git credential fill <<'EOF'
protocol=https
host=github.com
EOF
# paste password value here
)
REPO="suraj-0023/Flashcards_app"
ISSUE_NUMBER=<N>

# Close
curl -s -X PATCH "https://api.github.com/repos/${REPO}/issues/${ISSUE_NUMBER}" \
  -H "Authorization: token ${GITHUB_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"state": "closed"}' | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('state'), d.get('html_url'))"
```

---

## Repo Details

| Key | Value |
|---|---|
| Remote | `https://github.com/suraj-0023/Flashcards_app.git` |
| Default branch | `main` |
| GitHub CLI | Not installed — use `curl` + `git credential fill` |
| Credential store | macOS Keychain via `git credential fill` (protocol=https, host=github.com) |

---

## Commit Message Convention

```
feat:     new feature or content
fix:      bug fix
docs:     documentation only
refactor: code change, no behaviour change
chore:    tooling, config, deps
```

---

## Project Overview

Single-file app (`Flashcards_app_project/vocab_vscode.html`) backed by Firebase Auth + Firestore.  
Vocabulary sourced from `vocabulary.json` and `quiz_words.json`.  
User data (notes, custom cards, vocab, decks) stored in namespaced localStorage and synced to Firestore.
