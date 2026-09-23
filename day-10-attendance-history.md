# Day 10 — Attendance History

Today we will build the day-wise attendance records page for students.

We already have attendance records being saved. Now we'll give the student a simple GUI to view their own records.

Today's flow
Student Login
     ↓
Home
     ↓
My Attendance
     ↓
Day-wise records

Example:
Date          Status
**-------------------------**
2026-09-12    Pending
2026-09-11    Approved
2026-09-10    Rejected
No new model is needed.
No facial recognition or location code yet.

## 1. Models
No changes today.
File path: attendance/models.py
Do not modify it.

We already have:
student
attendance_date
status
created_at

That's enough to display attendance history.

## 2. Forms
No form is required today.
The student is only viewing records.
File path: attendance/forms.py
Do not modify it.

## 3. Views
We need a view that shows only the logged-in student's attendance.

This is important:
student1 → sees student1 records
student2 → sees student2 records

A student must not be able to see another student's attendance.

File path: attendance/views.py
Add this view below your existing views:

```python
@login_required
def attendance_history(request):

    if request.user.role != "STUDENT":
        return redirect("home")
    
    records = Attendance.objects.filter(student=request.user)

    return render(request,"attendance/attendance_history.html",{"records": records},)
```

## 4. URLs
File path: attendance/urls.py

Add:

```python
path("history/",views.attendance_history,name="attendance_history",),
]
```

The new URL will be:
/attendance/history/

## 5. Create Attendance History GUI
Use your project-level templates folder.

File path: student_attendance/templates/attendance/attendance_history.html

```html
{% extends "base.html" %}

{% block title %}My Attendance{% endblock %}

{% block content %}

<h2>My Attendance</h2>

{% if records %}

    <table border="1">

        <tr>

            <th>Date</th>

            <th>Status</th>

        </tr>

        {% for record in records %}

            <tr>

                <td>{{ record.attendance_date }}</td>

                <td>{{ record.get_status_display }}</td>

            </tr>

        {% endfor %}

    </table>

{% else %}

    <p>No attendance records found.</p>

{% endif %}

{% endblock %}
```

## 6. Add Link to Student Home
Students need a way to open their attendance history.

File path: student_attendance/templates/accounts/home.html

Find the student section you created earlier:
```html
{% if user.role == "STUDENT" %}
    <p>Student account</p>
    <a href="{% url 'submit_attendance' %}">Submit Attendance</a><br>
    <a href="{% url 'attendance_history' %}">My Attendance</a>
{% endif %}
```

## 7. Test the GUI
Start the server.
Terminal command
```bash
python manage.py runserver
```

Login as an approved student.

Go to:

http://127.0.0.1:8000/

You should now see:

Welcome, student1

Role: Student

Student account

Submit Attendance

My Attendance

Click:

My Attendance

## 8. If You Already Submitted Attendance
You should see something like:
My Attendance
Date          Status
**-------------------------**
2026-09-12    Pending
The date will be your actual attendance date.

## 9. Test Different Statuses
For testing, you can temporarily change an attendance record through Django Admin.

Open:
http://127.0.0.1:8000/admin/

Go to:
Attendance → Attendances
Change the status from:
Pending
to:
Approved

Then return to:
/attendance/history/

You should see:
Date          Status
**-------------------------**
2026-09-12    Approved
You can also test:
Rejected

## 10. Test Student Privacy
Create or use another student account.

For example:
student1
student2

Submit attendance for both.
Login as student1.

Open:
/attendance/history/

Student1 should see only student1's records.
Then login as student2.
Student2 should see only student2's records.

This is because the query uses:

```python
student=request.user
```

rather than retrieving all attendance records.

## 11. Test Admin
Login as Admin.
Try:
http://127.0.0.1:8000/attendance/history/

The view should redirect to Home because:

```python
if request.user.role != User.Role.STUDENT:

    return redirect("home")
```

The page is currently designed specifically for students.
