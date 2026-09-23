# Day 13 — Facial Verification

## Goal

Today we make sure the student's camera face is compared with the face image uploaded during signup.

```text
Signup face
     ↓
Student's registered face
     ↓
Camera face
     ↓
DeepFace comparison
     ↓
Match?
  ┌───┴───┐
 Yes      No
  ↓        ↓
Pending  Reject
```

We will keep the code simple.

---

# Part 1 — Reset the Virtual Environment

Before installing the facial-recognition packages, save the current package list and recreate the virtual environment.

### Step 1 — Save Current Requirements

Run:

```bash
pip freeze > requirements.txt
```

### Step 2 — Deactivate the Existing Virtual Environment

```bash
deactivate
```

### Step 3 — Remove the Existing Virtual Environment

On Windows:

```bash
rmdir /s /q venv
```

### Step 4 — Create a New Python 3.11 Virtual Environment

```bash
py -0p
```

if not present 3.11 version run these
```bash
winget install --id Python.Python.3.11 -e
```

```bash
py -3.11 -m venv venv
```

### Step 5 — Activate the Virtual Environment

```bash
venv\Scripts\activate
```

### Step 6 — Check Python Version

```bash
python --version
```

Make sure Python 3.11 is being used.

### Step 7 — Upgrade pip

```bash
python -m pip install --upgrade pip
```

---

# Part 2 — Update Django Version

Before installing the packages from `requirements.txt`, open:

```text
requirements.txt
```

Find the Django entry and change it to:

```text
Django==5.2.17
```

Save the file.

This makes sure the project uses Django 5.2.17.

---

# Part 3 — Install Project Requirements

Install the packages listed in `requirements.txt`:

```bash
pip install -r requirements.txt
```

---

# Part 4 — Install Facial Recognition Packages

### Step 1 — Install DeepFace

```bash
pip install "deepface[tensorflow]"

python -c "from deepface import DeepFace; print('DeepFace OK')"
```
If successful:
DeepFace OK

### Step 2 — Install tf-keras

Because the TensorFlow version used by DeepFace requires `tf-keras`, install it:

```bash
pip install tf-keras
```

---

# Part 5 — Install the Correct OpenCV Version

During facial-verification testing, OpenCV needs to work correctly with DeepFace.

First remove the existing OpenCV package:

```bash
pip uninstall opencv-python -y
```

Then install the required version:

```bash
pip install opencv-python==4.10.0.84
```

---

# Part 6 — Save the Updated Requirements

After installing DeepFace, tf-keras, and OpenCV, update the requirements file:

```bash
pip freeze > requirements.txt
```

Now the newly installed packages are included in `requirements.txt`.

---

# Part 7 — Create the Facial Verification Utility

Keep the facial-recognition logic separate from the view.

Create this file **inside the `attendance` app**:

```text
attendance/face_utils.py
```

Add:

```python
from deepface import DeepFace
import tempfile
import os


def verify_face(uploaded_image, reference_path):
    current_path = None

    try:
        with tempfile.NamedTemporaryFile(
            suffix=".jpg",
            delete=False
        ) as file:
            for chunk in uploaded_image.chunks():
                file.write(chunk)

            current_path = file.name

        result = DeepFace.verify(
            img1_path=reference_path,
            img2_path=current_path,
            model_name="VGG-Face",
            detector_backend="retinaface",
            distance_metric="cosine",
            enforce_detection=True,
            align=True,
        )

        return bool(result["verified"])

    except ValueError as e:
        if "Face could not be detected" in str(e):
            return False
        raise

    finally:
        if current_path and os.path.exists(current_path):
            os.remove(current_path)
```

### Simple Meaning

```text
reference
```

is the student's registered face image.

```text
image
```

is the current camera image.

DeepFace compares the two images and returns:

```text
True
```

or:

```text
False
```

The temporary camera image is deleted after verification.

---

# Part 8 — Update the Attendance View

Open:

```text
attendance/views.py
```

```python
from .face_utils import verify_face
@login_required
def submit_attendance(request):
    if request.user.role != "STUDENT":
        return redirect("home")

    today = timezone.localdate()

    if Attendance.objects.filter(
        student=request.user,
        attendance_date=today
    ).exists():
        return render(
            request,
            "attendance/submit_attendance.html",
            {"error": "You have already submitted attendance today."}
        )

    if request.method == "POST":
        form = AttendanceForm(request.POST)
        image = request.FILES.get("face_image")

        if not image:
            return render(
                request,
                "attendance/submit_attendance.html",
                {"form": form, "error": "Please capture your face."}
            )

        reference = request.user.face_image.path

        if form.is_valid() and verify_face(image, reference):
            attendance = form.save(commit=False)
            attendance.student = request.user
            attendance.attendance_date = today
            attendance.status = "PENDING"
            attendance.save()

            return redirect("attendance_success")

        return render(
            request,
            "attendance/submit_attendance.html",
            {"form": form, "error": "Face verification failed."}
        )

    return render(
        request,
        "attendance/submit_attendance.html",
        {"form": AttendanceForm()}
    )
```

### Important

Attendance is saved **only when**:

```python
verify_face(...) == True
```

If another person uses the camera:

```python
verify_face(...) == False
```

the attendance record is **not created**.

# Part 9 — Test the Correct Face

Start the Django development server:

```bash
python manage.py runserver
```

Login as an approved student.

Go to:

```text
/attendance/submit/
```

Then:

```text
Start Camera
     ↓
Capture Face
     ↓
Submit Attendance
```

Look at the terminal.

For the registered student, you should see:

```text
FACE MATCH: True
```

The attendance should be created with:

```text
PENDING
```

---

# Part 10 — Test Another Person

Now use another person's face.

Submit the attendance.

The terminal should show:

```text
FACE MATCH: False
```

The page should display:

```text
Face verification failed.
```

No attendance record should be created.

This confirms that facial verification is being enforced before attendance is saved.

---

# Part 11 — Test Duplicate Attendance

After a successful attendance submission, try submitting attendance again on the same date.

You should see:

```text
You have already submitted attendance today.
```

No second attendance record should be created.
