# LLRS - Login, Logout, Registration & Reset

A user authentication system built with Django and PostgreSQL. This project demonstrates how to implement user registration, login, logout, and password reset functionality using Django's built-in authentication system.

## Project Features

- User Registration
- User Login
- Login with Username or Email
- User Logout
- Protected Pages
- Password Reset (Planned)
- PostgreSQL Database Integration

## Technologies Used

- Python
- Django
- PostgreSQL
- HTML
- CSS
- JavaScript
- Bootstrap (if applicable)

---

# Project Setup Guide

## Step 1: Create a Virtual Environment

First, create a project directory and navigate into it.

```bash
mkdir LLRS
cd LLRS
```

Create a virtual environment:

```bash
python -m venv venv
```

Activate the virtual environment.

**Windows PowerShell:**

```powershell
.\venv\Scripts\Activate.ps1
```

If PowerShell blocks script execution, run:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Then activate the virtual environment again.

---

## Step 2: Install Django

Install Django inside the virtual environment:

```bash
pip install django
```

Check the installed Django version:

```bash
django-admin --version
```

---

## Step 3: Create a Django Project

Create the Django project:

```bash
django-admin startproject LLRS
```

Navigate into the project directory:

```bash
cd LLRS
```

The project structure will look like this:

```text
LLRS/
│
├── manage.py
│
└── LLRS/
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    ├── asgi.py
    └── wsgi.py
```

---

## Step 4: Create an Authentication App

Create an app named `authentication`:

```bash
python manage.py startapp authentication
```

Add the app to `INSTALLED_APPS` inside `settings.py`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'authentication',
]
```

---

## Step 5: Configure PostgreSQL Database

Install the PostgreSQL database adapter:

```bash
pip install psycopg2-binary
```

Create a PostgreSQL database named:

```text
LLRS
```

Configure the database inside `settings.py`:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'LLRS',
        'USER': 'postgres',
        'PASSWORD': 'YOUR_POSTGRES_PASSWORD',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

Replace `YOUR_POSTGRES_PASSWORD` with your PostgreSQL password.

**Note:** Never publish your actual database password on GitHub. Use environment variables for credentials in production.

---

## Step 6: Configure Templates Directory

Create a folder named `templates` in the same directory as `manage.py`.

Project structure:

```text
LLRS/
│
├── manage.py
│
├── LLRS/
│   ├── settings.py
│   └── urls.py
│
├── authentication/
│
└── templates/
    ├── register.html
    ├── login.html
    └── success.html
```

Configure the templates directory inside `settings.py`:

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

---

## Step 7: Run Database Migrations

Apply the initial migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

These commands create the necessary database tables for Django's built-in authentication system and other installed apps.

---

## Step 8: Create a Superuser

Create an administrator account:

```bash
python manage.py createsuperuser
```

Enter the required username, email, and password.

Run the development server:

```bash
python manage.py runserver
```

Open the Django admin panel:

```text
http://127.0.0.1:8000/admin/
```

---

# Authentication System Development

## Step 9: Create a Success Page View

Import the required modules inside `authentication/views.py`:

```python
from django.shortcuts import render, redirect
from django.contrib.auth.models import User
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.decorators import login_required
```

Create a success page view:

```python
@login_required(login_url='login')
def success_view(request):
    return render(
        request,
        template_name='success.html',
    )
```

### Explanation

- `login_required` protects the success page.
- Only authenticated users can access this page.
- Unauthenticated users are redirected to the login page.

---

## Step 10: Create a User Registration View

Create a registration view:

```python
def register_view(request):

    if request.method == 'POST':

        username = request.POST.get('username')
        email = request.POST.get('email')
        password = request.POST.get('password')

        if User.objects.filter(username=username).exists():
            return render(
                request,
                'register.html',
                {'error': 'Username already exists!'}
            )

        User.objects.create_user(
            username=username,
            email=email,
            password=password
        )

        return redirect('login')

    return render(
        request,
        template_name='register.html'
    )
```

### Explanation

- `request.method == 'POST'` checks whether the registration form was submitted.
- `request.POST.get()` retrieves the submitted form data.
- `User.objects.filter()` checks whether the username already exists.
- `User.objects.create_user()` creates a new user.
- Django automatically hashes the password.
- After successful registration, the user is redirected to the login page.

---

## Step 11: Create a User Login View

Create a login view:

```python
def login_view(request):

    if request.method == 'POST':

        username_or_email = request.POST.get('username')
        password = request.POST.get('password')

        user = User.objects.filter(
            email=username_or_email
        ).first()

        if user is None:
            user = User.objects.filter(
                username=username_or_email
            ).first()

        if user is not None:

            authenticate_user = authenticate(
                request,
                username=user.username,
                password=password
            )

            if authenticate_user is not None:

                login(request, authenticate_user)

                return redirect('success')

        return render(
            request,
            'login.html',
            {'error': 'Invalid username/email or password!'}
        )

    return render(
        request,
        template_name='login.html'
    )
```

### Explanation

- The user can enter either a username or an email address.
- Django searches for the matching user.
- `authenticate()` verifies the username and password.
- `login()` creates an authenticated session.
- After successful login, the user is redirected to the success page.
- If authentication fails, an error message is displayed.

---

## Step 12: Create a Logout View

Create a logout view:

```python
def logout_view(request):

    if request.method == 'POST':

        logout(request)

        return redirect('login')

    return redirect('success')
```

### Explanation

- `logout(request)` ends the user's authenticated session.
- After logout, the user is redirected to the login page.
- Logout should be performed through a POST request rather than a GET request.

In `success.html`, use a POST form:

```html
<form method="POST" action="{% url 'logout' %}">
    {% csrf_token %}

    <button type="submit">
        Logout
    </button>
</form>
```

---

## Step 13: Configure URL Routing

Open `authentication/urls.py` if you create a separate URL configuration for the app.

```python
from django.urls import path
from .views import (
    success_view,
    register_view,
    login_view,
    logout_view,
)

urlpatterns = [
    path('success/', success_view, name='success'),
    path('register/', register_view, name='register'),
    path('login/', login_view, name='login'),
    path('logout/', logout_view, name='logout'),
]
```

Include the app URLs inside the main `LLRS/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('authentication.urls')),
]
```

---

## Step 14: Configure Authentication Settings

Add the following settings to `settings.py`:

```python
LOGIN_URL = 'login'

LOGIN_REDIRECT_URL = 'success'

LOGOUT_REDIRECT_URL = 'login'
```

### Explanation

- `LOGIN_URL` defines the login page URL.
- `LOGIN_REDIRECT_URL` defines the default redirect after login.
- `LOGOUT_REDIRECT_URL` defines the default redirect after logout.

---

# Authentication Flow

```text
              Start
                |
                |
         User Registration
                |
                |
       Registration Successful
                |
                |
          Login Page
                |
                |
         Enter Credentials
                |
                |
          Authentication
                |
          /           \
       Valid          Invalid
         |               |
         |               |
    Success Page     Error Message
         |
         |
      Logout
         |
         |
      Login Page
```

---

# Important Django Concepts Learned

| Concept | Purpose |
|---|---|
| Virtual Environment | Isolates project dependencies |
| Django Project | Main configuration of the application |
| Django App | Organizes application functionality |
| PostgreSQL | Stores application data |
| User Model | Stores user account information |
| `create_user()` | Creates a user with a hashed password |
| `authenticate()` | Checks user credentials |
| `login()` | Logs a user into the application |
| `logout()` | Logs a user out |
| `login_required` | Protects pages from unauthorized access |
| `redirect()` | Redirects users to another URL |
| `render()` | Returns an HTML template |
| `request.POST` | Retrieves submitted form data |

---

# Common Commands

Activate virtual environment:

```powershell
.\venv\Scripts\Activate.ps1
```

Install Django:

```bash
pip install django
```

Create a Django project:

```bash
django-admin startproject LLRS
```

Create an app:

```bash
python manage.py startapp authentication
```

Create migrations:

```bash
python manage.py makemigrations
```

Apply migrations:

```bash
python manage.py migrate
```

Create a superuser:

```bash
python manage.py createsuperuser
```

Run the development server:

```bash
python manage.py runserver
```

---

# Future Improvements

- Password Reset
- Password Change
- Email Verification
- Username and Email Validation
- Better Error Handling
- User Profile Management
- Improved Security
- Deployment

---

# Project Status

<text>
Registration: Completed
Login: Completed
Logout: Completed
Protected Pages: Completed
Password Reset: Planned
</text>

---


GitHub: https://github.com/gagukibriya