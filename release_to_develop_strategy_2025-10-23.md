# 🧭 Merging an Out-of-Date Release Branch into Develop (Safely & Elegantly)

_Last updated: 2025-10-23_



---

## 🪄 Appendix: `git switch` vs `git checkout`

Starting with Git 2.23, `git switch` was introduced to make branch management more intuitive.  
It separates branch operations from file operations, which were both previously handled by the overloaded `git checkout` command.

| Task | Modern Command (`git switch`) | Legacy Equivalent (`git checkout`) |
|------|-------------------------------|------------------------------------|
| Switch to an existing branch | `git switch develop` | `git checkout develop` |
| Create and switch to a new branch | `git switch -c replay/release-emea-on-develop develop` | `git checkout -b replay/release-emea-on-develop develop` |
| Restore a file (modern separate command) | `git restore src/index.js` | `git checkout release/emea -- src/index.js` |

### 🧠 Key Differences

- `git switch` → only handles **branch movement** (safe, modern, human-friendly)  
- `git restore` → handles **file restoration** (introduced alongside switch)  
- `git checkout` → legacy command that handles both, but can be confusing or risky  

If you’re writing documentation, scripts, or onboarding teammates, prefer `git switch` for clarity.  
Only fall back to `git checkout` when working with older Git versions (pre‑2.23).

---
