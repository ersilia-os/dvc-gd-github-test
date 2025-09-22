# dvc-gd-github-test

Test repository for GitHub, DVC and Google Drive integration.

This repo contains the **instructions only**.
You will create and push a **separate** repository named **`test_integration`** below.

---

## Policies (who can use **Git** and **DVC**)

| Policy  | GitHub repo visibility | `git clone/pull` (code)            | `git push` (code)                     | Drive folder sharing (data)                                        | `dvc pull` (data)                                  | `dvc push` (data)                                   |
|---------|------------------------|-------------------------------------|---------------------------------------|--------------------------------------------------------------------|----------------------------------------------------|-----------------------------------------------------|
| Public  | **Public**             | Anyone on the internet              | Only collaborators with write access  | **Anyone with the link → Viewer**; add maintainers as **Editor/Content manager** | Anyone with a Google account (browser OAuth)       | Only emails added as **Editor/Content manager**     |
| Private | **Private**            | Only users with repo read access    | Only collaborators with write access  | **Restricted**; share explicitly by email                         | Only users you shared with (Viewer/Editor/CM)      | Only emails added as **Editor/Content manager**     |

> Code access is controlled by **GitHub**. Data access is controlled by **Google Drive**.  
> DVC never makes your data public by itself: **Drive sharing** is the source of truth for data.

**How to set each policy**

- **Public**
  - GitHub: create repo as **Public**; add collaborators with write access.
  - Drive: set folder to **Anyone with the link - Viewer**; add push maintainers as **Editor/Content manager**.
- **Private**
  - GitHub: create repo as **Private**; add collaborators with read/write as needed.
  - Drive: set folder to **Restricted**; share by email (Viewer = pull, Editor/CM = push).

**Security notes**
- Keep only the DVC remote **URL** in `.dvc/config`. Put OAuth values in **`.dvc/config.local`** (untracked).
- `.dvc` pointer files in Git reveal **paths + hashes**, not the data itself.

---

## Prerequisites

- Git + GitHub account (and optionally the GitHub CLI `gh`)
- Conda (or Python ≥3.10 + venv)
- A Google account with **Google Drive**
- Access to Ersilia's **Google Cloud Console** to enable **Google Drive API** and create an **OAuth client** (no server needed). This needs to be done only once, and it's already done! Exhaustive details on how to do it are provided below. 

**One-time Google Cloud setup (for DVC ↔ Google Drive)**

- Open Google Cloud Console and create/select a project: https://console.cloud.google.com/projectcreate (e.g. dvc-public). Set **Organisation** and **Location** to **ersilia.io**.
- Enable the Google Drive API for that project. In the top bar, pick your project (project picker next to the Google Cloud logo). Then, go to the **Navigation Menu** --> **APIs and services** --> **Library** --> search **Google Drive API**, and **Enable**. 
- Configure the OAuth consent screen. In the **Navigation Menu**, select **APIs and services** --> **OAuth consent screen**. In the **Overview** menu, select **Get started** and fill the **App information** (set the app name to your project name, include your email and specify whether the expected audience is **Internal** or **External**). 
- Create an **OAuth client** (Desktop app). In the **Navigation menu**, select **APIs and services** --> **Credentials** --> **Create credentials** --> **OAuth client ID** --> **Application type: Desktop app** --> **Create**. Make sure to store the Client ID and the Client secret associated to the OAuth client you just created before closing the dialogue. Conveniently, you can download everything in JSON file format to be later used in authentication steps. 


---


# Steps

1. Create the local project, initialize Git and make the first commit.

```bash
mkdir test_integration
cd test_integration
mkdir -p code data results
echo "# test_integration" > README.md
echo "" > code/.gitkeep

git init
git branch -M main
git add README.md code
git commit -m "Initial commit - project skeleton"
```

2. Create the GitHub repository

```bash
sudo apt update && sudo apt install -y gh
gh auth login  # Login to your GitHub account
gh auth status  # Should show you're logged in

# Replace <org-or-user> with your GitHub username or org (no angle brackets)

# PUBLIC
gh repo create <org-or-user>/test_integration --public --source=. --remote=origin --push

# PRIVATE
gh repo create <org-or-user>/test_integration --private --source=. --remote=origin --push
```
Alternatively, you can create the repository manually on the GitHub website (create an empty test_integration repo without a README), then set the remote locally and push.

3. Verify GitHub repository

```bash
git remote -v
git log --oneline -n 1
```

4. Create and activate a Python environment

```bash
conda create -n test_integration python=3.10 -y
conda activate test_integration
```

5. Install and initialize DVC

```bash
pip install "dvc[gdrive]"
dvc init

printf "data/\nresults/\n.dvc/config.local\n" >> .gitignore  # ignoring data, results and OAuth config

git add .
git commit -m "Initialize DVC; ignore data/ and results/"
```

6. Create the Google Drive folder for the project and get its ID

    In Google Drive, create a folder for project data (e.g., test_integration_data) inside your Drive path Github/private or Github/public. Copy its folder ID from the URL. (i.e., the long string after /folders/, e.g., 1jyTkx9qwxPAnpEeQsnjqDQTctIO6zgB_).

    a/ Public policy (dvc pull everyone, dvc push only selected users, see Policies).

        Set the folder's general access to "Anyone with the link --> Viewer". All users will be prompted to sign in with a Google account on their first dvc pull.

        Add selected maintainers by email as "Editor" or "Content manager"


    b/ Private policy (dvc pull only selected users, dvc push only selected users, see Policies).

        Set the folder's general access to "Restricted".

        Add only the selected user emails: "Viewer" for pull-only users, "Editor" or "Content manager" for push-capable users.


7. Configure the DVC remote

Client ID and Client Secret are specified in the corresponding [credentials-private.json](https://drive.google.com/file/d/1qBzS43UsQf7Qmjon3IAtM6FJTB-Ofsex/view?usp=sharing) and [credentials-public.json](https://drive.google.com/file/d/1AEhlLAbEP3oeETaHN3C8KF19RZHKw2_F/view?usp=sharing), both included in Ersilia's Google Drive. Pick one of the files accordingly. For safety, make sure that JSON files including Client IDs and secrets are not committed. 

```bash
mkdir -p ~/.config/dvc
dvc remote add -d gdrive gdrive://<DRIVE_FOLDER_ID>  # or, if the gdrive remote already exists, dvc remote modify gdrive url gdrive://<DRIVE_FOLDER_ID>
git add .dvc/config && git commit -m "DVC remote -> Google Drive"

dvc remote modify --local gdrive gdrive_client_id "<CLIENT_ID_FROM_JSON>"
dvc remote modify --local gdrive gdrive_client_secret "<CLIENT_SECRET_FROM_JSON>"
dvc remote modify --local gdrive gdrive_user_credentials_file ~/.config/dvc/${PWD##*/}-gdrive.json
```

With the last command, we tell DVC to store your Google OAuth token in a per-repo file under ~/.config/dvc (untracked), so auth persists locally and no secrets get committed.

8. Track data with DVC and push

Create a tiny file and a heavy (~1 GB) file.

```bash
mkdir -p data/raw
echo "hello dvc+gdrive" > data/raw/hello.txt
dd if=/dev/zero of=data/raw/big.bin bs=1M count=1024 status=progress
dvc add data results
git add data.dvc results.dvc .gitignore
git commit -m "Track data/ and results/ with DVC pointers"
dvc push -j 4
dvc status -c
git push origin main
```

To enumerate files tracked by Git and DVC, run the following commands:

```bash
git ls-files  # Tracked by Git
git ls-files '*.dvc'  # Tracked by DVC
```


# Collaborator, quick start

Prerequisites: You have access to the GitHub repo and the correct Drive folder (pull-only: Drive role = Viewer (or higher); push permissions: Drive role = Editor/Content manager)

```bash
# Clone the repo
git clone <repo>
cd <repo>

# Create conda environment and install dvc[gdrive]
conda create -n repo_env python=3.10 -y
conda activate repo_env
pip install "dvc[gdrive]"

# Add the Google Drive remote LOCALLY (not committed)
dvc remote add --local -d gdrive gdrive://<FOLDER_ID>  # replce FOLDER_ID with the actual folder's ID from Google Drive (see Steps: step 6)
dvc config --local core.remote gdrive

# Put OAuth **client** values and a local token path in LOCAL config
# JSON files with Client IDs and secrets are located in our Google Drive (see Steps: step 7)
mkdir -p ~/.config/dvc
dvc remote modify --local gdrive gdrive_client_id "<CLIENT_ID_FROM_JSON>"
dvc remote modify --local gdrive gdrive_client_secret "<CLIENT_SECRET_FROM_JSON>"
dvc remote modify --local gdrive gdrive_user_credentials_file ~/.config/dvc/${PWD##*/}-gdrive.json

# Pull all data
dvc pull
dvc status -c  # should say cache and remote are in sync
```
