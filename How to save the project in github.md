# Django Project — Upload to GitHub

This guide explains how to upload a Django project to GitHub safely and correctly.

---

## 1. Create `.gitignore`

Create a file named `.gitignore` in the **root folder of your Django project**, alongside `manage.py`.

Example `.gitignore`:

```gitignore
.env
__pycache__/
*.pyc
venv/
media/
db.sqlite3
```

> **Important:** Do not upload passwords, API keys, secret keys, virtual environments, or database files containing sensitive data.

---

## 2. Recommended Project Structure

A typical Django project should look like this:

```text
my-django-project/
│
├── manage.py
├── .gitignore
├── requirements.txt
├── README.md
│
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
│
└── myapp/
    ├── migrations/
    │   └── __init__.py
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── views.py
    └── ...
```

---

## 3. What to Upload to GitHub

### Upload These Files

You should normally commit:

* `manage.py`
* Django application code
* `settings.py`
* `urls.py`
* Models
* Views
* Templates
* CSS/JS source files
* Django migrations
* `requirements.txt`
* `.gitignore`
* `README.md`
* `asgi.py`
* `wsgi.py`

### Do NOT Upload These

Do not commit:

* `venv/`
* `.env`
* `db.sqlite3`
* `__pycache__/`
* `*.pyc`
* Passwords
* API keys
* Secret keys
* IDE configuration files
* Generated files
* Other sensitive or machine-specific files

---

## 4. Create `requirements.txt`

Activate your virtual environment first.

### Windows

```powershell
venv\Scripts\activate
```

### Linux / macOS

```bash
source venv/bin/activate
```

Then run:

```bash
pip freeze > requirements.txt
```

This saves the Python packages required by your project.

Another developer can install the required packages using:

```bash
pip install -r requirements.txt
```

---

## 5. Initialize Git

Open the terminal in the folder containing `manage.py`.

Run:

```bash
git init
```

This initializes a Git repository for your Django project.

---

## 6. Add Files to Git

Run:

```bash
git add .
```

Check which files are going to be committed:

```bash
git status
```

Make sure files such as these are **NOT** being added:

```text
venv/
.env
db.sqlite3
__pycache__/
*.pyc
```

If `.gitignore` is configured correctly, Git should automatically ignore them.

---

## 7. Create Your First Commit

Create the first commit:

```bash
git commit -m "Initial commit"
```

---

## 8. Create a GitHub Repository

Go to GitHub and create a new repository.

Example repository name:

```text
my-django-project
```

If your project already exists locally, you can create the GitHub repository **without** adding:

* README
* `.gitignore`
* License

You can add these files locally instead.

---

## 9. Connect the Local Project to GitHub

Copy your GitHub repository URL.

Example:

```bash
git remote add origin https://github.com/YOUR_USERNAME/my-django-project.git
```

Check the remote:

```bash
git remote -v
```

You should see something similar to:

```text
origin  https://github.com/YOUR_USERNAME/my-django-project.git (fetch)
origin  https://github.com/YOUR_USERNAME/my-django-project.git (push)
```

---

## 10. Rename the Branch to `main`

Run:

```bash
git branch -M main
```

This renames your current branch to `main`.

---

## 11. Push the Project to GitHub

Run:

```bash
git push -u origin main
```

Your Django project should now be available on GitHub.

---

# 12. Complete Command List

For a new Django project, the basic workflow is:

```bash
cd my-django-project
```

### Activate virtual environment

**Windows:**

```powershell
venv\Scripts\activate
```

**Linux / macOS:**

```bash
source venv/bin/activate
```

### Create `requirements.txt`

```bash
pip freeze > requirements.txt
```

### Initialize Git

```bash
git init
```

### Add files

```bash
git add .
```

### Check files

```bash
git status
```

### Commit

```bash
git commit -m "Initial commit"
```

### Connect GitHub repository

```bash
git remote add origin https://github.com/YOUR_USERNAME/my-django-project.git
```

### Rename branch

```bash
git branch -M main
```

### Push

```bash
git push -u origin main
```

---

# 13. After Making Changes

Whenever you make changes to your Django project, follow this workflow.

### 1. Check the changes

```bash
git status
```

### 2. Add the changes

```bash
git add .
```

### 3. Commit the changes

```bash
git commit -m "Describe your changes"
```

### 4. Push to GitHub

```bash
git push
```

### Typical Workflow

```text
Change code
    ↓
git status
    ↓
git add .
    ↓
git commit -m "message"
    ↓
git push
```

---

# 14. Important Django Security

## Never Upload Real Secrets to GitHub

Do **not** hard-code real secrets in your Django source code.

### ❌ Don't do this

```python
SECRET_KEY = "my-real-secret-key"

PASSWORD = "my-database-password"

API_KEY = "my-real-api-key"
```

Instead, use environment variables.

### ✅ Use Environment Variables

For example:

```python
import os

SECRET_KEY = os.environ.get("SECRET_KEY")
```

You can store the actual secret in a `.env` file:

```env
SECRET_KEY=your-real-secret-key
```

Because `.env` is included in `.gitignore`, it will not be uploaded to GitHub.

> **Important:** `.gitignore` only prevents files from being added to Git in the normal workflow. If a secret was already committed and pushed, simply adding it to `.gitignore` does **not** remove it from Git history. You should rotate the exposed secret immediately and clean the repository history if necessary.

---

# 15. Django Migrations

Do **not** ignore your migration files.

For example, this should normally be committed:

```text
myapp/
└── migrations/
    ├── __init__.py
    ├── 0001_initial.py
    ├── 0002_...
    └── ...
```

Run:

```bash
python manage.py makemigrations
```

Then:

```bash
python manage.py migrate
```

Commit the migration files:

```bash
git add .
git commit -m "Add database migrations"
git push
```

### Why Commit Migrations?

Migrations allow another developer or deployment environment to recreate the database schema from your Django models.

---

# 16. Useful Git Commands

### Check repository status

```bash
git status
```

### View commit history

```bash
git log --oneline
```

### View configured remote

```bash
git remote -v
```

### Add all changes

```bash
git add .
```

### Commit changes

```bash
git commit -m "Your commit message"
```

### Push changes

```bash
git push
```

### Pull changes from GitHub

```bash
git pull
```

---

# 17. Recommended Final Project Structure

A clean Django project for GitHub may look like:

```text
my-django-project/
│
├── .gitignore
├── README.md
├── requirements.txt
├── manage.py
│
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
│
├── myapp/
│   ├── migrations/
│   │   ├── __init__.py
│   │   └── 0001_initial.py
│   │
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   └── tests.py
│
├── templates/
│   └── ...
│
└── static/
    ├── css/
    ├── js/
    └── images/
```

---

# 18. Final Checklist

Before pushing your Django project to GitHub, verify:

* [ ] `.gitignore` exists
* [ ] `venv/` is ignored
* [ ] `.env` is ignored
* [ ] `db.sqlite3` is ignored
* [ ] `__pycache__/` is ignored
* [ ] `*.pyc` is ignored
* [ ] No passwords are in the source code
* [ ] No API keys are in the source code
* [ ] No real secret keys are exposed
* [ ] `requirements.txt` exists
* [ ] Migration files are included
* [ ] `README.md` exists
* [ ] `git status` was checked before committing
* [ ] GitHub remote is configured
* [ ] Branch is named `main`
* [ ] Project was successfully pushed to GitHub

---

# 19. Quick Reference

```bash
# Go to project
cd my-django-project

# Activate virtual environment
venv\Scripts\activate

# Create/update requirements
pip freeze > requirements.txt

# Initialize Git
git init

# Add files
git add .

# Check files
git status

# Create commit
git commit -m "Initial commit"

# Connect GitHub
git remote add origin https://github.com/YOUR_USERNAME/my-django-project.git

# Rename branch
git branch -M main

# Push to GitHub
git push -u origin main
```

For future changes:

```bash
git status
git add .
git commit -m "Describe your changes"
git push
```
