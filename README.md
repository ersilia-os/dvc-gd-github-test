# dvc-gd-github-test

Test repository for GitHub, DVC and Google Drive integration.

This repo contains the **instructions only**.
You will create and push a **separate** repository named **`test_integration`** below.

---

## Policies (who can `dvc pull`/`push`)

| Policy   | Drive folder sharing | `dvc pull`                    | `dvc push`                    |
|----------|----------------------|-------------------------------|-------------------------------|
| Public   | “Anyone with the link → Viewer” + add maintainers as **Editor/Content manager** | Anyone with a Google account (they’ll auth in browser) | Only emails you added as **Editor/Content manager** |
| Private  | “Restricted” (share by email only) | Only emails you added as **Viewer/Editor/Content manager** | Only emails you added as **Editor/Content manager** |

> DVC never makes your data public by itself. **Google Drive sharing** is the source of truth.

---

## Prerequisites

- Git + GitHub account (and optionally the GitHub CLI `gh`)
- Conda (or Python ≥3.10 + venv)
- A Google account with **Google Drive**
- Access to **Google Cloud Console** to enable **Google Drive API** and create an **OAuth client** (no server needed) <-- This needs to be done only once (and it's already done!)

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

printf "data/\nresults/\n" >> .gitignore  # ignoring data and results directories

git add .
git commit -m "Initialize DVC; ignore data/ and results/"
```

6. Create the Google Drive folder and get its ID

    In Google Drive, create a folder for project data (e.g., test_integration_data). Copy its folder ID from the URL. (the long string after /folders/, e.g., 1jyTkx9qwxPAnpEeQsnjqDQTctIO6zgB_).

    a/ Public policy (dvc pull everyone, dvc push only selected users).

        Set the folder's general access to "Anyone with the link --> Viewer". All users will be prompted to sign in with a Google account on their first dvc pull.

        Add selected maintainers by email as "Editor" or "Content manager"


    b/ Private policy (dvc pull only selected users, dvc push only selected users).

        Set the folder's general access to "Restricted".

        Add only the selected user emails: "Viewer" for pull-only users, "Editor" or "Content manager" for push-capable users.


7. Google Cloud Console

Open the Google Cloud Console, make sure you’re on the right Google account, and create or select a project.

Console home: https://console.cloud.google.com/
Project create: https://console.cloud.google.com/projectcreate

Enable the Google Drive API for that project. To do so, go to the API Library, find Google Drive API, and click Enable.

Enable Drive API directly: https://console.cloud.google.com/apis/library/drive.googleapis.com.


3) Configure the OAuth consent screen

In the left sidebar, open APIs & Services → OAuth consent screen and set it up. You’ll choose a User type and fill in App name, User support email, Developer contact email, etc.





7. Configure the DVC remote (OAuth per user; no secrets in Git)

```bash
dvc remote add -d gdrive gdrive://<YOUR_FOLDER_ID>
git add .dvc/config
git commit -m "Configure DVC remote (Google Drive via OAuth)"

# Optional: pick a local path for the OAuth token (not committed)
dvc remote modify --local gdrive gdrive_user_credentials_file ~/.config/dvc/gdrive-test_integration.json
```

8. Track data with DVC and push

# Create a tiny file and a heavy (~1 GB) file

```bash
mkdir -p data/raw
echo "hello dvc+gdrive" > data/raw/hello.txt
dd if=/dev/zero of=data/raw/big.bin bs=1M count=1024 status=progress
dvc add data results
git add data.dvc .gitignore
git commit -m "Track data/ with DVC pointers"
dvc push -j 4
```


