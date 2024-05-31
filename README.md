# Django-React-Todo-App-with-Authentication-Hindi-

# Django + React Todo App with Authentication (Hinglish)

## Prerequisites (Jarurat)

Pehle ye cheezein install honi chahiye:

- Python 3.x: Backend ke liye programming language.
- Node.js aur npm: React frontend run aur manage karne ke liye.
- Django: Python ka high-level web framework.
- Create React App (CRA): Ek tool jo ek command se modern web app set up karta hai.
- Virtualenv: Python environments ko isolated banane ke liye.

## Backend: Django Setup

### Step 1: Django Project Setup Karna

1. **Django Install Karein**:
   ```bash
   pip install django
   ```
   Ye command Django ko install kar degi.

2. **Django Project Banayein**:
   ```bash
   django-admin startproject backend
   cd backend
   ```
   Is command se `backend` naam ka naya Django project shuru hoga.

3. **Django App Banayein**:
   ```bash
   python manage.py startapp todos
   ```
   Ye command `todos` naam ka ek naya app banayegi. Django projects ko modules mein divide karta hai.

4. **Django Rest Framework Install Karein**:
   ```bash
   pip install djangorestframework
   ```
   Ye library web APIs banana easy banati hai.

5. **Django REST framework simple JWT Install Karein**:
   ```bash
   pip install djangorestframework-simplejwt
   ```
   JWT authentication ke liye ye library use hoti hai jo stateless hoti hai aur modern web apps ke liye ideal hai.

6. **`backend/settings.py` mein `rest_framework` aur `todos` add karein**:
   ```python
   INSTALLED_APPS = [
       ...
       'rest_framework',
       'todos',
       'rest_framework_simplejwt',
   ]
   ```

7. **Django Rest Framework aur Simple JWT configure karein `backend/settings.py` mein**:
   ```python
   REST_FRAMEWORK = {
       'DEFAULT_AUTHENTICATION_CLASSES': (
           'rest_framework_simplejwt.authentication.JWTAuthentication',
       ),
   }
   ```
   Ye configuration ensure karti hai ki hamari API JWT authentication use kare.

8. **Migrations Run Karein**:
   ```bash
   python manage.py migrate
   ```
   Ye command database tables setup karti hai hamare project ke liye.

### Step 2: Models, Serializers, aur Views Banana

1. **`todos/models.py` update karein Todo model ke saath**:
   ```python
   from django.db import models
   from django.contrib.auth.models import User

   class Todo(models.Model):
       user = models.ForeignKey(User, on_delete=models.CASCADE)
       title = models.CharField(max_length=200)
       description = models.TextField(blank=True)
       completed = models.BooleanField(default=False)
       created_at = models.DateTimeField(auto_now_add=True)

       def __str__(self):
           return self.title
   ```
   Ye Todo model structure define karta hai todo item ka, jo har item ko ek user se link karta hai.

2. **Serializers banayein `todos/serializers.py` mein**:
   ```python
   from rest_framework import serializers
   from .models import Todo

   class TodoSerializer(serializers.ModelSerializer):
       class Meta:
           model = Todo
           fields = '__all__'
   ```
   Serializers complex data types (jaise querysets aur model instances) ko native Python data types mein convert karte hain, taaki unhe JSON mein render kiya ja sake.

3. **Views banayein `todos/views.py` mein**:
   ```python
   from rest_framework.decorators import api_view, permission_classes
   from rest_framework.response import Response
   from rest_framework import status
   from rest_framework.permissions import IsAuthenticated
   from .models import Todo
   from .serializers import TodoSerializer

   @api_view(['GET'])
   @permission_classes([IsAuthenticated])
   def get_todos(request):
       todos = request.user.todo_set.all()
       serializer = TodoSerializer(todos, many=True)
       return Response(serializer.data)

   @api_view(['POST'])
   @permission_classes([IsAuthenticated])
   def create_todo(request):
       serializer = TodoSerializer(data=request.data)
       if serializer.is_valid():
           serializer.save(user=request.user)
           return Response(serializer.data, status=status.HTTP_201_CREATED)
       return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

   @api_view(['GET', 'PUT', 'DELETE'])
   @permission_classes([IsAuthenticated])
   def manage_todo(request, pk):
       try:
           todo = Todo.objects.get(pk=pk, user=request.user)
       except Todo.DoesNotExist:
           return Response(status=status.HTTP_404_NOT_FOUND)

       if request.method == 'GET':
           serializer = TodoSerializer(todo)
           return Response(serializer.data)

       elif request.method == 'PUT':
           serializer = TodoSerializer(todo, data=request.data)
           if serializer.is_valid():
               serializer.save()
               return Response(serializer.data)
           return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

       elif request.method == 'DELETE':
           todo.delete()
           return Response(status=status.HTTP_204_NO_CONTENT)
   ```
   Ye views CRUD operations handle karte hain (Create, Read, Update, Delete) Todo items ke liye.

4. **URLs add karein `todos/urls.py` mein**:
   ```python
   from django.urls import path
   from . import views

   urlpatterns = [
       path('todos/', views.get_todos),
       path('todos/create/', views.create_todo),
       path('todos/<int:pk>/', views.manage_todo),
   ]
   ```
   Ye URLs ko appropriate views ke saath route karte hain.

5. **`todos` URLs ko include karein `backend/urls.py` mein**:
   ```python
   from django.contrib import admin
   from django.urls import path, include

   urlpatterns = [
       path('admin/', admin.site.urls),
       path('api/', include('todos.urls')),
   ]
   ```

### Step 3: JWT Authentication Setup Karna

1. **JWT endpoints add karein `backend/urls.py` mein**:
   ```python
   from rest_framework_simplejwt.views import (
       TokenObtainPairView,
       TokenRefreshView,
   )

   urlpatterns = [
       ...
       path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
       path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
   ]
   ```
   Ye endpoints JWT tokens obtain aur refresh karne ke liye use hote hain.

### Step 4: API Test Karna

1. **Superuser banayein Django admin access karne ke liye**:
   ```bash
   python manage.py createsuperuser
   ```
   
2. **Django server run karein**:
   ```bash
   python manage.py runserver
   ```

3. **Django admin access karein `http://127.0.0.1:8000/admin` par aur Todo items aur users create karein.**

## Frontend: React Setup

### Step 1: React Project Setup Karna

1. **Create React App install karein agar pehle se nahi hai**:
   ```bash
   npx create-react-app frontend
   cd frontend
   ```
   Ye command `frontend` naam ka naya React project shuru karegi.

2. **Axios aur JWT-decode install karein**:
   ```bash
   npm install axios jwt-decode
   ```
   Axios ek promise-based HTTP client hai browser aur node.js ke liye. JWT-decode JSON web tokens ko decode karne ke liye use hota hai.

### Step 2: React Components Banana

1. **Folder structure banayein**:
   ```bash
   mkdir -p src/components src/services
   ```

2. **`authService.js` banayein `src/services` mein authentication handle karne ke liye**:
   ```javascript
   import axios from 'axios';
   const API_URL = 'http://127.0.0.1:8000/api/token/';

   const login = (username, password) => {
       return axios.post(API_URL, {
           username,
           password
       });
   };

   const authService = {
       login,
   };

   export default authService;
   ```
   Ye service user login handle karti hai, credentials ko backend ko bhej kar JWT tokens receive karti hai.

3. **`todoService.js` banayein `src/services` mein Todo API requests handle karne ke liye**:
   ```javascript
   import axios from 'axios';

   const API_URL = 'http://127.0.0.1:8000/api/todos/';

   const getTodos = (token) => {
       return axios.get(API_URL, {
           headers: { Authorization: `Bearer ${token}` }
       });
   };

   const createTodo = (token, todo) => {
       return axios.post(`${API_URL}create/`, todo, {
           headers: { Authorization: `Bearer ${token}` }
       });
   };

   const updateTodo = (token, id, todo) => {
       return axios.put(`${API_URL}${id}/`, todo, {
           headers: { Authorization: `Bearer ${token}` }
       });
   };

   const deleteTodo = (token, id) => {
       return axios.delete(`${API

_URL}${id}/`, {
           headers: { Authorization: `Bearer ${token}` }
       });
   };

   const todoService = {
       getTodos,
       createTodo,
       updateTodo,
       deleteTodo,
   };

   export default todoService;
   ```
   Ye service CRUD operations handle karti hai Todo items ke liye.

4. **`Login.js` banayein `src/components` mein**:
   ```javascript
   import React, { useState } from 'react';
   import authService from '../services/authService';

   function Login({ setToken }) {
       const [username, setUsername] = useState('');
       const [password, setPassword] = useState('');

       const handleSubmit = async (e) => {
           e.preventDefault();
           try {
               const { data } = await authService.login(username, password);
               setToken(data.access);
           } catch (error) {
               console.error("Invalid credentials");
           }
       };

       return (
           <form onSubmit={handleSubmit}>
               <div>
                   <label>Username:</label>
                   <input type="text" value={username} onChange={(e) => setUsername(e.target.value)} />
               </div>
               <div>
                   <label>Password:</label>
                   <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
               </div>
               <button type="submit">Login</button>
           </form>
       );
   }

   export default Login;
   ```
   Ye component user login handle karta hai, username aur password capture karta hai, aur backend ko bhej kar JWT token obtain karta hai.

5. **`TodoList.js` banayein `src/components` mein**:
   ```javascript
   import React, { useEffect, useState } from 'react';
   import todoService from '../services/todoService';

   function TodoList({ token }) {
       const [todos, setTodos] = useState([]);
       const [newTodo, setNewTodo] = useState({ title: '', description: '' });

       useEffect(() => {
           const fetchTodos = async () => {
               try {
                   const { data } = await todoService.getTodos(token);
                   setTodos(data);
               } catch (error) {
                   console.error("Error fetching todos");
               }
           };

           fetchTodos();
       }, [token]);

       const handleCreateTodo = async (e) => {
           e.preventDefault();
           try {
               const { data } = await todoService.createTodo(token, newTodo);
               setTodos([...todos, data]);
               setNewTodo({ title: '', description: '' });
           } catch (error) {
               console.error("Error creating todo");
           }
       };

       const handleUpdateTodo = async (id, updatedTodo) => {
           try {
               const { data } = await todoService.updateTodo(token, id, updatedTodo);
               setTodos(todos.map(todo => (todo.id === id ? data : todo)));
           } catch (error) {
               console.error("Error updating todo");
           }
       };

       const handleDeleteTodo = async (id) => {
           try {
               await todoService.deleteTodo(token, id);
               setTodos(todos.filter(todo => todo.id !== id));
           } catch (error) {
               console.error("Error deleting todo");
           }
       };

       return (
           <div>
               <h2>Todo List</h2>
               <form onSubmit={handleCreateTodo}>
                   <input
                       type="text"
                       placeholder="Title"
                       value={newTodo.title}
                       onChange={(e) => setNewTodo({ ...newTodo, title: e.target.value })}
                   />
                   <input
                       type="text"
                       placeholder="Description"
                       value={newTodo.description}
                       onChange={(e) => setNewTodo({ ...newTodo, description: e.target.value })}
                   />
                   <button type="submit">Add Todo</button>
               </form>
               <ul>
                   {todos.map(todo => (
                       <li key={todo.id}>
                           <input
                               type="text"
                               value={todo.title}
                               onChange={(e) => handleUpdateTodo(todo.id, { ...todo, title: e.target.value })}
                           />
                           <input
                               type="text"
                               value={todo.description}
                               onChange={(e) => handleUpdateTodo(todo.id, { ...todo, description: e.target.value })}
                           />
                           <button onClick={() => handleDeleteTodo(todo.id)}>Delete</button>
                       </li>
                   ))}
               </ul>
           </div>
       );
   }

   export default TodoList;
   ```
   Ye component Todo items ko list karta hai, aur user ko Todo items create, update, aur delete karne deta hai.

6. **`App.js` update karein `Login` aur `TodoList` components ko include karne ke liye**:
   ```javascript
   import React, { useState } from 'react';
   import Login from './components/Login';
   import TodoList from './components/TodoList';

   function App() {
       const [token, setToken] = useState('');

       return (
           <div className="App">
               {!token ? <Login setToken={setToken} /> : <TodoList token={token} />}
           </div>
       );
   }

   export default App;
   ```
   Ye main App component conditionally `Login` ya `TodoList` component render karta hai based on whether user authenticated hai ya nahi.

### Step 3: React App Run Karna

1. **React application start karein**:
   ```bash
   npm start
   ```
   Ye command React app ko local server par run karegi.

## Final Notes

1. **Ensure Django server `http://127.0.0.1:8000` par run ho raha hai aur React app `http://localhost:3000` par.**
2. **Django admin ke through users create karein aur unhe React app se login karne dein.**

## README.md

Here's a `README.md` file jo setup process explain karta hai:

```markdown
# Django + React Todo App with Authentication

Ye ek Todo application hai jo Django (backend) aur React (frontend) se bani hai. Is app mein user authentication aur Todo items ke liye CRUD operations included hain.

## Prerequisites

- Python 3.x
- Node.js aur npm
- Django
- Create React App (CRA)
- Virtualenv

## Setup Instructions

### Backend (Django)

1. **Django install karein aur ek Django project banayein:**
   ```bash
   pip install django
   django-admin startproject backend
   cd backend
   ```

2. **Ek Django app banayein aur dependencies install karein:**
   ```bash
   python manage.py startapp todos
   pip install djangorestframework djangorestframework-simplejwt
   ```

3. **`backend/settings.py` update karein:**
   - `INSTALLED_APPS` mein `rest_framework` aur `todos` add karein.
   - Django Rest Framework aur Simple JWT configure karein.

4. **Models, serializers, aur views banayein `todos` app mein.**

5. **URLs `todos/urls.py` mein add karein aur `backend/urls.py` mein include karein.**

6. **Migrations run karein aur ek superuser banayein:**
   ```bash
   python manage.py migrate
   python manage.py createsuperuser
   ```

7. **Django server run karein:**
   ```bash
   python manage.py runserver
   ```

### Frontend (React)

1. **Ek React app banayein aur dependencies install karein:**
   ```bash
   npx create-react-app frontend
   cd frontend
   npm install axios jwt-decode
   ```

2. **Authentication aur Todo API requests ke liye services banayein.**

3. **React components banayein login aur Todo list management ke liye.**

4. **`App.js` update karein `Login` aur `TodoList` components ko include karne ke liye.**

5. **React application start karein:**
   ```bash
   npm start
   ```

## Running the Application

- Ensure Django server `http://127.0.0.1:8000` par run ho raha hai.
- Ensure React app `http://localhost:3000` par run ho raha hai.
- Django admin ke through users create karein aur unhe React app se login karne dein.

## License

Ye project MIT License ke under licensed hai.
```

Ye guide Django backend aur React frontend setup karne ke steps cover karti hai, including user authentication aur CRUD operations ke liye ek Todo application. Har step mein tools aur practices ko use karne ka reason bhi explain kiya gaya hai, taaki setup process ko achi tarah samjha ja sake.
