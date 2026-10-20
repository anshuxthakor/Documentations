<div align="center">

# Git & GitHub Are Not What You Think

### *The history, collaboration, and recovery system behind modern software*

**Most developers think Git is just `git add`, `git commit`, and `git push`. This is for the ones who want to understand what Git is actually tracking — and how GitHub turns it into a team workflow.**

</div>

## What's Inside

- **Version Control** — Why developers need a history of their work
- **Git** — The distributed system that tracks project changes
- **GitHub** — The online platform built around Git
- **Git vs GitHub** — The difference beginners confuse most
- **Repositories** — The complete home of a project
- **Commits** — Saved snapshots with meaning
- **The Working Tree** — Where your changes exist before committing
- **The Staging Area** — Choosing exactly what belongs in a commit
- **Branches** — Parallel lines of development
- **Remotes** — Connecting local Git to online repositories
- **Push and Pull** — Moving commits between computers
- **Clone and Fork** — Two different kinds of copying
- **Pull Requests** — Reviewing changes before merging
- **Issues** — Tracking bugs, tasks, and ideas
- **Merge Conflicts** — When Git needs human judgment
- **GitHub Actions** — Automating tests and deployments
- **Recovery** — How Git helps you undo mistakes
- **Professional Workflow** — How teams actually use Git and GitHub

---

## 01 — Git Is a Time Machine for Your Project

Imagine working on a project and creating a new folder every time you make a change:

```txt
Portfolio
Portfolio New
Portfolio Latest
Portfolio Final
Portfolio Final New
Portfolio Final Real
Portfolio Final Real 2
Portfolio Final Real Latest
```

After a few days, nobody knows which folder is correct.

Git solves this problem by storing the history of your project inside one repository.

> Git is a distributed version control system that tracks changes to files over time.

Instead of creating complete copies of the project, Git stores meaningful snapshots called **commits**.

```txt
One project
    ├── Commit 1
    ├── Commit 2
    ├── Commit 3
    ├── Commit 4
    └── Commit 5
```

With Git, you can:

- Save versions of your project
- Compare changes
- Restore previous versions
- Create experimental branches
- Work with other developers
- Merge separate lines of work
- Recover deleted or overwritten files
- Understand who changed what and when

### The Time Machine Analogy

Imagine a document editor with complete version history:

```txt
Monday    → Login page
Tuesday   → Signup page
Wednesday → Password reset
Thursday  → Dark mode
```

If Thursday's change breaks the project, Git lets you inspect or restore the earlier state.

```txt
Current version
      ↓
Inspect history
      ↓
Find the working version
      ↓
Restore or revert
```

Git does not just save the latest version.

It remembers how you arrived there.

---

## 02 — Why Git Was Created

Before Git, large open-source projects needed a fast and distributed way to manage changes.

Git was created in **2005** by **Linus Torvalds**, the creator of Linux.

The Linux kernel was too large and active for an inefficient version control system. Developers needed a tool that was:

- Fast
- Distributed
- Reliable
- Secure
- Capable of handling large projects
- Suitable for thousands of contributors

Git was designed around the idea that every developer should have a complete local copy of the project and its history.

```txt
Developer A → complete repository + history
Developer B → complete repository + history
Developer C → complete repository + history
```

This means most Git operations do not require the internet.

You can create commits, inspect history, create branches, and compare changes while completely offline.

---

## 03 — Version Control Is More Than Backup

Version control records the evolution of a project.

It remembers:

- **Who** changed the project
- **When** the change happened
- **What** changed
- **Why** it changed
- **Which version** came before it
- **Which version** came after it

```txt
Version 1 → Homepage
Version 2 → About page
Version 3 → Contact page
Version 4 → Dark mode
Version 5 → Authentication
```

A version control system allows you to move through this history.

### Without Version Control

```txt
project-old.zip
project-new.zip
project-latest.zip
project-final.zip
project-final-2.zip
```

Problems:

- Full duplicate folders consume storage
- You cannot easily see what changed
- Collaboration becomes difficult
- Restoring one specific change is painful
- Nobody knows which copy is correct

### With Git

```txt
Repository
    ├── commit: create homepage
    ├── commit: add about page
    ├── commit: add contact form
    ├── commit: add dark mode
    └── commit: add authentication
```

One project.

A complete history.

Precise recovery.

---

## 04 — Local, Centralized, and Distributed Version Control

There are three major models of version control.

### Local Version Control

Everything exists on one computer:

```txt
Computer
  ├── Version 1
  ├── Version 2
  └── Version 3
```

The problem is obvious:

```txt
Laptop lost
    ↓
Project history lost
```

### Centralized Version Control

Everyone connects to one central server:

```txt
Developer A
      \
Developer B ─── Central Server
      /
Developer C
```

Examples include older systems such as:

- SVN
- CVS

The problem:

```txt
Central server unavailable
    ↓
Everyone is affected
```

### Distributed Version Control

Git gives every developer a complete repository:

```txt
Developer A → full project + full history
Developer B → full project + full history
Developer C → full project + full history
```

Even if the online server is unavailable, developers can continue working locally.

| Type | Where History Exists | Main Risk |
|---|---|---|
| Local VCS | One computer | Computer failure |
| Centralized VCS | One server | Server failure |
| Distributed VCS | Every developer's machine | Much less centralized risk |

GitHub adds online hosting and collaboration, but Git itself remains distributed.

---

## 05 — Git and GitHub Are Different Things

This is the most common beginner confusion.

| | Git | GitHub |
|---|---|---|
| What it is | Version control software | Online hosting and collaboration platform |
| Where it runs | On your computer | In the cloud |
| Internet required | No | Usually required for remote operations |
| Main purpose | Track history | Host, share, review, and collaborate |
| Created by | Linus Torvalds and contributors | GitHub founding team |
| Alternatives | Mercurial, SVN | GitLab, Bitbucket |

> Git is the engine that tracks your project.  
> GitHub is the online platform where you store and collaborate on Git repositories.

### Another Analogy

```txt
Git    → The technology for sending email
GitHub → A specific online service built around that technology
```

You can use Git without GitHub:

```bash
git init
git add .
git commit -m "Initial commit"
```

No account or internet connection is required.

You need GitHub when you want to:

- Back up your repository online
- Share your project
- Collaborate with a team
- Review changes
- Manage issues
- Run automated workflows
- Publish a public portfolio

---

## 06 — What Is a Repository?

A repository, or repo, is a project managed by Git.

A repository contains:

- Project files
- Git history
- Commits
- Branches
- Configuration
- Tags
- Remote connections
- Collaboration metadata

A repository may contain:

```txt
my-project/
  ├── src/
  ├── public/
  ├── package.json
  ├── README.md
  ├── .gitignore
  └── .git/
```

The hidden `.git` directory contains the local Git database.

It stores:

- Commit objects
- Branch references
- Configuration
- Staged snapshots
- Repository metadata

Do not manually edit `.git` unless you deeply understand what you are doing.

### Create a Repository Locally

```bash
mkdir my-project
cd my-project

git init
```

This creates a new `.git` directory.

```txt
my-project/
  └── .git/
```

Git is now watching the directory, but it has not saved any project snapshot yet.

### Create a Repository on GitHub

You can create a repository directly through GitHub.

A GitHub repository can be:

| Type | Visibility |
|---|---|
| Public | Anyone can view it |
| Private | Only you and permitted collaborators can view it |

Public repositories are commonly used for:

- Open-source projects
- Learning projects
- Developer portfolios
- Documentation
- Public tools

Private repositories are commonly used for:

- Company code
- Unreleased products
- Personal applications
- Sensitive projects
- Internal experiments

---

## 07 — The Three Areas of Git

Git work moves through three important areas:

```txt
Working Directory
      ↓ git add
Staging Area
      ↓ git commit
Repository History
```

### 1. Working Directory

This is the actual folder where you edit files.

```txt
You write code here.
You create files here.
You delete files here.
```

### 2. Staging Area

The staging area is where you choose which changes belong in the next commit.

```bash
git add index.html
```

This stages only `index.html`.

```bash
git add .
```

This stages all changes in the current directory.

### 3. Repository

The repository stores committed snapshots.

```bash
git commit -m "Add homepage"
```

This creates a permanent entry in Git history.

### Complete Flow

```txt
Edit files
    ↓
git status
    ↓
git add
    ↓
git diff --staged
    ↓
git commit
    ↓
git log
```

A commit only contains staged changes.

That is why the staging area matters.

---

## 08 — The Basic Git Workflow

A normal local workflow looks like this:

```bash
# 1. Check the current state
git status

# 2. See unstaged changes
git diff

# 3. Stage changes
git add .

# 4. Inspect staged changes
git diff --staged

# 5. Create a commit
git commit -m "Add contact form"

# 6. Inspect history
git log
```

### `git status`

```bash
git status
```

This tells you:

- Which branch you are on
- Which files are modified
- Which files are staged
- Which files are untracked
- Whether your branch is ahead or behind a remote

Use it frequently.

```txt
git status is not an emergency command.
It is a navigation command.
```

### `git diff`

```bash
git diff
```

Shows changes that are not staged yet.

### `git diff --staged`

```bash
git diff --staged
```

Shows what will enter the next commit.

### `git commit`

```bash
git commit -m "Add password reset form"
```

A commit should describe the change clearly.

Good commit messages:

```txt
Add login form validation
Fix mobile navbar overflow
Create product card component
Update README setup instructions
```

Weak commit messages:

```txt
changes
update
fix
new
final
```

A commit message is part of the project's history.

Write it for the person who will read it later.

---

## 09 — What Is a Commit?

A commit is a saved snapshot of the staged project.

```txt
Working changes
      ↓ git add
Staged changes
      ↓ git commit
Saved snapshot
```

A commit records:

- The changed files
- The content of those files
- The author
- The commit message
- The timestamp
- The parent commit

Commits form a history:

```txt
Commit A
   ↓
Commit B
   ↓
Commit C
   ↓
Commit D
```

### View Commit History

```bash
git log
```

A compact version:

```bash
git log --oneline
```

Example:

```txt
d91abc2 Add contact form
2a7bc18 Create navbar
8c32f10 Initial commit
```

Each commit has a unique identifier.

You can inspect a commit:

```bash
git show d91abc2
```

### Good Commits Are Focused

Avoid combining unrelated work:

```txt
Add authentication
Fix navbar
Change button colors
Rewrite README
```

This is difficult to review and difficult to undo.

Prefer separate commits:

```txt
Add authentication flow
Fix navbar overflow
Update button colors
Rewrite README setup instructions
```

Focused commits create a clearer project history.

---

## 10 — Tracked, Untracked, and Ignored Files

Git distinguishes between different kinds of files.

### Untracked File

A new file that Git does not know about yet:

```txt
?? app.js
```

Stage it:

```bash
git add app.js
```

### Modified File

A tracked file changed after the last commit:

```txt
 M app.js
```

Stage it:

```bash
git add app.js
```

### Staged File

A file selected for the next commit:

```txt
M  app.js
```

Commit it:

```bash
git commit -m "Update app logic"
```

### Ignored File

A file intentionally excluded by `.gitignore`.

Example `.gitignore`:

```gitignore
node_modules/
.env
dist/
build/
.DS_Store
*.log
```

These files remain on your computer but are not tracked by Git.

### Never Commit Secrets

Do not commit:

```txt
.env
.env.local
private keys
API secrets
database passwords
access tokens
```

If a secret is accidentally committed, deleting it from the latest file is not always enough.

It may still exist in Git history.

Rotate the secret immediately and remove it from history when necessary.

---

## 11 — The `.gitignore` File

A `.gitignore` file tells Git which files should not be tracked.

Example:

```gitignore
# Dependencies
node_modules/

# Build output
dist/
build/

# Environment files
.env
.env.local
.env.production

# Logs
*.log

# Operating system files
.DS_Store
Thumbs.db

# Editor files
.vscode/
.idea/
```

### Why Ignore `node_modules`?

Dependencies can contain thousands of files and should be recreated from:

```txt
package.json
package-lock.json
```

Instead of committing the dependency folder, commit the dependency manifest.

```bash
npm install
```

recreates the dependencies later.

### Why Ignore `.env`?

Environment files often contain:

```txt
API keys
Database URLs
Authentication secrets
Private configuration
```

Commit an example file instead:

```txt
.env.example
```

```env
API_URL=
DATABASE_URL=
```

This documents what variables are required without exposing the real values.

---

## 12 — Branches Are Parallel Timelines

A branch is an independent line of development.

```txt
main
  ├── commit A
  ├── commit B
  └── commit C

feature/login
  └── commit D
  └── commit E
```

The main branch remains stable while the feature branch changes.

### Create a Branch

```bash
git branch feature/login
```

Switch to it:

```bash
git switch feature/login
```

Create and switch in one command:

```bash
git switch -c feature/login
```

Older syntax:

```bash
git checkout -b feature/login
```

### List Branches

```bash
git branch
```

List local and remote branches:

```bash
git branch -a
```

### Rename a Branch

```bash
git branch -m old-name new-name
```

Rename the current branch:

```bash
git branch -m new-name
```

### Delete a Branch

```bash
git branch -d feature/login
```

Force delete an unmerged branch:

```bash
git branch -D feature/login
```

Use force deletion carefully.

### Branch Naming

Good names:

```txt
feature/login-form
feature/product-search
fix/navbar-overflow
docs/update-readme
refactor/api-client
```

Branch names should communicate intent.

---

## 13 — Why Branches Matter

Without branches, every change is made directly on the main line:

```txt
main → unstable feature → broken main → emergency fix
```

With branches:

```txt
main ──────────────── stable
  \
   feature/login ──── experiment
```

The feature can be:

- Developed independently
- Tested independently
- Reviewed independently
- Merged when ready
- Deleted if abandoned

Branches are not separate copies of the entire project in the way beginners often imagine.

They are references to commits.

Creating a branch is usually fast because Git stores a lightweight pointer.

---

## 14 — Merging Branches

Suppose you are on the `main` branch:

```bash
git switch main
```

Update it first:

```bash
git pull
```

Merge the feature:

```bash
git merge feature/login
```

Git combines the history of the two branches.

### Fast-Forward Merge

If main has not changed since the feature branch was created:

```txt
main:    A ── B
              \
feature:       C ── D
```

Git may simply move the main pointer forward:

```txt
main:    A ── B ── C ── D
```

### Merge Commit

If both branches changed independently:

```txt
main:    A ── B ── C
              \
feature:       D ── E
```

Git may create a merge commit:

```txt
main:    A ── B ── C ── M
              \       /
feature:       D ── E
```

The merge commit combines both histories.

### Delete the Feature Branch

After merging:

```bash
git branch -d feature/login
```

The branch can be deleted because its commits remain in history.

---

## 15 — Merge Conflicts Are Human Decisions

Git can merge many changes automatically.

A conflict occurs when Git cannot safely decide which version to keep.

Example:

```txt
main changed the same line
feature changed the same line
```

Git marks the conflict:

```txt
<<<<<<< HEAD
Code from the current branch
=======
Code from the incoming branch
>>>>>>> feature/login
```

You must manually edit the file and choose the correct result.

Remove the conflict markers:

```txt
<<<<<<<
=======
>>>>>>>
```

Then stage the resolved file:

```bash
git add conflicted-file.js
```

Complete the merge:

```bash
git commit
```

### Conflict Workflow

```bash
git status

# Open conflicted files
# Decide what the final code should be

git add file.js
git commit
```

### Abort a Merge

If you want to cancel the merge:

```bash
git merge --abort
```

This returns the working tree to its state before the merge began.

> A merge conflict is not a Git failure.  
> It means Git needs a developer to make a semantic decision.

Git can compare text.

It cannot always understand intention.

---

## 16 — Remotes Connect Local Git to GitHub

A remote is a name for another copy of the repository.

Most GitHub repositories use a remote named:

```txt
origin
```

View remotes:

```bash
git remote -v
```

Example:

```txt
origin  git@github.com:user/project.git (fetch)
origin  git@github.com:user/project.git (push)
```

Add a remote:

```bash
git remote add origin https://github.com/user/project.git
```

Or with SSH:

```bash
git remote add origin git@github.com:user/project.git
```

Change a remote:

```bash
git remote set-url origin NEW_URL
```

Remove a remote:

```bash
git remote remove origin
```

A remote does not automatically synchronize your files.

It only gives Git a location where commits can be exchanged.

---

## 17 — GitHub Is Git's Online Home

GitHub is a cloud-based platform for hosting Git repositories.

It adds features around Git:

- Repository hosting
- Access control
- Code browsing
- Pull Requests
- Issues
- Discussions
- Project boards
- Code review
- GitHub Actions
- Releases
- Packages
- Developer profiles

Think of the relationship like this:

```txt
Git:
  tracks the project locally

GitHub:
  hosts the project online and coordinates people
```

### Why Developers Use GitHub

1. **Backup** — Protects code from local computer failure
2. **Collaboration** — Allows multiple developers to work together
3. **Code review** — Pull Requests create a review process
4. **Open source** — Makes projects accessible to the public
5. **Portfolio** — Shows your work to others
6. **Issue tracking** — Organizes bugs and tasks
7. **Automation** — Runs tests and deployments
8. **Documentation** — Stores READMEs and project notes

---

## 18 — Clone Downloads a Repository

To copy a repository from GitHub to your computer:

```bash
git clone https://github.com/user/project.git
```

This downloads:

- Project files
- Commit history
- Branch references
- Remote configuration

Git automatically names the remote:

```txt
origin
```

### Clone Into a Custom Folder

```bash
git clone \
  https://github.com/user/project.git \
  my-project
```

### After Cloning

```bash
cd my-project
git remote -v
git status
```

The normal workflow becomes:

```txt
Clone
  ↓
Create a branch
  ↓
Make changes
  ↓
Commit
  ↓
Push
  ↓
Open a Pull Request
```

### Clone vs Fork

These are different operations.

| | Clone | Fork |
|---|---|---|
| What it creates | Local copy on your computer | GitHub copy under your account |
| Where it exists | Your machine | GitHub |
| Common use | Start working locally | Contribute to a repository you do not own |
| Changes account ownership? | No | Yes |

You may fork a repository and then clone your fork locally.

---

## 19 — Push Uploads Commits to GitHub

After creating local commits, upload them:

```bash
git push origin main
```

The structure is:

```txt
git push <remote> <branch>
```

For a new branch:

```bash
git push -u origin feature/login
```

The `-u` option sets the upstream relationship.

After that, you can usually run:

```bash
git push
```

### Push Flow

```txt
Local commit
      ↓
git push
      ↓
GitHub receives commit
      ↓
Remote branch updates
```

Important:

```txt
git push sends commits.
It does not send uncommitted changes.
```

You must commit first:

```bash
git add .
git commit -m "Add login form"
git push
```

---

## 20 — Pull Downloads and Integrates Changes

To get changes from GitHub:

```bash
git pull
```

Conceptually, `git pull` performs:

```txt
git fetch
    +
git merge
```

It downloads remote commits and integrates them into the current branch.

### Safer Inspection With `fetch`

```bash
git fetch origin
```

This downloads remote updates without changing your current files.

Inspect remote branches:

```bash
git log origin/main
```

Then integrate manually:

```bash
git merge origin/main
```

### Pull Workflow

```bash
git switch main
git pull origin main
```

Before starting new work, update your local main branch.

```txt
Old local main
      ↓ git pull
Current remote main
      ↓
Create a new feature branch
```

---

## 21 — The Complete GitHub Workflow

A common feature workflow:

```bash
# Clone the repository
git clone https://github.com/user/project.git
cd project

# Create a feature branch
git switch -c feature/login

# Make changes
# Edit files

# Review changes
git status
git diff

# Stage changes
git add .

# Commit changes
git commit -m "Add login form"

# Push the feature branch
git push -u origin feature/login
```

Then on GitHub:

```txt
Open Pull Request
    ↓
Review code
    ↓
Run checks
    ↓
Discuss changes
    ↓
Make corrections if necessary
    ↓
Merge into main
```

After merging locally:

```bash
git switch main
git pull
git branch -d feature/login
```

This workflow keeps the main branch cleaner and makes changes reviewable.

---

## 22 — Pull Requests Are Proposed Changes

A Pull Request, or PR, is a request to merge changes from one branch into another.

```txt
feature/login
      ↓ Pull Request
main
```

A Pull Request is not simply a button for merging.

It provides a place to:

- Review changed files
- Discuss implementation
- Suggest improvements
- Run automated checks
- Connect issues
- Track review approval
- Confirm the final result

### Pull Request Flow

```txt
Create branch
      ↓
Make changes
      ↓
Push branch
      ↓
Open Pull Request
      ↓
Review changes
      ↓
Run tests
      ↓
Fix requested changes
      ↓
Approve
      ↓
Merge
```

### Good Pull Requests

A good PR should have:

- A clear title
- A useful description
- One focused purpose
- Small, reviewable changes
- Tests when appropriate
- Screenshots for UI changes
- Links to related issues
- No unrelated formatting changes

Weak Pull Request:

```txt
Update stuff
```

Better Pull Request:

```txt
Add product search with debounced API requests
```

### Pull Request Review Questions

Reviewers often ask:

- Does the code solve the intended problem?
- Is the implementation easy to understand?
- Does it introduce a bug?
- Are edge cases handled?
- Are loading and error states represented?
- Are tests included?
- Does the change affect performance?
- Does the code follow project conventions?

---

## 23 — Issues Track Work

An Issue is a tracked piece of work.

Issues can represent:

- Bugs
- Features
- Questions
- Documentation tasks
- Refactoring
- Improvements
- Investigations

Example issue:

```txt
Title:
Navbar overflows on mobile screens

Description:
The navigation links extend outside the viewport below 400px.
Add a responsive menu or allow the links to wrap.
```

Issues can have:

- Labels
- Assignees
- Milestones
- Projects
- Comments
- Relationships
- Linked Pull Requests

### Issue vs Pull Request

| | Issue | Pull Request |
|---|---|---|
| Purpose | Describe work or a problem | Propose code changes |
| Contains code changes | Not necessarily | Usually yes |
| Main question | What should be done? | Should this change be merged? |
| Workflow | Plan and track | Review and merge |

A Pull Request can close an Issue automatically when its description includes:

```txt
Closes #42
```

---

## 24 — Forks Enable Open-Source Contributions

A fork is your own GitHub copy of someone else's repository.

```txt
Original repository
        ↓ fork
Your GitHub repository
        ↓ clone
Your computer
```

Typical open-source workflow:

```txt
1. Fork the repository
2. Clone your fork
3. Create a feature branch
4. Make changes
5. Commit changes
6. Push to your fork
7. Open a Pull Request to the original repository
```

Add the original repository as another remote:

```bash
git remote add upstream \
  https://github.com/original-owner/project.git
```

View remotes:

```bash
git remote -v
```

You may see:

```txt
origin    → your fork
upstream  → original repository
```

Update your fork's local main branch:

```bash
git fetch upstream
git switch main
git merge upstream/main
git push origin main
```

### Fork vs Branch

A branch exists inside a repository:

```txt
one repository
  ├── main
  └── feature
```

A fork is a separate GitHub repository:

```txt
original/project
your-account/project
```

Use branches for work inside a project you control.

Use forks when contributing to a project owned by someone else.

---

## 25 — GitHub Profiles Can Become Public Portfolios

A GitHub profile can communicate how you work.

A strong repository may include:

```txt
README.md
Screenshots
Installation steps
Feature list
Technology list
Usage instructions
Project structure
Deployment link
License
```

Recruiters and collaborators may look at:

- Repository quality
- Commit history
- Project organization
- Documentation
- Pull Requests
- Issue discussions
- Consistency of contributions

### A Useful README Structure

```markdown
# Project Name

Short explanation of what the project does.

## Features

- Feature one
- Feature two
- Feature three

## Tech Stack

- React
- TypeScript
- Node.js

## Installation

```bash
npm install
npm run dev
```

## Screenshots

Add screenshots here.

## License

MIT
```

Public repositories are not automatically good portfolios.

A clean, understandable project is more valuable than a large but confusing one.

---

## 26 — GitHub Actions Automate Repetitive Work

GitHub Actions can run workflows automatically.

Common uses:

- Running tests
- Checking formatting
- Running linters
- Building projects
- Deploying applications
- Publishing packages
- Creating releases

A workflow file usually lives inside:

```txt
.github/workflows/
```

Example:

```yaml
name: Continuous Integration

on:
  push:
    branches:
      - main

  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 22

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Run build
        run: npm run build
```

The workflow is triggered by events such as:

```txt
push
pull_request
workflow_dispatch
release
schedule
```

### Continuous Integration

```txt
Developer pushes code
      ↓
GitHub Actions starts
      ↓
Install dependencies
      ↓
Run lint
      ↓
Run tests
      ↓
Build project
      ↓
Report success or failure
```

Automated checks prevent broken code from being merged unnoticed.

---

## 27 — Git Tags Mark Important Versions

A tag is a name attached to a specific commit.

Tags are useful for:

- Releases
- Versions
- Milestones
- Deployments
- Stable snapshots

Create a tag:

```bash
git tag v1.0.0
```

Push it:

```bash
git push origin v1.0.0
```

Push all tags:

```bash
git push origin --tags
```

List tags:

```bash
git tag
```

Annotated tags contain additional metadata:

```bash
git tag -a v1.0.0 \
  -m "First stable release"
```

A tag is different from a branch.

| | Branch | Tag |
|---|---|---|
| Purpose | Moving line of development | Fixed reference |
| Changes over time | Yes | No |
| Common use | Features and work | Releases and versions |

---

## 28 — Git Can Undo Mistakes

Git is powerful because it remembers previous states.

But different commands undo different things.

### Restore Unstaged Changes

```bash
git restore file.js
```

This discards changes in the working directory.

Be careful: the changes may be lost.

### Unstage a File

```bash
git restore --staged file.js
```

This removes the file from the staging area but keeps the changes in your working directory.

### Amend the Latest Commit

If you forgot a file:

```bash
git add forgotten-file.js
git commit --amend
```

Or update the message:

```bash
git commit --amend -m "Correct commit message"
```

Do not amend commits that have already been shared unless you understand the consequences.

### Revert a Public Commit

```bash
git revert <commit-id>
```

This creates a new commit that reverses the old commit.

This is usually safer for shared branches because it does not rewrite existing history.

### Reset Local History

```bash
git reset --soft HEAD~1
```

Moves the branch pointer back while keeping changes staged.

```bash
git reset --mixed HEAD~1
```

Moves the branch pointer back and unstages changes.

```bash
git reset --hard HEAD~1
```

Moves the branch pointer back and discards changes.

`--hard` can permanently remove work.

Use it carefully.

### Recovery Principle

```txt
Shared history:
  prefer git revert

Private local history:
  reset may be acceptable

Uncommitted work:
  restore or stash carefully
```

---

## 29 — Stashing Temporarily Saves Uncommitted Work

Sometimes you have unfinished work but need to switch branches.

Use:

```bash
git stash
```

This temporarily stores uncommitted changes.

Now you can switch branches:

```bash
git switch main
```

Return later:

```bash
git switch feature/login
git stash pop
```

List stashes:

```bash
git stash list
```

Create a named stash:

```bash
git stash push -m "unfinished login form"
```

Apply without removing it:

```bash
git stash apply
```

Delete a stash:

```bash
git stash drop
```

Stash is useful, but it should not become a permanent storage system.

If the work matters, create a proper commit on a temporary branch.

---

## 30 — Rebase Replays Your Work on a New Base

Suppose your feature branch started from an old main branch:

```txt
main:    A ── B ── C
              \
feature:       D ── E
```

Meanwhile, main advanced:

```txt
main:    A ── B ── C ── F ── G
              \
feature:       D ── E
```

Rebase replays your feature commits on top of the latest main:

```txt
main:    A ── B ── C ── F ── G
                              \
feature:                       D' ── E'
```

Command:

```bash
git switch feature/login
git fetch origin
git rebase origin/main
```

Rebase creates new commit identifiers because the commits are replayed.

### Rebase vs Merge

| | Merge | Rebase |
|---|---|---|
| History | Preserves branching | Creates a cleaner line |
| New merge commit | Sometimes | No |
| Rewrites commits | No | Yes |
| Safe for shared branches | Usually | Be careful |
| Common use | Integrating branches | Updating private feature branches |

Do not rebase shared history unless your team agrees.

---

## 31 — Git Worktrees Allow Multiple Checkouts

A worktree lets you check out multiple branches into separate directories from one repository.

```bash
git worktree add ../project-main main
git worktree add ../project-login feature/login
```

Now you can work on:

```txt
project-main/
project-login/
```

without repeatedly switching branches in one folder.

List worktrees:

```bash
git worktree list
```

Remove one:

```bash
git worktree remove ../project-login
```

Worktrees are useful when:

- Reviewing a Pull Request while keeping current work open
- Running two branches side by side
- Testing a release branch
- Comparing implementations
- Working on multiple tasks simultaneously

---

## 32 — A Professional Branch Workflow

A simple team workflow:

```txt
main
  └── always stable

feature/*
  └── new functionality

fix/*
  └── bug fixes

docs/*
  └── documentation changes

refactor/*
  └── internal improvements
```

Example:

```bash
git switch main
git pull origin main

git switch -c feature/search
```

Work and commit:

```bash
git add .
git commit -m "Add product search"
```

Push:

```bash
git push -u origin feature/search
```

Open a Pull Request.

After the PR is merged:

```bash
git switch main
git pull origin main
git branch -d feature/search
```

### Keep Branches Focused

One branch should ideally represent one logical change.

Avoid branches that contain:

```txt
New feature
Unrelated bug fix
Large formatting rewrite
Dependency upgrade
README rewrite
```

Separate work is easier to review, merge, and revert.

---

## 33 — Common Beginner Mistakes

### Mistake 1: Confusing Git and GitHub

```txt
Git    → local version control
GitHub → online hosting and collaboration
```

### Mistake 2: Committing Everything Blindly

```bash
git add .
```

is convenient, but inspect what you are staging:

```bash
git status
git diff --staged
```

### Mistake 3: Committing Secrets

Never commit:

```txt
.env
private keys
passwords
tokens
```

### Mistake 4: Working Directly on Main

Use a branch for feature work:

```bash
git switch -c feature/name
```

### Mistake 5: Writing Meaningless Commit Messages

Avoid:

```txt
update
changes
final
fix
```

Write:

```txt
Handle empty search results
Add responsive mobile navigation
Validate registration form fields
```

### Mistake 6: Pulling With Uncommitted Work

Before pulling, check:

```bash
git status
```

Commit or stash work when necessary.

### Mistake 7: Using Force Push Carelessly

```bash
git push --force
```

can overwrite remote history.

Prefer:

```bash
git push --force-with-lease
```

when force pushing is genuinely necessary.

### Mistake 8: Treating a Merge Conflict as a Technical Failure

A conflict means Git needs you to decide which code is correct.

Read the conflicting file.

Understand both changes.

Then resolve it intentionally.

---

## 34 — Git and GitHub Command Reference

### Repository Setup

```bash
git init
git clone <url>
git remote -v
git remote add origin <url>
```

### Daily Status

```bash
git status
git diff
git diff --staged
git log
git log --oneline
```

### Staging and Commits

```bash
git add file.js
git add .
git restore file.js
git restore --staged file.js
git commit -m "Message"
git commit --amend
```

### Branches

```bash
git branch
git branch -a
git switch main
git switch -c feature/name
git branch -d feature/name
git branch -m new-name
```

### Remotes

```bash
git fetch origin
git pull origin main
git push origin main
git push -u origin feature/name
git remote get-url origin
```

### Merging and Rebasing

```bash
git merge feature/name
git merge --abort
git rebase main
git rebase --abort
git rebase --continue
```

### Temporary Work

```bash
git stash
git stash list
git stash pop
git stash apply
```

### Recovery

```bash
git revert <commit>
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
git reflog
```

### Tags

```bash
git tag
git tag v1.0.0
git push origin v1.0.0
git push origin --tags
```

---

## 35 — Interview Questions and Answers

### What is Git?

Git is a distributed version control system that tracks changes in files and stores project history.

### What is GitHub?

GitHub is a cloud-based platform for hosting Git repositories and enabling collaboration around them.

### What is the difference between Git and GitHub?

Git is local version control software.

GitHub is an online service that hosts Git repositories and provides collaboration features.

### Who created Git?

Git was created by Linus Torvalds in 2005.

### What is a repository?

A repository is a project managed by Git, including its files and history.

### What is a commit?

A commit is a saved snapshot of staged project changes.

### What is a branch?

A branch is an independent line of development.

### What is a remote?

A remote is a reference to another copy of a repository, usually hosted online.

### What does `git push` do?

It uploads local commits to a remote repository.

### What does `git pull` do?

It downloads remote commits and integrates them into the current branch.

### What is a Pull Request?

A Pull Request is a proposal to merge changes from one branch into another, usually with review.

### What is a fork?

A fork is a GitHub copy of another user's repository under your own account.

### What is the difference between fork and clone?

A fork creates a repository copy on GitHub.

A clone downloads a repository to your computer.

### Can Git work without GitHub?

Yes. Git works completely offline.

### Does GitHub replace Git?

No. GitHub is built around Git; it does not replace the local version control system.

### What is a merge conflict?

A merge conflict happens when Git cannot automatically combine competing changes.

---

## 36 — Practice Questions

### Beginner

1. What is Git?
2. What is GitHub?
3. What is version control?
4. What is a repository?
5. What is a commit?
6. What is a branch?
7. What is a remote?
8. What does `git status` show?
9. What does `git add` do?
10. What does `git commit` do?

### Intermediate

1. Explain the difference between Git and GitHub.
2. Explain the working directory, staging area, and repository.
3. What is the difference between `git fetch` and `git pull`?
4. What is the difference between `git merge` and `git rebase`?
5. Why are branches useful?
6. What happens during a merge conflict?
7. What is the difference between a fork and a clone?
8. Why should secrets not be committed?
9. What does `.gitignore` do?
10. Why should commits be focused?

### Think and Answer

> You have worked locally for six months without pushing to GitHub. Your laptop is stolen. Which parts of your work can Git recover, and which parts are lost?

### Another Scenario

> Two developers changed the same line in a file. Why can Git not automatically choose the correct version?

---

## 37 — Mini Activity: Your First Git Repository

Create a folder:

```txt
Git Practice
```

Inside it, create:

```txt
index.html
style.css
script.js
```

Initialize Git:

```bash
cd "Git Practice"
git init
```

Create your first snapshot:

```bash
git add .
git commit -m "Initial project setup"
```

Now make a change to `index.html`.

Check the status:

```bash
git status
```

View the difference:

```bash
git diff
```

Stage and commit it:

```bash
git add index.html
git commit -m "Add homepage content"
```

Create a branch:

```bash
git switch -c feature/dark-mode
```

Update the CSS and commit:

```bash
git add style.css
git commit -m "Add dark mode styles"
```

Return to main:

```bash
git switch main
```

Merge the branch:

```bash
git merge feature/dark-mode
```

View the complete history:

```bash
git log --oneline --graph --all
```

This small activity demonstrates:

```txt
Initialize
  ↓
Stage
  ↓
Commit
  ↓
Branch
  ↓
Change
  ↓
Merge
  ↓
Inspect history
```

---

## 38 — Mini Activity: Connect Git to GitHub

Create a repository on GitHub called:

```txt
git-practice
```

Inside your local project:

```bash
git remote add origin \
  https://github.com/your-username/git-practice.git
```

Rename your branch if necessary:

```bash
git branch -M main
```

Push the project:

```bash
git push -u origin main
```

Now create a feature branch:

```bash
git switch -c feature/readme
```

Add a `README.md` file:

```markdown
# Git Practice

A small repository used to practice Git and GitHub workflows.
```

Commit it:

```bash
git add README.md
git commit -m "Add project README"
```

Push the feature branch:

```bash
git push -u origin feature/readme
```

Open GitHub and create a Pull Request.

```txt
Local branch
      ↓ push
GitHub branch
      ↓ Pull Request
main branch
      ↓ review and merge
```

This is the foundation of professional GitHub collaboration.

---

## Quick Summary

```txt
Git:
  Distributed version control system
  Tracks files and project history locally

GitHub:
  Online platform for hosting Git repositories
  Adds collaboration, review, issues, and automation

Repository:
  Project + complete Git history

Working directory:
  Files you are currently editing

Staging area:
  Changes selected for the next commit

Commit:
  Saved snapshot of staged changes

Branch:
  Independent line of development

Remote:
  Another copy of the repository, usually online

Clone:
  Downloads a repository to your computer

Fork:
  Creates a GitHub copy under your account

Push:
  Uploads local commits

Pull:
  Downloads and integrates remote commits

Fetch:
  Downloads remote information without changing your files

Pull Request:
  Proposal to merge changes after review

Issue:
  Tracked bug, task, question, or feature request

Merge:
  Combines branch histories

Conflict:
  Git cannot safely choose between competing changes

Revert:
  Creates a new commit that undoes an earlier commit

Reset:
  Moves a branch pointer and may rewrite local history

Stash:
  Temporarily stores uncommitted work

Tag:
  Fixed name attached to an important commit

GitHub Actions:
  Automated workflows for tests, builds, and deployments
```

---

<div align="center">

## The History Is the Product

Most developers think Git is a collection of commands to memorize.

The developers who understand Git know that it is really a system for preserving decisions.

A commit is not just a save button.

It is an explanation of what changed.

A branch is not just another folder.

It is a safe place to explore an idea.

A Pull Request is not just a merge screen.

It is a conversation about whether a change belongs in the project.

GitHub is not just a place to upload code.

It is where code becomes reviewable, shareable, reproducible, and collaborative.

**Do not just learn how to push code.**

**Learn how to make your project's history understandable.**

**Go be one of those.**

</div>