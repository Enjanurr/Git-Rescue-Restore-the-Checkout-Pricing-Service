\# WORKFLOW.md



\## 1. Git Bisect Finding



\*\*Commit:\*\* `c99fb4209e6fb6e5ed2789893fdb2f893d61d6c6`



\*\*What it broke:\*\* This commit changed the BULK20 discount condition from `items.length >= 5` to `items.length > 5`, causing orders of exactly 5 items to be incorrectly excluded from the 20% discount.



\## 2. Branching Strategy for a Team of 4



I'd recommend \*\*GitHub Flow\*\*. With a small team of 4, Git Flow's multiple long-lived branches (develop, release, hotfix, feature) add process overhead that isn't justified at this scale — merge conflicts and coordination costs go up without much benefit. Pure trunk-based development (everyone committing straight to main) works for very disciplined, CI/CD-heavy teams, but skips code review structure that's valuable for a smaller team still building conventions. GitHub Flow strikes the balance: short-lived feature branches, pull requests for review, and continuous merges into main — simple enough to move fast, structured enough to catch bugs like the one in this lab before they reach main.



\## 3. Fully Removing the Secret from History



`git rm --cached` and `.gitignore` only stop the file from being tracked \*going forward\* — the old commit (`bb459e45...`) still contains the `.env` file's full content in the repository's history and is reachable via `git log`, `git show`, or anyone who has already cloned the repo.



To fully remove it, I'd need to rewrite history using `git filter-repo` (or the older `BFG Repo-Cleaner`) to strip the file from every commit that ever contained it, then force-push the rewritten history and have every collaborator delete their local clone and re-clone fresh (old clones and any forks would still retain the secret in their own history).



This assignment didn't require that step because:

\- The secret was fake/example data, not a real credential — no actual security risk.

\- Rewriting shared history is destructive and disruptive to collaborators, so it's normally treated as a last resort, not a routine step.

\- In a real incident, the more urgent and reliable fix is to \*\*rotate the actual credential\*\* (invalidate the old key, issue a new one) — that neutralizes the leak immediately regardless of whether it's still visible in old commits.



\## 4. Why Rewriting History Was OK in Task 2 but Not After Teammates Pull



In Task 2, the `asdf` commit was rewritten via `git rebase -i` while it existed \*\*only on my local machine\*\* — it had never been pushed, so no one else's work depended on it. Rebasing generates new commit hashes for every commit after the rewritten one, meaning it's technically a new, different history from that point forward.



If teammates had already pulled the original commit, they'd have that old hash in their own local history. If I then rewrote and force-pushed, their local main would diverge from the remote — Git would see two conflicting histories with no common ancestor at that point. They'd either need to force-pull (silently discarding their local work built on the old commit) or manually reconcile the split, which is confusing and error-prone at scale. Shared/pushed history is treated as an append-only contract that other people build on; local, unpushed history is just a personal draft that's safe to edit freely.

