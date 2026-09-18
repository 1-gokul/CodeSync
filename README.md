
# CodeSync | Automated GitHub Code Synchronization & Version Control System

CodeSync is a native Python desktop application that simplifies local-to-GitHub synchronization by bringing Git operations, GitHub repository management, and synchronization history into a single dashboard.

Instead of repeatedly running `git add`, `git commit`, and `git push` manually, CodeSync provides a streamlined workflow for monitoring projects, reviewing changes, generating commit messages, and synchronizing updates with GitHub.

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

- **Project Workspace Integration**: Monitor any local project folder for file changes.
- **Git State Discovery**: Automatically detects Modified (`M`), Staged/Added (`A`), Deleted (`D`), and Untracked (`??`) files.
- **Auto-generated Commit Messages**: Analyzes detected file changes and suggests a relevant commit message using rule-based classification.
- **Interactive Repository Connection**: Connect an existing GitHub repository or create a new repository with a default `main` branch.
- **MongoDB Synchronization History**: Stores synchronization records including changed-file counts, commit messages, timestamps, and error details.
- **Background Scheduler**: Automatically synchronizes configured repositories at regular intervals using a non-blocking background thread.
- **Secure Token Authentication**: Supports GitHub Personal Access Token authentication with credentials masked in the application and logs.
- **Responsive Desktop Interface**: Disk operations, network requests, and Git operations run in background workers to keep the GUI responsive during synchronization.

---

## Technical Architecture

``
               +-------------------------------------------------+
               |                   CodeSync GUI                  |
               |  (Dashboard, Repositories, History, Scheduler)  |
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
       |  (GitPython)    |     |    (PyGithub)    |     |    (PyMongo)    |
       +-----------------+     +-----------------+     +-----------------+
                |                       |                       |
                v                       v                       v
       [ Local Git Repo ]      [ GitHub Cloud API ]    [ MongoDB — local or Atlas ]
````

All database operations are handled through `DatabaseManager`, providing a centralized interface for data access.

This keeps database logic isolated from the rest of the application and makes storage-related changes easier to manage. The project previously used SQLite before migrating to MongoDB.

---

## Technologies Used

| Library / Technology | Role                                                                   |
| -------------------- | ---------------------------------------------------------------------- |
| **Python 3.11+**     | Application runtime                                                    |
| **CustomTkinter**    | Desktop GUI framework                                                  |
| **GitPython**        | Local Git operations such as status, staging, commits, and branches    |
| **PyGithub**         | GitHub API integration and repository management                       |
| **PyMongo**          | MongoDB database driver                                                |
| **MongoDB**          | Stores repositories, synchronization history, and application settings |
| **python-dotenv**    | Loads environment variables and application configuration              |
| **schedule**         | Background synchronization scheduling                                  |
| **Pillow**           | GUI image and asset handling                                           |

---

## Installation

### 1. Prerequisites

**Python** — Install Python 3.11 or newer from [python.org](https://www.python.org/downloads/).

On Windows, enable **"Add Python to PATH"** during installation.

**Git** — CodeSync uses Git for local repository operations, so Git must be installed separately.

* Windows: [git-scm.com](https://git-scm.com/download/win)
* macOS: `brew install git`
* Linux: `sudo apt install git`

**MongoDB** — Choose either a local MongoDB installation or MongoDB Atlas.

* **Local installation**: Install [MongoDB Community Server](https://www.mongodb.com/try/download/community) and make sure the MongoDB service is running.
* **MongoDB Atlas**: Create a cluster through [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) and use the provided connection string.

### 2. Project Setup

```bash
pip install -r requirements.txt
```

---

## Environment Setup

### 1. Create the environment file

Copy the example environment file:

```bash
cp .env.example .env
```

### 2. Configure the environment variables

Update `.env` with your GitHub and MongoDB configuration:

GITHUB_TOKEN=ghp_yourpersonaltokenhere
GITHUB_USERNAME=yourusername
DEFAULT_BRANCH=main
DEFAULT_COMMIT_MESSAGE=Update project files via CodeSync

MONGO_URI=mongodb://localhost:27017
MONGO_DB_NAME=codesync

For MongoDB Atlas, replace the local MongoDB URI with your Atlas connection string, typically starting with:

mongodb+srv://

### Creating a GitHub Personal Access Token (PAT)

1. Open GitHub → **Settings** → **Developer Settings** → **Personal Access Tokens**.
2. Select **Tokens (classic)** and choose **Generate new token (classic)**.
3. Set an expiration date and provide a descriptive name such as `CodeSync App`.
4. Grant the required `repo` permission for repository access.
5. Generate the token and copy it immediately.

The token can be entered through the application's **Settings** panel or stored in the `.env` file.

---

## Running the Application

Start CodeSync with:

python main.py

## Complete Usage Workflow

### 1. Authenticate

Go to:

**Settings → GitHub Username + PAT → Test GitHub Connection → Save Settings**

### 2. Add a Project

Go to:

**Repositories → Add Repository**

Select a local project folder and choose either:

* **Connect Existing Repo** — connect the project to an existing GitHub repository.
* **Create New Repo** — create a new repository on GitHub.

Then select **Add Project**.

### 3. Synchronize Manually

From the **Dashboard**:

1. Select a project under **Active Project**.
2. Review the detected file changes.
3. Generate a commit message or enter a custom message.
4. Select **Sync to GitHub**.

### 4. Automate Synchronization

Open the **Scheduler** tab:

1. Enable **Automatic Sync**.
2. Select the required synchronization interval.
3. Select the repository.
4. Save the scheduler configuration.

CodeSync then performs synchronization automatically in the background.

### 5. Review Synchronization History

The **History** tab records previous synchronization operations.

Double-click a history entry to view detailed information about that synchronization run, including errors and execution details.

---

## Project Structure

CodeSync/
│
├── main.py                     # Application entry point
├── config.py                   # Configuration and filesystem path handling
├── requirements.txt            # Python dependencies
├── README.md                   # Project documentation
├── .env.example                # Environment configuration template
├── .gitignore                  # Git tracking exclusions
│
├── core/
│   ├── git_manager.py          # GitPython operations
│   ├── github_manager.py       # GitHub API operations
│   ├── sync_manager.py         # End-to-end synchronization coordinator
│   └── scheduler.py            # Background synchronization scheduler
│
├── database/
│   └── database_manager.py     # MongoDB collections and database operations
│
├── gui/
│   ├── app.py                  # Main application window
│   ├── dashboard.py            # Synchronization workspace
│   ├── repositories.py         # Repository management interface
│   ├── settings.py             # Application settings
│   ├── history.py              # Synchronization history
│   ├── scheduler.py            # Automation interface
│   └── about.py                # Application information
│
├── utils/
│   ├── logger.py               # Application logger with token masking
│   ├── validators.py           # Input validation helpers
│   └── helpers.py              # Commit message generation utilities
│
├── tests/
│   ├── test_git.py
│   ├── test_github.py
│   ├── test_database.py
│   └── test_utils.py
│
└── logs/
    └── automation.log          # Application execution log

Application data is stored in MongoDB, either through a local MongoDB instance or MongoDB Atlas. CodeSync does not rely on a local `.db` file.

---

## Safety Rules

* CodeSync does not force-push by default, helping prevent unintended overwriting of remote Git history.
* Project folders are never deleted automatically.
* `.env` files are Git-ignored by default to prevent credentials and connection strings from being committed.
* GitHub Personal Access Tokens are masked in the application interface and logs.

```
