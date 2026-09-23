# Day 11 — Admin Attendance Approval
Today we build the Admin/Superadmin attendance approval GUI.

The flow becomes:
Student
   ↓
Submit Attendance
   ↓
PENDING
   ↓
Admin / Superadmin
   ↓
Approve or Reject
   ↓
APPROVED / REJECTED

## 1. Models
No changes today.
File path: attendance/models.py
Do not modify it.
The existing Attendance.Status already has:
PENDING
APPROVED
REJECTED

## 2. Forms
No form changes today.
File path: attendance/forms.py
Do not modify it.
The Approve/Reject buttons will use POST requests directly.

## 3. Views
```python
# Show pending attendance and summary
@login_required
def attendance_approvals(request):
    if request.user.role not in ["ADMIN", "SUPERADMIN"]:
        return redirect("home")

    pending = Attendance.objects.filter(status="PENDING")

    summary = {
        "pending": pending.count(),
        "approved": Attendance.objects.filter(status="APPROVED").count(),
        "rejected": Attendance.objects.filter(status="REJECTED").count(),
    }

    return render(request, "attendance/attendance_approvals.html", {
        "pending_records": pending,
        "summary": summary,
    })

# Approve selected attendance
@login_required
def approve_attendance(request, attendance_id):
    if request.user.role not in ["ADMIN", "SUPERADMIN"]:
        return redirect("home")

    if request.method == "POST":
        Attendance.objects.filter(id=attendance_id,status="PENDING"
        ).update(status="APPROVED")
    return redirect("attendance_approvals")
```

## 4. URLs

File path: attendance/urls.py

Add these paths to your existing urlpatterns:

```python
path("attendance/approvals/",views.attendance_approvals,name="attendance_approvals"),

path("attendance/approve/<int:attendance_id>/",views.approve_attendance,name="approve_attendance"),
```

## 5. Create Approval GUI

Now create the Admin/Superadmin page.

File path: student_attendance/templates/attendance/attendance_approvals.html

```html
{% extends "base.html" %}

{% block title %}Attendance Approvals{% endblock %}

{% block content %}

<h2>Attendance Approvals</h2>

<p>Pending: {{ summary.pending }}</p>
<p>Approved: {{ summary.approved }}</p>
<p>Rejected: {{ summary.rejected }}</p>

{% for record in pending_records %}

    <div>
        <p>Student: {{ record.student.username }}</p>
        <p>Date: {{ record.attendance_date }}</p>
        <p>Status: {{ record.get_status_display }}</p>

        <form method="post"
              action="{% url 'approve_attendance' record.id %}">
            {% csrf_token %}
            <button type="submit">Approve</button>
        </form>
    </div>

    <hr>

{% empty %}

    <p>No pending attendance.</p>

{% endfor %}

{% endblock %}
```

## 6. Add Link to Header

Admin and Superadmin need access to the approval page.

File path: student_attendance/templates/base.html

Inside your existing logged-in navigation, add:

```html
{% if user.role == "ADMIN" or user.role == "SUPERADMIN" %}
    <a href="{% url 'attendance_approvals' %}">Attendance Approvals</a>
{% endif %}
```

Now the header will behave like this:

Student:

Home | User | Logout

Admin:

Home | Attendance Approvals | User | Logout

Superadmin:

Home | Student Approvals | Attendance Approvals | User | Logout

## 7. Test With a Student

First, make sure you have a pending attendance record.

Login as a student and submit attendance.

Then check Admin:

Attendance → Attendances

You should have:

Student       Date          Status

**----------------------------------------**

student1      2026-09-12    Pending

## 8. Test Admin Approval

Logout from the student.

Login as your Admin account.

You should see:

Attendance Approvals

Click it.

You should see:

Attendance Approvals

Student: student1

Date: 2026-09-12

Status: Pending

[Approve] [Reject]

Click Approve.

The record should become:

APPROVED

## 9. Test Rejection

Create another pending attendance record using another student/day if necessary.

Open:

Attendance Approvals

Click:

Reject

The status should become:

REJECTED

## 10. Test Superadmin

Login as Superadmin.

Superadmin should also see:

Attendance Approvals

and be able to:

Approve

Reject

This gives:

                Attendance Approval

                         │

             ┌──────────┴──────────┐

             ↓                     ↓

          Admin                Superadmin

             ↓                     ↓

          Approve               Approve

          Reject                Reject

## 11. Test Student Restriction

Login as a Student.

Try opening:

/attendance/approvals/

The student should be redirected to Home.

The student should also not see:

Attendance Approvals

in the header.

## 12. Current System Flow

We now have:

                STUDENT

                   │

                   ↓

             Submit Attendance

                   │

                   ↓

                PENDING

                   │

          ┌────────┴────────┐

          ↓                 ↓

       ADMIN           SUPERADMIN

          │                 │

       Approve           Approve

       Reject            Reject

          │                 │

          └────────┬────────┘

                   ↓

             APPROVED/REJECTED

The student's attendance history from Day 10 will automatically show the new status.

For example:

Date          Status

**-------------------------**

2026-09-12    Approved

2026-09-11    Rejected
