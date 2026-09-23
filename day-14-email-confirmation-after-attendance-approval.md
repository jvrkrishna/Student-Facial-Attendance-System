# Day 14 — Email Confirmation After Attendance Approval

We will only work on the email confirmation now.

## Today's goal

Student submits attendance
        ↓
PENDING
        ↓
Admin / Superadmin approves
        ↓
APPROVED
        ↓
Email sent to student

## Part 1 — Test Django Email

First we will test email without sending a real email.

File path: config/settings.py

Add this at the bottom:

```python
EMAIL_BACKEND = "django.core.mail.backends.console.EmailBackend"
```

This means Django will print the email in the terminal.

## Part 2 — Test Email

Start your server first.

Terminal command

```bash
python manage.py runserver
```

Open another terminal while runserver is running.

Terminal command

```bash
python manage.py shell
```

Then:

Terminal command

```python
from django.core.mail import send_mail
```

Then:

Terminal command

```python
send_mail(
    "Attendance Approved",
    "Your attendance has been approved.",
    "test@example.com",
    ["student@example.com"],
)
```

You should see the email content printed in the terminal where runserver is running.

You should see something similar to:

```text
Subject: Attendance Approved
From: test@example.com
To: student@example.com

Your attendance has been approved.
```

Exit the shell.

Terminal command

```python
exit()
```

## Part 3 — Connect Email to Existing Approval

Your approval view is already working, so do not create another view.

File path: attendance/views.py

Add this import if it is not already present:

```python
from django.core.mail import send_mail
from django.conf import settings
```

Then find your existing:

```python
@login_required
def approve_attendance(request, attendance_id):
    if request.user.role not in ["ADMIN", "SUPERADMIN"]:
        return redirect("home")

    if request.method == "POST":
        Attendance.objects.filter(id=attendance_id,status="PENDING"
        ).update(status="APPROVED")
    return redirect("attendance_approvals")
```

Immediately after attendance.save(), add:

```python
send_mail(
    "Attendance Approved",
    f"Your attendance for {attendance.attendance_date} has been approved.",
    None,
    [attendance.student.email],
)
```

So your existing approval logic becomes:

File path: attendance/views.py

```python
@login_required
def approve_attendance(request, attendance_id):
    if request.user.role not in ["ADMIN", "SUPERADMIN"]:
        return redirect("home")

    if request.method == "POST":
        attendance = Attendance.objects.filter(id=attendance_id,status="PENDING").first()

        if attendance:
            attendance.status = "APPROVED"
            attendance.save()

            send_mail(
                "Attendance Approved",
                f"Your attendance for {attendance.attendance_date} has been approved.",
                settings.DEFAULT_FROM_EMAIL,
                [attendance.student.email],
            )

    return redirect("attendance_approvals")
```

Do not change your existing approval view structure.

## Part 4 - Now Modify settings.py 

Remove these lines from settings.py which are in bottom
```python
MAILERS = {
    'default': {
        'BACKEND': 'django.core.mail.backends.console.EmailBackend',
    },
}

EMAIL_BACKEND = "django.core.mail.backends.console.EmailBackend"
```

Add these 
```python
# Email
EMAIL_BACKEND = "django.core.mail.backends.smtp.EmailBackend"

EMAIL_HOST = "smtp.gmail.com"
EMAIL_PORT = 587
EMAIL_USE_TLS = True

EMAIL_HOST_USER = "hietechsolutions@gmail.com"
EMAIL_HOST_PASSWORD = "gplhrpuojrqxxly"

DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
```

### For HOST PASSWORD create app password first
Get the App Password
For the Gmail account you're using as the sender:
Google Account → Security → 2-Step Verification → App passwords

Create an app password, for example named:
Student Attendance

Google will give you a 16-character password.

Put that password here:
EMAIL_HOST_PASSWORD = "your-16-character-app-password"

It is not your normal Gmail password.

## Part 5 — Test Complete Flow

Terminal command

```bash
python manage.py runserver
```

Then test:

Student
   ↓
Submit Attendance
   ↓
PENDING

Login as Admin or Superadmin:

Attendance Approvals
        ↓
     Approve

Expected result:

Attendance → APPROVED

And the mail should display:

```text
Subject: Attendance Approved
To: student's email

Your attendance for YYYY-MM-DD has been approved.
```
