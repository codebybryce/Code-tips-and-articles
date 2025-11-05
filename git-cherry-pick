Perfect. Here’s a GitHub-ready version with:
✅ Syntax highlighting for Git commands
✅ A clean Markdown header banner
✅ GitHub Pages/Docsify-friendly formatting (renders beautifully in dark/light themes)

You can save this as:

/docs/git/cherry-pick-specific-files.md


⸻


# 🧩 Git Cherry-Pick: Apply a Single Commit or Specific Files from Another Branch

![Git Tips by Bryce](https://raw.githubusercontent.com/github/explore/main/topics/git/git.png)

**Author:** Bryce Robinson  
**Date:** November 2025  
**Purpose:**  
A concise, developer-friendly guide for merging a single commit or specific files (like `.less` files) from another remote branch into your current branch.

---

## 🧭 Overview

This guide covers how to:

- Merge a **single commit** from another branch  
- Cherry-pick only **specific files** (e.g. `.less`)  
- Clean up or remove unwanted files after partial cherry-picks  

---

## ⚙️ Scenario Example

You’re working on a feature branch (`feature/STRAT-12839_int_2`) and want to pull one commit from another branch (`STRAT-12839`):

```bash
Commit ID: 5961e0fb5dc5b9ba443b99a8302d6f262a71549b
Branch: STRAT-12839


⸻

🧱 Part 1 — Cherry-Pick a Single Commit
	1.	Checkout your local target branch

git checkout feature/STRAT-12839_int_2


	2.	Fetch the latest remote branches and commits

git fetch origin


	3.	Cherry-pick the desired commit

git cherry-pick 5961e0fb5dc5b9ba443b99a8302d6f262a71549b

💡 If the commit lives on a remote branch not yet fetched:

git fetch origin STRAT-12839
git cherry-pick 5961e0fb5dc5b9ba443b99a8302d6f262a71549b


	4.	Resolve conflicts if they appear

git add .
git cherry-pick --continue


	5.	Push your branch after verifying

git push origin feature/STRAT-12839_int_2



⸻

🎯 Part 2 — Cherry-Pick Only Specific Files (e.g. .less)

When you only need certain files from the commit.

⸻

🧩 Option 1 — Cherry-Pick Without Committing, Then Stage Selectively

git checkout feature/STRAT-12839_int_2
git fetch origin
git cherry-pick -n 5961e0fb5dc5b9ba443b99a8302d6f262a71549b

Now unstage all files:

git restore --staged .

Restore all changes (remove unwanted files):

git restore .

Reapply and commit only .less files:

git checkout 5961e0fb5dc5b9ba443b99a8302d6f262a71549b -- '*.less'
git add '*.less'
git commit -m "Cherry-picked .less file changes from commit 5961e0f"


⸻

⚙️ Option 2 — Checkout Specific Files Directly (Simplest)

git fetch origin
git checkout 5961e0fb5dc5b9ba443b99a8302d6f262a71549b -- '*.less'
git commit -m "Pulled only .less files from commit 5961e0f"

✅ This avoids cherry-pick conflicts and brings in only what you want.

⸻

🧠 Option 3 — Apply a Patch for .less Files Only

git diff 5961e0fb5dc5b9ba443b99a8302d6f262a71549b^ \
         5961e0fb5dc5b9ba443b99a8302d6f262a71549b -- '*.less' > less_changes.patch

git apply less_changes.patch
git commit -am "Applied .less changes from commit 5961e0f"


⸻

🧹 Cleanup Commands

Task	Command
Unstage all files	git restore --staged .
Discard unwanted changes	git restore .
Checkout only specific files	git checkout <commit> -- '*.less'
Undo a mistaken commit	git reset HEAD~1


⸻

✅ Recommended Approach
	•	For specific file types, use Option 2 – clean and conflict-free.
	•	For partial commit integration, use Option 1 – offers fine-grained control.

⸻

🧾 Quick Reference

# Merge a single commit
git fetch origin
git cherry-pick <commit-hash>

# Apply but don’t auto-commit
git cherry-pick -n <commit-hash>

# Keep only .less files
git restore --staged .
git restore .
git checkout <commit-hash> -- '*.less'
git add '*.less'
git commit -m "Applied .less file changes only"


⸻

🧠 Notes
	•	Use git status often to verify staged files.
	•	Always create a temporary backup branch before experimenting with partial cherry-picks.
	•	Consider documenting cherry-picked commits in your PR description for traceability.

⸻

Last Updated: November 2025
Maintained by: Bryce Robinson