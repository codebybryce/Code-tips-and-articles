# 🧭 Merging an Out-of-Date Release Branch into Develop (Safely & Elegantly)

### Scenario

You have a `release/emea` branch that has diverged from `develop`.  
Several feature branches were based on `release/emea` and merged back into it, while other teams continued merging directly into `develop`.

Now you want to merge `release/emea` → `develop`, but:

- There are **many merge conflicts**
- You **don’t want to overwrite unrelated code**
- You **don’t have PR approval rights** for `release/emea`

---

## 🎯 Goal

Bring valid `release/emea` changes into `develop` **without overwriting or losing anyone else’s work**, and do it cleanly enough for review.

---

## ✅ Recommended Approach A — “Replay Release on Top of Develop”

This replays all `release/emea` changes on top of the latest `develop`.  
It’s clean, traceable, and ideal when you want to preserve all legitimate release work.

---

### 1. Prepare & Inspect

```bash
git fetch origin
git switch develop && git pull --ff-only
git switch -c replay/release-emea-on-develop

# Find divergence point between branches
MB=$(git merge-base origin/develop origin/release/emea)

# See what’s unique on release/emea
git log --oneline --graph --left-right --cherry-pick origin/develop...origin/release/emea
```

---

### 2. Rebase the Release Branch onto Develop

```bash
git switch -c replay/release-emea origin/release/emea
git config rerere.enabled true   # Reuse conflict resolutions

# Replay release/emea onto develop (preserves merge history)
git rebase --rebase-merges --onto origin/develop $MB

# Resolve conflicts as they appear
git rebase --continue
```

---

### 3. Verify the Replayed Branch

```bash
# Compare the old vs new diff for sanity
git range-diff origin/develop...origin/release/emea origin/develop...HEAD

# Quick summary of what will land
git log --oneline origin/develop..HEAD
```

---

### 4. Push & Open a Pull Request

```bash
git push -u origin replay/release-emea
# → Open a PR with base = develop, compare = replay/release-emea
```

---

### 🧠 Why This Works

- You **don’t overwrite** anything on `develop`
- You **re-apply release changes** as if they were built today
- You get **fine-grained conflict visibility** instead of one giant merge

---

## 🧩 Alternate Approach B — “Cherry-Pick Only What You Need”

If you only want certain fixes or features from `release/emea`, cherry-pick those commits.

---

### 1. Identify Commits Unique to Release

```bash
git fetch origin
git switch develop && git pull --ff-only
git switch -c integration/release-emea-trueup

MB=$(git merge-base origin/develop origin/release/emea)

# List commits on release/emea not in develop
git log --reverse --no-merges --pretty=%H %s $MB..origin/release/emea
```

---

### 2. (Optional) Filter by Path

```bash
git log --reverse --pretty=%H -- packages/app-emea services/emea-api   --ancestry-path $MB..origin/release/emea
```

---

### 3. Cherry-Pick the Needed Commits

```bash
git config rerere.enabled true
git cherry-pick -x <sha1> <sha2> <sha3> ...
git cherry-pick --continue
```

---

### 4. Validate & Open a PR

```bash
git log --oneline origin/develop..HEAD
git push -u origin integration/release-emea-trueup
# → Open a PR with base = develop, compare = integration/release-emea-trueup
```

💡 Tip:  
Use `git cherry -v origin/develop origin/release/emea` to see which commits are missing (`+`) or already present (`-`).

---

## 🧱 Safety Guardrails

Always do these before making big merges or rebases:

```bash
git tag backup/release-emea $(git rev-parse origin/release/emea)
git tag backup/develop      $(git rev-parse origin/develop)
```

Then:

- Run tests & linters after resolving conflicts
- Use `git range-diff` before opening the PR
- Avoid blanket merge strategies like `ours`/`theirs` — they hide real differences

---

## ⚙️ If Forced to Merge (Not Recommended)

If policy requires a direct merge instead of replay/cherry-pick:

```bash
git fetch origin
git switch develop && git pull --ff-only
git switch -c merge/release-emea-into-develop
git merge --no-ff origin/release/emea
# Resolve conflicts carefully
git push -u origin merge/release-emea-into-develop
```

Expect heavy conflicts and a messy diff.

---

## 🔒 Bitbucket Permission Note

Since you **can’t approve PRs to `release`**, this workflow focuses on creating a PR **into `develop`**, which you can request reviewers for.  
All changes are self-contained and traceable.

---

## 🤖 Copilot Prompts for Easier Conflict Resolution (VS Code)

Use GitHub Copilot Chat while resolving conflicts:

### Explain the conflict
> “Copilot, explain the cause of this merge conflict in this file and summarize what each side changed. Suggest a minimal, correct resolution consistent with the surrounding code.”

### Suggest a resolution
> “Copilot, merge the EMEA-specific logic from `release/emea` with the new validation added in `develop`.”

### Generate regression tests
> “Copilot, write a test that ensures the merged function keeps both: (1) the new null guard from develop, and (2) the rounding rule from release/emea.”

---

## 🧭 Which Should You Use?

| Scenario | Best Approach |
|-----------|----------------|
| You need all EMEA release work carried forward | **Approach A – Replay** |
| You only need a few fixes or features | **Approach B – Cherry-pick** |
| You must merge by policy (not ideal) | Direct merge, with caution |

---

## 📘 Summary

- Avoid a raw merge between divergent branches  
- Rebase (`--rebase-merges`) or cherry-pick for clarity and traceability  
- Validate with `git range-diff`  
- Protect both branches with tags before proceeding  
- Keep all work reviewable via PR into `develop`

---

**Author:** Bryce Robinson  
**Category:** Git Workflows  
**Tags:** `git`, `branching`, `release`, `develop`, `merge`, `cherry-pick`, `Bitbucket`, `Copilot`

---
