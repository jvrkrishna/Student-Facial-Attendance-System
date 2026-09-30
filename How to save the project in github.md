Django Project — Upload to GitHub

1. Create ".gitignore"

Create a file named ".gitignore" in the root folder of the Django project, alongside "manage.py".

.env
__pycache__/
venv/
media/
db.sqlite3

---

2. Recommended Project Structure
A typical Django project should look like:

my-django-project/
│
├── manage.py
├── .gitignore
├── requirements.txt
│
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
└── myapp/
    ├── migrations/
    ├── models.py
    ├── views.py
    └── ...

---

3. What to Upload to GitHub

Upload these

- "manage.py"
- Django application code
- "settings.py"
- "urls.py"
- Models
- Views
- Templates
- CSS/JS source files
- Django migrations
- "requirements.txt"
- ".gitignore"
- "README.md"

Do NOT upload these

- "venv/"
- ".env"
- "db.sqlite3"
- "__pycache__/"
- ".pyc" files
- Passwords
- API keys
- Secret keys
- IDE configuration files
- Generated files

---

4. Create "requirements.txt"

Activate your virtual environment first.

Windows

venv\Scripts\activate

Linux / macOS

source venv/bin/activate

Then run:

pip freeze > requirements.txt

This saves the Python packages required by your project.

Another developer can install them using:

pip install -r requirements.txt

---

5. Initialize Git

Open the terminal in the folder containing "manage.py".

Run:

git init

---

6. Add Files to Git

git add .

Check the files before committing:

git status

Make sure files such as these are NOT being added:

venv/
.env
db.sqlite3
__pycache__/

---

7. Create Your First Commit

git commit -m "Initial commit"

---

8. Create a GitHub Repository

Go to GitHub and create a new repository.

Example repository name:

my-django-project

If your project already exists locally, you can create the GitHub repository without adding:

- README
- ".gitignore"
- License

---

9. Connect Local Project to GitHub

Copy your GitHub repository URL.

Example:

git remote add origin https://github.com/YOUR_USERNAME/my-django-project.git

Check the remote:

git remote -v

---

10. Rename Branch to "main"

git branch -M main

---

11. Push Project to GitHub

git push -u origin main

Your Django project should now be available on GitHub.

---

12. Complete Command List

For a new Django project, the basic workflow is:

cd my-django-project

# Activate virtual environment
venv\Scripts\activate

# Create requirements.txt
pip freeze > requirements.txt

# Initialize Git
git init

# Add files
git add .

# Check files
git status

# Commit
git commit -m "Initial commit"

# Connect GitHub repository
git remote add origin https://github.com/YOUR_USERNAME/my-django-project.git

# Rename branch
git branch -M main

# Push
git push -u origin main

---

13. After Making Changes

Whenever you make changes to your Django project:

git status

Then:

git add .

Commit the changes:

git commit -m "Describe your changes"

Push to GitHub:

git push

Typical workflow:

Change code
    ↓
git status
    ↓
git add .
    ↓
git commit -m "message"
    ↓
git push

---

14. Important Django Security

Never upload real secrets to GitHub.

Don't do this:

SECRET_KEY = "my-real-secret-key"

PASSWORD = "my-database-password"

API_KEY = "my-real-api-key"

Instead, use environment variables.

Example:

import os

SECRET_KEY = os.environ.get("SECRET_KEY")

Put the actual secret in ".env":

SECRET_KEY=your-real-secret-key

Because ".env" is in ".gitignore", it won't be uploaded to GitHub.

---

15. Django Migrations

Do not ignore your migration files.

For example, this should normally be committed:

myapp/
└── migration