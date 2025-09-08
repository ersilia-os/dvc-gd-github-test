# test-repository
Test repository for DVC, Google Drive and Github integration

# Steps

1. Clone this repository.

```
git clone git@github.com:ersilia-os/dvc-gd-github-test.git
cd dvc-gd-github-test
```

2. Create and activate conda environment.

```
conda create -n dvctest python=3.10 -y
conda activate dvctest
```

3. Install and init DVC. File .dvcignore and directory .dvc will be automatically created.

```
pip install "dvc[gdrive]"
dvc init
```

5. Create a Google Drive folder to include all heavy files

- 5.1: Log in to the Google Cloud console
- 5.2: In the search bar, select "Create a project". Choose a Project name, and leave Organisation and Location as "ersilia.io"
- 5.3: DVC will use the Google Drive API to connect to your Google Drive account. To enable this feature, in the upper left menu, select "APIs & Services" → "Library", look for Google Drive API, press "Enable". 
- 5.4: Create a Service Account. In the upper left menu, select "APIs & Services" → "Credentials" → "Create credentials" → "Service account". Give it a service account ID (e.g. dvc-gdrive), and "Create and continue", "Done".
- 5.5: Create a Key for your Service Account. In the upper left menu, select "IAM and admin", "Service accounts". Click your service account, go to the "Keys" tab, click "Add key" → "Create new key" (JSON). A JSON file with your credentials will be automatically downloaded. Inside, you’ll see fields like "client_email" and "private_key". CAUTION: Don't share these credentials, as they grant direct access to your Google Drive account. Don't include this file in your Github repository. 
- 5.6: Create a Google Drive folder and copy its ID from the corresponding URL. e.g. 1MYMynlRVCZvn0GMC4VMBX2h8EahSXS7A in https://drive.google.com/drive/folders/1MYMynlRVCZvn0GMC4VMBX2h8EahSXS7A
- 5.7: Share the folder with the Service account. Service account has an associated email specified in the JSON credentials file. In Google Drive, right-click the folder → "Share", add that email as "Content Manager".

6. Configure DVC to use that Drive remote

```
dvc remote add -d gdrive gdrive://<YOUR_FOLDER_ID>
dvc remote modify gdrive gdrive_use_service_account true
dvc remote modify gdrive gdrive_service_account_json_file_path /absolute/path/to/sa-key.json

```

7. Track data with DVC and keep it out of Git

```
dvc add data
git add data.dvc .gitignore .dvc .dvcignore
git commit -m "Track data with DVC; keep data/ out of Git"

```

5. Create a folder in Google Drive to store data

6. Point DVC to the Google Drive folder

```
dvc remote add -d gdrive gdrive://<FOLDER_ID>  # e.g. FOLDER_ID: 1yyf6Gxgu0YrxNvf_WHoExBZ87MO-T1ct
git add .dvc/config
git commit -m "Configure DVC remote: Google Drive"

```
