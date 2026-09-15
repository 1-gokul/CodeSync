# CodeSync | Automated GitHub Code Synchronization & Version Control System

CodeSync is a native Python desktop application that automates local workspace synchronization with remote GitHub repositories. It combines local Git version control and the GitHub API into a single dashboard, so you don't run `git add`, `git commit`, `git push` by hand every time.

## Table of Contents

- [Features](#features)
- [Technical Architecture](#technical-architecture)
- [Technologies Used](#technologies-used)
- [Installation](#installation)
- [Environment Setup](#environment-setup)
- [Running the Application](#running-the-application)
- [Complete Usage Workflow](#complete-usage-workflow)
- [Project Structure](#project-structure)
- [Safety Rules](#safety-rules)

---

## Features

- **Project Workspace Integration**: Monitor any local project folder for changes.
- **Git State Discovery**: Automatically scans workspaces for Modified (`M`), Staged/Added (`A`), Deleted (`D`), and Untracked (`??`) files.
- **Auto-generated Commit Messages**: Uses a rule-based classification algorithm (not AI/ML) to analyze modified files and suggest a commit message.
- **Interactive Repository Connection**: Connect an existing GitHub repository, or create a brand-new one on GitHub with a default `main` branch.
- **MongoDB Synchronization History**: Archives every sync operation — files changed count, commit message, timestamp, and any error trace.
- **Background Scheduler Daemon**: Automates syncing on an interval (15 min, 30 min, 1 hour, 6 hours, daily) using a non-blocking background thread.
- **Secure Token Authentication**: Supports GitHub Personal Access Token auth, with the token masked everywhere it could appear — logs and GUI input.
- **Non-Freezing UI**: Disk I/O, network calls, and Git operations all run on background worker threads, so the GUI never locks up mid-sync.

---

## Technical Architecture

```text
               +-------------------------------------------------+
               |                   CodeSync GUI                  |
               |  (Dashboard, Repositories, History, Scheduler)   |
               +-------------------------------------------------+
                                        |
                                        v
               +-------------------------------------------------+
               |                   Sync Manager                  |
               +-------------------------------------------------+
                     /                  |                  \
                    /                   |                   \
                   v                    v                    v
       +-----------------+     +-----------------+     +-----------------+
       |   Git Manager   |     |  GitHub Manager  |     | Database Manager|
       |  (GitPython CLI)|     |    (PyGithub)    |     |    (PyMongo)    |
       +-----------------+     +-----------------+     +-----------------+
                |                       |                       |
                v                       v                       v
       [ Local Git Repo ]      [ GitHub Cloud API ]    [ MongoDB — local or Atlas ]
```

Every database call in the app goes through `DatabaseManager` — no other module runs a raw query. That single choke point is what makes swapping the storage backend (this project moved from SQLite to MongoDB) a contained, low-risk change.

---

## Technologies Used

| Library | Role |
|---|---|
| **Python 3.11+** | Base runtime |
| **CustomTkinter** | Desktop GUI framework |
| **GitPython** | Local Git operations (detect, stage, commit, branch) |
| **PyGithub** | GitHub REST API v3 client — auth and repo management |
| **PyMongo** | MongoDB driver |
| **MongoDB** | Stores tracked repositories, sync history, and settings |
| **python-dotenv** | Loads credentials/config from `.env` |
| **schedule** | Powers the background interval scheduler |
| **Pillow** | GUI image asset handling |

---

## Installation

### 1. Prerequisites

**Python** — Install 3.11 or newer from [python.org](https://www.python.org/downloads/). On Windows, check **"Add Python to PATH"** during install.

**Git** — CodeSync shells out to the Git CLI, so it must be installed separately:
- Windows: [git-scm.com](https://git-scm.com/download/win)
- Mac: `brew install git`
- Linux: `sudo apt install git`

**MongoDB** — pick one:
- **Local install**: [MongoDB Community Server](https://www.mongodb.com/try/download/community). Make sure `mongod` is running before you launch CodeSync.
- **MongoDB Atlas** (no local install): create a free cluster at [mongodb.com/cloud/atlas](https://www.mongodb.com/cloud/atlas) and copy its connection string for the next step.

### 2. Project Setup

```bash
pip install -r requirements.txt
```

---

## Environment Setup

1. Copy the template:
   ```bash
   cp .env.example .env
   ```
2. Fill in `.env`:
   ```text
   GITHUB_TOKEN=ghp_yourpersonaltokenhere
   GITHUB_USERNAME=yourusername
   DEFAULT_BRANCH=main
   DEFAULT_COMMIT_MESSAGE=Update project files via CodeSync

   MONGO_URI=mongodb://localhost:27017
   MONGO_DB_NAME=codesync
   ```
   Use `mongodb://localhost:27017` for a local install, or your Atlas connection string (starts with `mongodb+srv://`) if using Atlas.

### Creating a GitHub Personal Access Token (PAT)

1. GitHub profile → **Settings** → **Developer Settings** → **Personal Access Tokens** → **Tokens (classic)**.
2. **Generate new token (classic)**.
3. Set an expiration, name it (e.g. `CodeSync App`), and grant the **`repo`** scope (full control of private and public repositories).
4. Generate, copy it immediately, and paste it into either the app's **Settings** panel or your `.env` file. GitHub won't show it to you again.

---

## Running the Application

```bash
python main.py
```

---

## Complete Usage Workflow

1. **Authenticate**: Settings → paste GitHub username + PAT → **Test GitHub Connection** → **Save Settings**.
2. **Track a project**: Repositories → **Add Repository** → pick a local folder → **Connect Existing Repo** (paste its GitHub URL) or **Create New Repo** → **Add Project**.
3. **Sync manually**: Dashboard → pick the project from **Active Project** → review the changed-files table → **Generate Message** (or type your own) → **Sync to GitHub**.
4. **Automate it**: Scheduler → toggle **Automatic Sync** on → pick an interval and a repository → **Save Scheduler Configuration**.
5. **Audit**: History tab shows every past sync — double-click a row for full debug detail on that run.

---

## Project Structure

```text
CodeSync/
│
├── main.py                     # App bootstrap loader
├── config.py                   # Configuration parser and file system paths resolver
├── requirements.txt            # Python environment packages
├── README.md                   # This file
├── .env.example                # Configuration template
├── .gitignore                  # Git tracking exclusion list
│
├── core/
│   ├── git_manager.py          # Wrapper for GitPython operations
│   ├── github_manager.py       # Wrapper for PyGithub cloud operations
│   ├── sync_manager.py         # End-to-end sync coordinator
│   └── scheduler.py            # Non-blocking scheduler thread engine
│
├── database/
│   └── database_manager.py     # MongoDB collections and query methods
│
├── gui/
│   ├── app.py                  # Main window shell and sidebar navigation
│   ├── dashboard.py            # Sync workspace panel
│   ├── repositories.py         # Project configurations panel
│   ├── settings.py             # User preferences panel
│   ├── history.py              # Sync history log panel
│   ├── scheduler.py            # Automation panel
│   └── about.py                # App metadata panel
│
├── utils/
│   ├── logger.py                # Logger with token masking
│   ├── validators.py            # Form input verification helpers
│   └── helpers.py                # Rule-based commit message generator
│
├── tests/
│   ├── test_git.py
│   ├── test_github.py
│   ├── test_database.py
│   └── test_utils.py
│
└── logs/
    └── automation.log          # Application execution log (Git-ignored)
```

Data lives in MongoDB (local `mongod` instance or Atlas) — there's no local `.db` file.

---

## Safety Rules

- CodeSync **never** force-pushes (`git push --force`) by default — remote history is never silently overwritten by a push.
- Project folder deletion is never automated.
- `.env` (including your Mongo connection string and GitHub token) is Git-ignored by default.
- Personal Access Tokens are masked in every log line and input box.
