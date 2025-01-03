# **Building Spokane Tech: Part 3**

Welcome to part 3 of the "Building Spokane Tech" series! In this article, we go through steps to get the web app running locally on your machine or environment.


## **Prerequisites**
- git installed on system
- python installed on system (3.10+ recommended)
- access to the SpokaneTechWeb github repo: [https://github.com/SpokaneTech/SpokaneTechWeb](https://github.com/SpokaneTech/SpokaneTechWeb)

## **Local Setup**

### Cloning the Repo
```
git git@github.com:SpokaneTech/SpokaneTechWeb.git
```

### cd into the repo directory
```
cd SpokaneTechWeb
```

### Create a python virtual environment
```
python -m venv venv
```

### Activate the python virtual environment
for linux, mac, or wsl:
```
source venv/bin/activate
```
for powershell:

```powershell
venv\Scripts\activate
```

### Install the python dependencies
```
pip install .[dev]
```

### Create an .env.local file from the .env.template file and update contents as applicable (optional)
```
cp src/envs/.env.template src/envs/.env.local
```

### cd to the django_project directory
```
cd src/django_project
```

### Create a local database by running django migrations
```
python./manage.py migrate
```

### Create a local admin user
```
python ./manage.py add_superuser --group admin
```

### Generate some local test data
```
python ./manage.py runscript generate_dev_data
```

### Start the local demo server
```
python ./manage.py runserver
```

open a browser and navigate to http://127.0.0.1:8000 (log in with admin/admin)

** you can stop the local demo server anytime via ```ctrl + c ```

## **Enable Git Hooks (optional)**
### git config
To enable pre-commit code quality checks, update the location of git hooks with the following command:
```shell
git config core.hooksPath .github/hooks
```

Note: to make a commit with the precommit hooks temporarily disabled, run the following:
```
git commit --no-verify
```
