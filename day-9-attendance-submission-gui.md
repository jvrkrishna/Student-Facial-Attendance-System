# Day 9 — Attendance Submission GUI

Today we will create the first student attendance GUI.
We will keep it intentionally simple:

Student Login
      ↓
Student Dashboard
      ↓
Submit Attendance
      ↓
Attendance record created
      ↓
    PENDING

Important: Facial verification and location validation are not connected yet. We will add them before allowing real attendance approval.

## 1. Models
No changes today.

## 2. Forms
Now we create the attendance form.

The student should not select their username or date manually.
Django will determine:
Student → logged-in user
Date → today's date
Status → PENDING

File path: attendance/forms.py

Create the file:
```python
from django import forms
from .models import Attendance

class AttendanceForm(forms.ModelForm):
    class Meta:
        model = Attendance
        fields = []
```

## 3. Views
Now create the attendance submission view.
File path: attendance/views.py

Replace the empty file with:
```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import render, redirect
from django.utils import timezone

from .forms import AttendanceForm
from .models import Attendance


@login_required
def submit_attendance(request):
    # Only students can submit attendance
    if request.user.role != "STUDENT":
        return redirect("home")

    today = timezone.localdate()

    # Prevent duplicate attendance for the same day
    if Attendance.objects.filter(student=request.user,attendance_date=today).exists():
        return render(request,"attendance/submit_attendance.html",           {
                "form": AttendanceForm(),
                "error": "You have already submitted attendance today.",
            })

    form = AttendanceForm(request.POST or None)

    if form.is_valid():
        Attendance.objects.create(
            student=request.user,
            attendance_date=today,
            status="PENDING",)
        return redirect("attendance_success")

    return render(request,"attendance/submit_attendance.html",
        {"form": form,})


@login_required
def attendance_success(request):
    return render(request,"attendance/attendance_success.html")
```

## 4. URLs
Create the attendance URLs.
File path: attendance/urls.py

```python
from django.urls import path
from . import views

urlpatterns = [
    path("submit/",views.submit_attendance,name="submit_attendance",),
    path("success/",views.attendance_success,name="attendance_success",),
]
```

## 5. Connect Attendance URLs
File path: config/urls.py

Add:

```python
path("attendance/", include("attendance.urls")),
```

## 6. Create Attendance GUI

Use your agreed project-level template structure.

File path: student_attendance/templates/attendance/submit_attendance.html

```html
{% extends "base.html" %}

{% block title %}Submit Attendance{% endblock %}

{% block content %}

<h2>Submit Attendance</h2>

{% if error %}

    <p>{{ error }}</p>

{% else %}

    <p>Student: {{ user.username }}</p>

    <p>Today's attendance will be submitted for the current date.</p>

    <form method="post">

        {% csrf_token %}

        {{ form.as_p }}

        <button type="submit">Submit Attendance</button>

    </form>

{% endif %}

{% endblock %}
```

## 8. Success GUI

File path: student_attendance/templates/attendance/attendance_success.html

```html
{% extends "base.html" %}

{% block title %}Attendance Submitted{% endblock %}

{% block content %}

<h2>Attendance Submitted</h2>

<p>Your attendance has been submitted.</p>

<p>Status: Pending approval</p>

{% endblock %}
```

## 9. Add Submit Attendance to Student Home

Now students need a way to reach the page.

File path: student_attendance/templates/accounts/home.html

Find the existing student section:

```html
{% if user.role == "STUDENT" %}

    <p>Student account</p>

{% endif %}
```

Replace only that section with:

```html
{% if user.role == "STUDENT" %}

    <p>Student account</p>

    <a href="{% url 'submit_attendance' %}">

        Submit Attendance

    </a>

{% endif %}
```

Now an approved student will see:

Welcome, student1

Role: Student

Student account

Submit Attendance

## 10. Test the GUI

Start the server.

Terminal command

```bash
python manage.py runserver
```

Login as an approved student.

Go to:

http://127.0.0.1:8000/

Click:

Submit Attendance

You should see:

Submit Attendance

Student: student1

Today's attendance will be submitted for the current date.

[ Submit Attendance ]

## 11. Submit Attendance

Click:

Submit Attendance

You should see:

Attendance Submitted

Your attendance has been submitted.

Status: Pending approval

## 12. Check Django Admin

Open:

http://127.0.0.1:8000/admin/

Go to:

Attendance → Attendances

You should see something like:

Student       Date          Status

**----------------------------------------**

student1      2026-09-12    Pending

The exact date will be your current local date.

## 13. Test Duplicate Attendance

Go back to:

http://127.0.0.1:8000/attendance/submit/

Try submitting again.

You should see:

You have already submitted attendance today.

This works because Day 8 added the database constraint and today's view also checks for an existing record.

## 14. Test Admin

Login as Admin.

Try:

http://127.0.0.1:8000/attendance/submit/

You should be redirected to Home.

The attendance submission page is only for:

role = STUDENT

## 15. Very Important: This Is Not Final Attendance Yet

At this stage, the student can submit attendance without face or location verification.

That is intentional for Day 9.

Our current flow is:

Student
   ↓
Login
   ↓
Submit Attendance
   ↓
Attendance record
   ↓
PENDING

The final flow will become:

Student
   ↓
Submit Attendance
   ↓
Camera
   ↓
Facial verification
   ↓
Location permission
   ↓
Location validation
   ↓
Attendance record
   ↓
PENDING
   ↓
Admin / Superadmin approval
   ↓
APPROVED
   ↓
Email confirmation

We will modify the submission process later to include those checks rather than adding incomplete code now.
