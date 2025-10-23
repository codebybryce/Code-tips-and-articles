# 🧭 Merging an Out-of-Date Release Branch into Develop (Safely & Elegantly)

_Last updated: 2025-10-23_

---

## 📘 Overview

This guide explains how to safely merge an outdated release branch (like `release/emea`) into `develop` without overwriting other teams’ work, while keeping changes auditable and clean.  
It includes best practices, step-by-step commands, and merge conflict handling guidelines for VS Code.

---

## 🎯 Scenario

You have a `release/emea` branch that has diverged from `develop`.  
Several feature branches were based on `release/emea` and merged back into it, while other teams continued merging directly into `develop`.

Now you want to merge `release/emea` → `develop`, but:

- There are **many merge conflicts**
- You **don’t want to overwrite unrelated code**
- You **don’t have PR approval rights** for `release/emea`

---

## ✅ Recommended Approach A — “Replay Release on Top of Develop”

This replays all `release/emea` changes on top of the latest `develop`.  
It’s clean, traceable, and ideal when you want to preserve all legitimate release work.

### Steps

```bash
git fetch origin
git switch develop && git pull --ff-only
git switch -c replay/release-emea-on-develop

MB=$(git merge-base origin/develop origin/release/emea)
git log --oneline --graph --left-right --cherry-pick origin/develop...origin/release/emea

git switch -c replay/release-emea origin/release/emea
git config rerere.enabled true
git rebase --rebase-merges --onto origin/develop $MB
git rebase --continue

git range-diff origin/develop...origin/release/emea origin/develop...HEAD
git log --oneline origin/develop..HEAD

git push -u origin replay/release-emea
# → Open a PR with base = develop, compare = replay/release-emea
```

---

## 🧩 Alternate Approach B — “Cherry-Pick Only What You Need”

Use if you only need specific fixes or features from the release branch.

```bash
git fetch origin
git switch develop && git pull --ff-only
git switch -c integration/release-emea-trueup

MB=$(git merge-base origin/develop origin/release/emea)
git log --reverse --no-merges --pretty=%H %s $MB..origin/release/emea

git cherry-pick -x <sha1> <sha2> <sha3> ...
git cherry-pick --continue

git push -u origin integration/release-emea-trueup
# → PR: base = develop, compare = integration/release-emea-trueup
```

💡 Use `git cherry -v origin/develop origin/release/emea` to see which commits are missing (`+`) or already present (`-`).

---

## 🧱 Safety Guardrails

Before rebasing or merging:

```bash
git tag backup/release-emea $(git rev-parse origin/release/emea)
git tag backup/develop $(git rev-parse origin/develop)
```

Then:
- Run tests & linters after each conflict resolution  
- Use `git range-diff` before opening the PR  
- Avoid using `ours`/`theirs` strategies — they can hide changes

---

## ⚙️ If Forced to Merge (Not Recommended)

```bash
git fetch origin
git switch develop && git pull --ff-only
git switch -c merge/release-emea-into-develop
git merge --no-ff origin/release/emea
git push -u origin merge/release-emea-into-develop
```

Expect heavy conflicts and extra review.

---

## 🔒 Bitbucket Permission Note

Since you can’t approve PRs to `release`, this workflow focuses on creating a PR **into `develop`**, where you can still request reviewers.

---

## 🪄 Appendix: `git switch` vs `git checkout`

Starting with Git 2.23, `git switch` was introduced to simplify branch operations.

| Task | Modern Command (`git switch`) | Legacy Equivalent (`git checkout`) |
|------|-------------------------------|------------------------------------|
| Switch to an existing branch | `git switch develop` | `git checkout develop` |
| Create and switch to a new branch | `git switch -c replay/release-emea-on-develop develop` | `git checkout -b replay/release-emea-on-develop develop` |
| Restore a file | `git restore src/index.js` | `git checkout release/emea -- src/index.js` |

### Key Takeaways

- `git switch` → branch operations only  
- `git restore` → file restoration  
- `git checkout` → legacy catch-all (can be confusing)

Prefer `switch` + `restore` for clarity and safety.

---

# 🧰 Merge Conflict Resolution in VS Code

## 1. Understand the Context

- “Current Change” = your branch (e.g., `develop`)  
- “Incoming Change” = the branch being merged (e.g., `release/emea`)  

Read both sides carefully before choosing.

## 2. Keep `develop` as the Baseline

`develop` usually has the latest truth. Only override it if the `release` change fixes a bug or adds new functionality.

## 3. Preserve Both When in Doubt

Manually merge logical differences rather than accepting one side blindly.

Example:
```diff
<<<<<<< HEAD
if (validateInput(data)) processData(data);
=======
if (isValid(data)) processData(data, 'emea');
>>>>>>> release/emea
```
✅ **Merged:**
```js
if (validateInput(data) || isValid(data)) processData(data, 'emea');
```

## 4. Test Early and Often

```bash
npm run test
```
Run after every few files to catch issues early.

## 5. Commit in Small Chunks

Commit one file or module at a time:
```bash
git add src/utils/helper.js
git commit -m "Resolve conflict in helper.js"
git rebase --continue
```

## 6. Use VS Code Tools

- **Accept Both Changes** when both make sense  
- **Compare Changes** view helps visualize intent  
- Use **Source Control panel** to track unresolved conflicts

## 7. Ask “What Was the Intention?”

If unsure, consider:
- Which branch has the latest logic?
- Was this change region-specific or global?
- Could merging both create duplicates?

## 8. Use Copilot for Context-Aware Help

> “Explain this merge conflict and propose a merged version that keeps develop’s null check and release’s localization.”

## 9. Use CLI for Deep Diffs

```bash
git diff --merge
git mergetool
```

## 10. Final Review Before Pushing

```bash
git diff origin/develop
```
Ensure only intentional changes remain.

---

## ✅ Final Merge Checklist

- [x] No conflict markers remain (`<<<<<<<`, `=======`, `>>>>>>>`)  
- [x] Builds and tests pass  
- [x] Functional smoke tests done  
- [x] `git diff origin/develop` reviewed  
- [x] Clear commit message documenting resolution decisions  

---

**Author:** Bryce Robinson  
**Category:** Git Workflows  
**Tags:** `git`, `branching`, `release`, `develop`, `merge`, `VSCode`, `Copilot`, `Bitbucket`

---
