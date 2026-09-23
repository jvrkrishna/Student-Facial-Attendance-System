# Day 8 — Create Attendance App + Attendance Model

Today we start the actual attendance part of the project.

## Today's goal
Create a separate attendance app and store:

```text
Student
   ↓
Attendance record
   ↓
Date
   ↓
Status
```

For now, we are not adding facial recognition, camera, location, email, or approval GUI. Those will be added when their respective stages are reached.

## 1. Create Attendance App
### Terminal command

```bash
python manage.py startapp attendance
```

Your project should now contain:

```text
student_attendance/
│
├── accounts/
│
├── attendance/
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   └── ...
│
├── templates/
│   ├── base.html
│   └── accounts/
│
├── config/
└── manage.py
```

## 2. Register the App

File path: `config/settings.py`

Find:

```python
INSTALLED_APPS = [
```

Add:

```python
"attendance",
```

Do not change anything else.

## 3. Models

Our attendance record needs only the basic information for now:

- Student
- Date
- Status
- Created time

File path: `attendance/models.py`

Add:

```python
from django.conf import settings
from django.db import models

class Attendance(models.Model):
    student = models.ForeignKey(settings.AUTH_USER_MODEL,on_delete=models.CASCADE)

    attendance_date = models.DateField()
    status = models.CharField(max_length=10,
        choices=[("PENDING", "Pending"),("APPROVED", "Approved"),
        ("REJECTED", "Rejected"),],default="PENDING",)

    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ["-attendance_date"]
        constraints = [models.UniqueConstraint(
                fields=["student", "attendance_date"],
                name="unique_student_attendance_per_day",)]
```

### Why these fields?

| Field | Purpose |
|---|---|
| `student` | Which student submitted |
| `attendance_date` | Day of attendance |
| `status` | Pending/Approved/Rejected |
| `created_at` | When the record was created |

The unique constraint prevents the same student from creating multiple attendance records for the same day.

## 4. Why settings.AUTH_USER_MODEL?

We don't directly write:

```python
User
```

Instead:

```python
settings.AUTH_USER_MODEL
```

because you created your own custom User model in accounts.

This keeps the attendance app properly connected to your custom authentication system.

## 5. Create Migration

### Terminal command

```bash
python manage.py makemigrations attendance
```

Then:

### Terminal command

```bash
python manage.py migrate
```

Expected result should include something similar to:

```text
Applying attendance.0001_initial... OK
```

## 6. Admin

For today's testing, let's make attendance visible in Django Admin.

File path: `attendance/admin.py`

Add:

```python
from django.contrib import admin

from .models import Attendance


@admin.register(Attendance)
class AttendanceAdmin(admin.ModelAdmin):

    list_display = (
        "student",
        "attendance_date",
        "status",
        "created_at",
    )

    list_filter = (
        "status",
        "attendance_date",
    )
```

Now Django Admin can display attendance records day-wise.

## 7. Test Admin

Start the server.

### Terminal command

```bash
python manage.py runserver
```

Open:

```text
http://127.0.0.1:8000/admin/
```

You should now see:

```text
Attendance

    ↓

Attendances
```

There won't be any records yet.

That's expected.

## 8. Why We Don't Create a Form Today

We could create an attendance form now, but it would be incomplete because attendance will eventually require:

```text
Face verification

+

Location validation

+

Student

+

Current date
```

If we create a form now and then keep modifying it later, that creates unnecessary work.

So today we only establish the database structure.

## 9. Future Attendance Flow

The model we created will eventually support this:

```text
Student Login
      ↓
Submit Attendance
      ↓
Camera
      ↓
Face Verification
      ↓
Location Verification
      ↓
Attendance Record
      ↓
PENDING
      ↓
Admin / Superadmin
      ↓
APPROVED
      ↓
Email Student
```

The database is now ready for the basic attendance status workflow.

## 10. Day-wise Records
Because we have:

```python
attendance_date = models.DateField()
```

we can later display:

```text
Date          Status
-------------------------
12 Sep 2026   Approved
11 Sep 2026   Pending
10 Sep 2026   Approved
09 Sep 2026   Rejected
```

We don't need a separate AttendanceDate model.

## 11. Important Role Rule
The attendance model points to your custom User:

```text
Attendance
     ↓
User
```

But later, when submitting attendance, the view will check that the logged-in user is:

```text
STUDENT
```

So Admin and Superadmin won't submit student attendance.

Later:
```text
Student
    → Submit attendance

Admin
    → Approve attendance

Superadmin
    → Approve attendance
    → Approve student registration
```

This keeps the responsibilities separate.
