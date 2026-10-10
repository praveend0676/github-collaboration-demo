# Merge Conflict Resolution - Step-by-Step Guide (From Scratch)

Complete walkthrough of the GitHub collaboration workflow between two developers,
including how to **create**, **understand**, and **resolve** a **merge conflict**
- entirely from the **command line**.

---

## 1. Overview of What You Will Build

This guide simulates a real-world project with two developers:

| Developer | Role                                   | Folder used locally      |
| --------- | -------------------------------------- | ------------------------ |
| Developer A | Creates the project, manages `main`  | `github-collaboration-demo`  |
| Developer B | Clones the repo, builds a feature     | `developer-b-project`        |

The final workflow is:

```text
        GitHub Repository  (https://github.com/YOUR-USERNAME/github-collaboration-demo)
                |   ^
                |   |  Developer B pushes feature-login, opens a PR
                v   |
      Developer A    Developer B
         main        feature-login
          |               |
          |  A edits main.py  |
          |  on `main`        |  B edits main.py on feature-login
          |                   |      (SAME LINE -> CONFLICT)
          |                   |
          |<-- Pull Request -----------+
          |
          `----> Resolve conflict on the command line
                 Merge PR
                 Delete branch
```

By the end you will understand:

* git clone, fetch, pull, push
* branches and feature branches
* Pull Requests
* merge conflict markers
* resolving conflicts with the command line
* completing the merge

---

## 2. Requirements / Prerequisites

* [Git](https://git-scm.com) installed
* A [GitHub](https://github.com) account
* Python (optional, only to run the app)
* A terminal (PowerShell / Command Prompt / bash)

---

## 3. Start From Zero (Clean Slate)

> If you are redoing this project, remove the old folders and (optionally)
> delete the GitHub repository so you start completely clean.

```powershell
# remove old local copies (from the parent folder)
Remove-Item -Recurse -Force github-collaboration-demo
Remove-Item -Recurse -Force developer-b-project
```

Then create a fresh folder for **Developer A**:

```powershell
mkdir github-collaboration-demo
cd github-collaboration-demo
```

---

## 4. Developer A - Create the Initial Project

### Step 4.1 - Create `main.py`

Create a file called `main.py` containing:

```python
print("GitHub Collaboration Demo")
print("Main application")
```

### Step 4.2 - Initialize Git

```powershell
git init
```

`git init` creates a hidden `.git` folder that stores the Git history.

### Step 4.3 - Stage and Commit

```powershell
git add .
git commit -m "Initial project"
```

### Step 4.4 - Rename the Branch to `main`

```powershell
git branch -M main
git branch
```

Expected output:

```text
* main
```

### Step 4.5 - Create the GitHub Repository

1. Go to https://github.com
2. Click **New repository**
3. Name it: `github-collaboration-demo`
4. Make it **Public**
5. **Do NOT** initialize it with a README, `.gitignore`, or license
   (this keeps the local and remote histories aligned)

### Step 4.6 - Connect Local to Remote

```powershell
git remote add origin https://github.com/YOUR-USERNAME/github-collaboration-demo.git
git remote -v
```

Expected output:

```text
origin  https://github.com/YOUR-USERNAME/github-collaboration-demo.git (fetch)
origin  https://github.com/YOUR-USERNAME/github-collaboration-demo.git (push)
```

### Step 4.7 - Push `main` to GitHub

```powershell
git push -u origin main
```

`-u` sets the upstream so future pushes/pulls on `main` work automatically.

---

## 5. Developer B - Clone the Repository

Move out of Developer A's folder (for example `cd ..`) and clone:

```powershell
git clone https://github.com/YOUR-USERNAME/github-collaboration-demo.git developer-b-project
cd developer-b-project
```

### Verify the clone

```powershell
git status
git remote -v
```

Expected:

```text
On branch main
Your branch is up to date with 'origin/main'.
```

### Fetch and Pull

```powershell
git fetch
git pull origin main
```

Expected output:

```text
Already up to date.
```

* `git fetch` downloads remote changes **without** merging them.
* `git pull` downloads **and** merges them into your branch.

---

## 6. Developer B - Create the Feature Branch

Developer B should never work directly on `main`.

```powershell
git switch -c feature-login
git branch
```

Expected output:

```text
* feature-login
  main
```

The history now looks like:

```text
main (Initial project)
  |
  `---> feature-login   <- you are here
```

---

## 7. Developer B - Implement the Feature

Open `main.py` and edit it:

```python
print("GitHub Collaboration Demo")
print("Main application for developer B")     # <-- line 2 CHANGED
print("Login feature added by developer B")   # <-- new line added
```

### Review the changes

```powershell
git status
git diff
```

Expected:

```text
modified:   main.py
```

```diff
 print("GitHub Collaboration Demo")
-print("Main application")
+print("Main application for developer B")
+print("Login feature added by developer B")
```

### Commit the feature

```powershell
git add main.py
git commit -m "Add login feature by developer B"
```

### Push the feature branch

```powershell
git push -u origin feature-login
```

GitHub now has two branches:

```text
main
feature-login
```

---

## 8. Developer A - Edit `main` (This Creates the Conflict)

At the same time, **Developer A** also edits `main.py`, changing the **same line**:

Go back to Developer A's folder:

```powershell
# from developer-b-project folder:
cd ..
cd github-collaboration-demo
git switch main
```

Open `main.py` and change it to:

```python
print("GitHub Collaboration Demo")
print("Main application for developer A")     # <-- line 2 CHANGED DIFFERENTLY
```

Commit and push:

```powershell
git add main.py
git commit -m "Update main application for developer A"
git push origin main
```

Now both branches changed the same file, and more importantly the **same line**:

```text
main         :  "Main application for developer A"
feature-login:  "Main application for developer B"  + "Login feature added..."
```

That is exactly the situation that produces a **merge conflict**.

---

## 9. Developer B - Create the Pull Request

Go back to Developer B's folder and push any remaining work:

```powershell
cd ..
cd developer-b-project
git push -u origin feature-login
```

Then on GitHub.com:

1. Open the repository.
2. Click **Compare & pull request** (GitHub offers it for `feature-login`).
3. Confirm:
   * Base: `main`
   * Compare: `feature-login`
4. Title: `Add login feature`
5. Description: (optional, e.g. "Adds a login feature by developer B.")
6. Click **Create pull request**.

GitHub will now show:

```text
This branch has conflicts that must be resolved.
```

This is expected - we designed it this way.

---

## 10. Understand the Conflict Markers

Git marks a conflict in the file using three types of lines:

```text
<<<<<<< HEAD
... content that is on the CURRENT branch (feature-login) ...
=======
... content that is coming IN from the other branch (main) ...
>>>>>>> origin/main
```

For `main.py` the conflict will look like:

```text
print("GitHub Collaboration Demo")
<<<<<<< HEAD
print("Main application for developer B")
print("Login feature added by developer B")
=======
print("Main application for developer A")
>>>>>>> origin/main
```

| Marker | Meaning                                   |
| ------ | ----------------------------------------- |
| `<<<<<<< HEAD` | Start of the section on your current branch |
| `=======`      | Separator between the two versions        |
| `>>>>>>> <branch>` | End of the section coming from the other branch |

You must **choose one version, merge both, or combine them** - and then
**delete all three marker lines**.

---

## 11. Resolve the Conflict (Command Line) - Developer B

### Step 11.1 - Make sure you are on the feature branch

```powershell
git switch feature-login
```

### Step 11.2 - Fetch and merge the remote `main`

```powershell
git fetch origin
git merge origin/main
```

Expected output:

```text
Auto-merging main.py
CONFLICT (content): Merge conflict in main.py
Automatic merge failed; fix conflicts and then commit the result.
```

Git has NOT finished - it is waiting for you to fix `main.py`.

### Step 11.3 - Check the status

```powershell
git status
```

Expected:

```text
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

unmerged:
        main.py
```

### Step 11.4 - Open `main.py` and fix the conflict

Open the file. It contains conflict markers. Edit it to the **final** content that
you want. For example, keep Developer A's line AND Developer B's feature:

```python
print("GitHub Collaboration Demo")
print("Main application for developer A")
print("Login feature added by developer B")
```

Make sure all of these are gone:

```text
<<<<<<< HEAD
=======
>>>>>>> origin/main
```

### Step 11.5 - Mark it resolved and commit

```powershell
git add main.py
git commit -m "Resolve merge conflict in main.py"
```

Your commit is now a **merge commit** with two parents.

### Step 11.6 - Push the resolved branch

```powershell
git push origin feature-login
```

Expected output:

```text
Everything up-to-date   (or the new merge commit was pushed)
```

### Step 11.7 - Verify the conflict is gone on GitHub

Refresh the Pull Request page. 
This branch has no conflicts with the base branch.
```

The PR is now ready to merge.

---

## 12. Complete the Merge (GitHub)

1. On the PR page click **Merge pull request**.
2. Click **Confirm merge**.
3. Click **Delete branch** to clean up `feature-login`.

The merge is complete.

---

## 13. Final Verification - Developer A

Go back to Developer A's folder:

```powershell
cd ..
cd github-collaboration-demo
git switch main
git pull origin main
```

Run the application:

```powershell
python main.py
```

Expected output (the resolved content):

```text
GitHub Collaboration Demo
Main application for developer A
Login feature added by developer B
```

View the full history:

```powershell
git log --oneline --graph --decorate --all
```

Expected graph:

```text
*   Merge pull request #2 from YOUR-USERNAME/feature-login
|\
| * Resolve merge conflict in main.py
| * Update main application for developer A   (merged from origin/main)
| * Add login feature by developer B
* | Add login feature
|/
* Initial project
```

---

## 14. Optional - Re-do the Conflict Resolution Quickly

If you want to practice again without creating a new project, you can abort a
merge at any time:

```powershell
git merge --abort
```

This undoes the partial merge and puts you back before the conflict.

---

## 15. Other Conflict Resolution Techniques (Command Line)

### 15.1 - Keep your version (current branch) for the whole file

```powershell
git checkout --ours main.py
git add main.py
git commit
```

### 15.2 - Accept the other branch's version for the whole file

```powershell
git checkout --theirs main.py
git add main.py
git commit
```

### 15.3 - Use a merge tool (visual)

Git will open your configured merge tool:

```powershell
git mergetool
```

### 15.4 - Resolve during a rebase instead of a merge

```powershell
git pull origin main --rebase
# fix conflicts, then:
git add main.py
git rebase --continue
git push --force-with-lease origin feature-login
```

> Caution: `--force-with-lease` is safe only on a branch that no one else shares.

### 15.5 - Other conflict types

* **both added** - file created in both branches -> merge the two files.
* **deleted by us / them** - one side deleted a file -> choose with
  `git add` / `git rm`.

---

## 16. Common Errors and Fixes

| Problem | Error Text | Fix |
| ------- | ---------- | --- |
| Remote has new commits | `non-fast-forward` | `git pull origin main` then push |
| Separate histories | `refusing to merge unrelated histories` | Push a locally committed repo into an **empty** GitHub repo |
| Wrong branch | you edited `main` | `git switch feature-login` |
| Merge left unfinished | `You have unmerged paths` | Fix marker, `git add`, `git commit` (or `git merge --abort`) |
| Pushed wrong branch | --- | `git reset --hard HEAD~1` (locally) and push correct branch |

---

## 17. Command Cheat Sheet

| Command | Purpose |
| ------- | ------- |
| `git init` | Create a local Git repository |
| `git status` | Show current state |
| `git add .` | Stage all changes |
| `git commit -m "msg"` | Create a commit |
| `git branch -M main` | Rename current branch to `main` |
| `git remote add origin URL` | Connect the local repo to GitHub |
| `git push -u origin main` | Push `main` and set upstream |
| `git clone URL folder` | Copy a remote repo locally |
| `git fetch` | Download remote info without merging |
| `git pull origin main` | Download and merge `main` |
| `git switch -c name` | Create and switch to a branch |
| `git merge origin/main` | Merge `main` into the current branch |
| `git merge --abort` | Cancel a merge and go back |
| `git checkout --ours/--theirs file` | Take one side for a whole file |
| `git log --oneline --graph --all` | View the history graph |

---

## 18. Checklist

- [ ] Project created and pushed to GitHub
- [ ] Developer B cloned the repo
- [ ] `git fetch` / `git pull` demonstrated
- [ ] Feature branch `feature-login` created
- [ ] Feature committed and pushed
- [ ] Developer A edited `main` (conflict created)
- [ ] Pull Request opened showing conflicts
- [ ] Conflict resolved on the command line
- [ ] `git add` + `git commit` completed the merge
- [ ] Branch pushed, PR becomes green
- [ ] Pull Request merged on GitHub
- [ ] Feature branch deleted
- [ ] `main` pulled and application tested
- [ ] Git history demonstrated with `git log --oneline --graph --all`

---

## Key Takeaway

A merge conflict is not a problem to be feared. It simply means two people
changed the **same part** of the same file. Fix the file, remove the markers,
commit, and push - then complete the merge.