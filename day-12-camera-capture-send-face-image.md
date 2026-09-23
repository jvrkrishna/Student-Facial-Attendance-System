# Day 12 — Camera Capture + Send Face Image to Django


Today's goal is:

Student
   ↓
Submit Attendance
   ↓
Start Camera
   ↓
Capture Image
   ↓
Submit Image to Django
   ↓
Attendance PENDING

We are not implementing facial recognition yet. The captured image will be sent to Django so we can use it for facial verification in the next stage.

## 1. Models

No changes are required.

## 2. Forms

Add the form that receives the captured face image.

File path: student_attendance/attendance/forms.py

Add:

```python
class FaceVerificationForm(forms.Form):
    face_image = forms.ImageField()
```

## 3. Views

We need Django to receive the image.

File path: student_attendance/attendance/views.py

Add FaceVerificationForm to your existing forms import.

For example, if you have:

```python
from .forms import AttendanceForm
```

change it to:

```python
from .forms import AttendanceForm, FaceVerificationForm
```

Do not replace your complete view.

Your existing attendance logic should remain.

## 4. Attendance Template

Now we connect the camera capture to the Django form.

File path: student_attendance/templates/attendance/submit_attendance.html

Replace the complete file with:

```html
{% extends "base.html" %}

{% block title %}Submit Attendance{% endblock %}

{% block content %}

<h2>Submit Attendance</h2>

<p>Student: {{ user.username }}</p>

{% if error %}

    <p>{{ error }}</p>

{% else %}

    <h3>Face Verification</h3>

    <p>Allow camera access and capture your face.</p>

    <video id="camera" width="400" height="300" autoplay></video>

    <br><br>

    <button type="button" id="startCamera">
        Start Camera
    </button>

    <button type="button" id="capture">
        Capture Image
    </button>

    <br><br>

    <canvas id="preview" width="400" height="300"></canvas>

    <p id="message"></p>

    <br>

    <form method="post" enctype="multipart/form-data">

        {% csrf_token %}

        <input
            type="file"
            name="face_image"
            id="faceImage"
            hidden
        >

        <button type="submit">
            Submit Attendance
        </button>

    </form>

    <script>
        startCamera.onclick = async function() {
            camera.srcObject = await navigator.mediaDevices.getUserMedia({
                video: true
            });
        };

        capture.onclick = function() {
            preview.getContext("2d").drawImage(
                camera,
                0,
                0,
                400,
                300
            );

            preview.toBlob(function(blob) {
                const file = new File([blob], "face.jpg", {
                    type: "image/jpeg"
                });

                const data = new DataTransfer();

                data.items.add(file);

                faceImage.files = data.files;
            });

            message.textContent = "Image captured.";
        };
    </script>

{% endif %}

{% endblock %}

```

## 5. What the Page Does

The student will see:

Submit Attendance

Student: rama

Face Verification

Allow camera access and capture your face.

[ Camera ]

[Start Camera] [Capture Image]

[Captured Image]

[Submit Attendance]

### Start Camera

Click:

Start Camera

The browser asks for camera permission.

Choose:

Allow

### Capture Image

Click:

Capture Image

The current camera frame is copied to the canvas.

You should see:

Image captured.

### Submit Attendance

Click:

Submit Attendance

The captured image is included in the POST request.

Django can receive it through:

request.FILES

## 6. Important Difference

Previously:

Camera
   ↓
Capture
   ↓
Image exists only in browser

Now:

Camera
   ↓
Capture
   ↓
Image
   ↓
Form
   ↓
Django
   ↓
request.FILES

This prepares the system for facial verification.

## 7. Duplicate Attendance

Your existing duplicate check remains unchanged.

If the student already submitted attendance today:

You have already submitted attendance today.

The camera and submit controls will not be shown.

## 8. Do Not Add a New Attendance Image Field Yet

We are not adding something like:

camera_image

to the Attendance model.

The reason is simple:

Registered face
→ stored during signup

Daily camera image
→ used for verification
→ no permanent database storage yet

This avoids unnecessary biometric-data storage.

## 9. Facial Recognition Comes Next

Once Django can receive the image, the next stage is:

Student's registered face_image
             +
       Camera image
             ↓
      Face comparison
             ↓
       Match / No Match

Only after that will we connect location validation:

Face Match
    +
Location Valid
    ↓
Create Attendance
    ↓
PENDING

## 10. Testing

Terminal command

```bash
python manage.py runserver
```

Open:

http://127.0.0.1:8000/attendance/submit/
