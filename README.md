# Full-Stack To-Do App Project: React + Django REST API

---

## Project Goal

Build a **simple full-stack To-Do App** with:

- A **React frontend** (using Create React App)
- A **Django REST API backend** (using Django REST Framework)
- Full CRUD functionality
- User login system (simple session auth)
- Undo (delete recovery) using browser **sessions** (React localStorage or Django session optional)

# Setup Instructions

## 1. Install Needed Tools

* **Python** (3.10+ preferred)
* **Node.js** (v18+ preferred) — [Download Node.js](https://nodejs.org/)
* **PyCharm** — Professional (best) or Community + Node.js plugin.

## 2. Create Backend (Django API)

### 2.1. Create a Django Project

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install django djangorestframework
django-admin startproject todo_backend
cd todo_backend
python manage.py startapp tasks
```

### 2.2. Configure Settings

In `todo_backend/settings.py`:

- Add `'rest_framework'` and `'tasks'` to `INSTALLED_APPS`.
- Allow CORS (for frontend communication):
  ```bash
  pip install django-cors-headers
  ```
- In `settings.py`:

  ```python
  INSTALLED_APPS += ['corsheaders']

  MIDDLEWARE = ['corsheaders.middleware.CorsMiddleware'] + MIDDLEWARE

  CORS_ALLOW_ALL_ORIGINS = True  # (for development only)
  ```

---

### 2.3. Define Task Model

In `tasks/models.py`:

```python
from django.db import models

class Task(models.Model):
    title = models.CharField(max_length=255)
    completed = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

```bash
python manage.py makemigrations
python manage.py migrate
```

---

### 2.4. Create Serializers

In `tasks/serializers.py`:

```python
from rest_framework import serializers
from .models import Task

class TaskSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = '__all__'
```

---

### 2.5. Create API Views

In `tasks/views.py`:

```python
from rest_framework import viewsets
from .models import Task
from .serializers import TaskSerializer

class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all().order_by('-created_at')
    serializer_class = TaskSerializer
```

---

### 2.6. Setup URLs

In `tasks/urls.py`:

```python
from rest_framework import routers
from .views import TaskViewSet
from django.urls import path, include

router = routers.DefaultRouter()
router.register(r'tasks', TaskViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

In `todo_backend/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('tasks.urls')),
]
```

Your Django API now works at:
```
GET    /api/tasks/
POST   /api/tasks/
PUT    /api/tasks/<id>/
DELETE /api/tasks/<id>/
```

Run it:

```bash
python manage.py runserver
```

## 3. Create Frontend (React App)

### 3.1. Create React App

```bash
npx create-react-app todo_frontend
cd todo_frontend
npm install axios react-router-dom
```

### 3.2. Connect to API

In `package.json`, add:

```json
"proxy": "http://localhost:8000",
```

### 3.3. Create To-Do UI

Example components:

- `App.js` → load tasks and route pages
- `TaskList.js` → list tasks
- `TaskForm.js` → create/edit tasks
- `UndoContext.js` (optional) → use React context to store deleted tasks in session

Use **Axios** to connect React to Django:

Example in `TaskList.js`:

```javascript
import axios from 'axios';
import { useState, useEffect } from 'react';

function TaskList() {
    const [tasks, setTasks] = useState([]);

    useEffect(() => {
        axios.get('/api/tasks/')
            .then(res => setTasks(res.data))
            .catch(err => console.error(err));
    }, []);

    return (
        <div>
            <h2>Tasks</h2>
            <ul>
                {tasks.map(task => (
                    <li key={task.id}>
                        {task.title} - {task.completed ? 'Done' : 'Pending'}
                    </li>
                ))}
            </ul>
        </div>
    );
}

export default TaskList;
```

Style it with Bootstrap (`npm install bootstrap`) if you want.


## 4. Authentication (Bonus)

You can also:

- Use Django built-in login system (`rest_framework.authtoken` or sessions)
- Restrict tasks to logged-in users

* Add login/logout endpoints.
* Add login page in React using Axios POST to Django `/login/`.


## 5. Undo Feature (Bonus)

- When deleting a task, **save it in localStorage or React Context**.
- Allow user to **undo the last delete**.

---

# Helpful Resources

- [Create React App Docs](https://react.dev/learn/start-a-new-react-project)
- [Django REST Framework Docs](https://www.django-rest-framework.org/)
- [Axios Docs](https://axios-http.com/docs/intro)
- [React Router Docs](https://reactrouter.com/en/main)


# Final Project Checklist

* React app runs on `localhost:3000`
* Django API runs on `localhost:8000`
* Full CRUD functionality
* (Bonus) User login/logout
* (Bonus) Undo delete

# ✏️ Deliverables

- **GitHub repo** (recommended)
- Include a **README** explaining:
  - How to run Django
  - How to run React
  - How Undo works
  - Any bonus features

## Sample Solution to Add Authentication

We will use **Django's session-based login** (no token, no OAuth — keeping it beginner-friendly).

## 🛠 1. Django Backend: Enable Login and Logout

First, update Django backend to allow login/logout **through API**.

---

### 1.1 Install Django REST Auth Tools

```bash
pip install djangorestframework
pip install djangorestframework-simplejwt
```

(We will NOT use JWT now — just sessions.)

---

### 1.2 Update `settings.py`

```python
INSTALLED_APPS += [
    'rest_framework',
    'rest_framework.authtoken',
    'django.contrib.sites',
]

# Needed for Django login
SITE_ID = 1

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
    ],
}
```

This tells Django REST Framework to **use session-based authentication**.

---

### 1.3 Create Login / Logout Views

In `tasks/views.py`, add:

```python
from django.contrib.auth import authenticate, login, logout
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status

@api_view(['POST'])
def login_view(request):
    username = request.data.get('username')
    password = request.data.get('password')

    user = authenticate(request, username=username, password=password)
    if user is not None:
        login(request, user)
        return Response({'message': 'Logged in successfully'})
    else:
        return Response({'error': 'Invalid credentials'}, status=status.HTTP_400_BAD_REQUEST)

@api_view(['POST'])
def logout_view(request):
    logout(request)
    return Response({'message': 'Logged out successfully'})
```

### 1.4 Wire Up URLs

In `tasks/urls.py`, add:

```python
from .views import login_view, logout_view

urlpatterns = [
    path('login/', login_view),
    path('logout/', logout_view),
    path('', include(router.urls)),
]
```

Now you have:

| URL | Method | Action |
|:----|:-------|:-------|
| `/api/login/` | POST | Log a user in |
| `/api/logout/` | POST | Log a user out |


### 1.5 Protect the API

You can **require login** for the Task API:

In `tasks/views.py`, update `TaskViewSet`:

```python
from rest_framework.permissions import IsAuthenticated

class TaskViewSet(viewsets.ModelViewSet):
    queryset = Task.objects.all().order_by('-created_at')
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated]
```

Now, you must be logged in to see or edit tasks!


# 2. React Frontend: Build Login Page

### 2.1 Create `LoginPage.js`

```javascript
import { useState } from 'react';
import axios from 'axios';
import { useNavigate } from 'react-router-dom';

function LoginPage() {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');
    const [error, setError] = useState('');
    const navigate = useNavigate();

    const handleSubmit = async (e) => {
        e.preventDefault();
        try {
            await axios.post('/api/login/', { username, password }, { withCredentials: true });
            navigate('/tasks');
        } catch (err) {
            setError('Login failed. Check your credentials.');
        }
    };

    return (
        <div>
            <h2>Login</h2>
            {error && <p style={{ color: 'red' }}>{error}</p>}
            <form onSubmit={handleSubmit}>
                <div>
                    <label>Username:</label>
                    <input value={username} onChange={e => setUsername(e.target.value)} />
                </div>
                <div>
                    <label>Password:</label>
                    <input type="password" value={password} onChange={e => setPassword(e.target.value)} />
                </div>
                <button type="submit">Login</button>
            </form>
        </div>
    );
}

export default LoginPage;
```

### 2.2 Axios Configuration

Make sure **Axios sends cookies** with every request:

In `App.js` or at top of your project:

```javascript
axios.defaults.withCredentials = true;
```

This will allow Django sessions to work!

### 2.3 Setup Routes

In `App.js`:

```javascript
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import LoginPage from './LoginPage';
import TaskList from './TaskList';

function App() {
    return (
        <Router>
            <Routes>
                <Route path="/" element={<LoginPage />} />
                <Route path="/tasks" element={<TaskList />} />
            </Routes>
        </Router>
    );
}

export default App;
```

After login, redirect to `/tasks`.


### 2.4 Logout Button

In `TaskList.js`, add:

```javascript
const handleLogout = async () => {
    await axios.post('/api/logout/', {}, { withCredentials: true });
    window.location.href = '/'; // Redirect to login
};

// In your component's return()
<button onClick={handleLogout}>Logout</button>
```

# Review: What You Will Learn

| Concept | Tool |
|:--------|:-----|
| Session-based authentication | Django REST Framework |
| Login and logout API design | Django Views |
| Handling sessions/cookies in React | Axios + React Router |
| Protecting API resources | Django DRF Permissions |
| Simple user experience | Redirects and error handling |


# Final Working Flow

1. **Visit `/`** → login form.
2. **POST /api/login/** → authenticate.
3. **On success** → redirect to `/tasks`.
4. **Tasks API** only works **if logged in**.
5. **Click Logout** → **POST /api/logout/** → redirect back to login.

---

# 📎 Notes

- Django **stores the session ID in a cookie**.
- Axios must **send cookies** (`withCredentials: true`).
- Django backend matches the session ID to a logged-in user.
- If not authenticated, **403 Forbidden** will be returned automatically.

## Add User Registration

## 1. Django Backend: Create a Register View

First, allow users to **create an account** from the frontend.

In `tasks/views.py`, add:

```python
from django.contrib.auth.models import User
from rest_framework import status

@api_view(['POST'])
def register_view(request):
    username = request.data.get('username')
    password = request.data.get('password')

    if not username or not password:
        return Response({'error': 'Username and password are required.'}, status=status.HTTP_400_BAD_REQUEST)

    if User.objects.filter(username=username).exists():
        return Response({'error': 'Username already exists.'}, status=status.HTTP_400_BAD_REQUEST)

    user = User.objects.create_user(username=username, password=password)
    return Response({'message': 'User created successfully'})
```

This creates a new Django user **and hashes the password properly**!

### 1.1 Wire Up the URL

In `tasks/urls.py`, add:

```python
from .views import register_view

urlpatterns = [
    path('register/', register_view),
    path('login/', login_view),
    path('logout/', logout_view),
    path('', include(router.urls)),
]
```

Now your Django backend has:

| URL | Method | Action |
|:----|:-------|:-------|
| `/api/register/` | POST | Create a new user |

---

## 2. React Frontend: Build Register Page

---

### 2.1 Create `RegisterPage.js`

```javascript
import { useState } from 'react';
import axios from 'axios';
import { useNavigate, Link } from 'react-router-dom';

function RegisterPage() {
    const [username, setUsername] = useState('');
    const [password, setPassword] = useState('');
    const [error, setError] = useState('');
    const [success, setSuccess] = useState('');
    const navigate = useNavigate();

    const handleSubmit = async (e) => {
        e.preventDefault();
        try {
            await axios.post('/api/register/', { username, password });
            setSuccess('Registration successful. Please log in.');
            setTimeout(() => navigate('/'), 2000); // Redirect after 2 sec
        } catch (err) {
            setError('Registration failed: ' + (err.response?.data?.error || 'Unknown error'));
        }
    };

    return (
        <div>
            <h2>Register</h2>
            {error && <p style={{ color: 'red' }}>{error}</p>}
            {success && <p style={{ color: 'green' }}>{success}</p>}
            <form onSubmit={handleSubmit}>
                <div>
                    <label>Username:</label>
                    <input value={username} onChange={e => setUsername(e.target.value)} />
                </div>
                <div>
                    <label>Password:</label>
                    <input type="password" value={password} onChange={e => setPassword(e.target.value)} />
                </div>
                <button type="submit">Register</button>
            </form>
            <p>Already have an account? <Link to="/">Login</Link></p>
        </div>
    );
}

export default RegisterPage;
```

---

### 2.2 Update React Router

In `App.js`, add the route:

```javascript
import RegisterPage from './RegisterPage';

function App() {
    return (
        <Router>
            <Routes>
                <Route path="/" element={<LoginPage />} />
                <Route path="/register" element={<RegisterPage />} />
                <Route path="/tasks" element={<TaskList />} />
            </Routes>
        </Router>
    );
}
```

---

### 2.3 Add "Register" Link on Login Page

In `LoginPage.js`, add at the bottom:

```javascript
<p>Don't have an account? <Link to="/register">Register here</Link></p>
```

Now users can **click "Register here"** from the login page!

---

# Full User Flow

| Page | Action |
|:-----|:-------|
| `/register` | Register new account |
| `/` (Login) | Login with account |
| `/tasks` | View tasks (only if logged in) |
| Logout | End session and return to login page |

---

# Things You Will Learn

| Concept | Skill |
|:--------|:------|
| User creation | Django's `create_user()` |
| Authentication + session cookies | Django REST Framework + React Axios |
| Redirects and handling success/failure | React State + Navigation |
| Building simple authentication flows | Real-world project experience |
