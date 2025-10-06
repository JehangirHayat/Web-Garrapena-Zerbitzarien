# 📘 Django Apunteak – README Completo

## ⚡ Introducción

Este documento recopila todos los pasos para crear y gestionar un proyecto **Django** desde cero. Incluye:

- 🚀 Instalación y entorno virtual
- 🛠️ Creación de proyectos y aplicaciones
- 📂 Modelos, migraciones y base de datos
- 🔄 CRUD completo (Create, Read, Update, Delete)
- 🔍 Consultas avanzadas
- 🖥️ Panel de administración
- 🌍 Parámetros en URL
- 🎨 Templates y herencia
- 📝 Formularios y vistas genéricas
- 📦 Archivos estáticos (CSS, JS, imágenes)

---

## 1️⃣ Preparación del Proyecto

### Crear entorno de trabajo

```bash
mkdir Django-project
cd Django-project
```

### Instalar Virtualenv

```bash
python -m pip install virtualenv
python -m virtualenv venv
```

### Activar entorno virtual

- **Windows**

```bash
.venv\Scripts/activate
```

- **Linux / Mac**

```bash
source venv/bin/activate
```

### Instalar Django

```bash
pip install django
django-admin --version
```

---

## 2️⃣ Crear Proyecto y Aplicación

### Crear proyecto

```bash
django-admin startproject mysite .
cd mysite
```

### Iniciar servidor

```bash
python manage.py runserver
# o en puerto distinto
python manage.py runserver 3000
```

### Crear aplicación

```bash
python manage.py startapp myapp
```

### Registrar aplicación

`mysite/settings.py`

```python
INSTALLED_APPS = [
    ...,
    'myapp',
]
```

---

## 3️⃣ Modelos y Migraciones

### Modelos

```python
from django.db import models

class Project(models.Model):
    name = models.CharField(max_length=200)
    def __str__(self):
        return self.name

class Task(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField()
    project = models.ForeignKey(Project, on_delete=models.CASCADE)
    done = models.BooleanField(default=False)
    def __str__(self):
        return f"{self.title} - {self.project.name}"
```

### Migraciones

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## 4️⃣ Operaciones CRUD

### CREATE

```python
proiektu1 = Project.objects.create(name="Web Proiektua")
ataza1 = Task.objects.create(
    title="DB diseinatu",
    description="Modelo ER",
    project=proiektu1,
    done=False
)
```

### READ

```python
# Todos
Project.objects.all()
Task.objects.all()

# Filtrar
Task.objects.filter(done=True)
Task.objects.filter(project__name="Web Proiektua")

# Uno solo
Project.objects.get(id=1)
```

### UPDATE

```python
proiektua = Project.objects.get(id=1)
proiektua.name = "Web Proiektua Eguneratua"
proiektua.save()

Task.objects.filter(project=proiektu1).update(done=True)
```

### DELETE

```python
Task.objects.get(id=3).delete()
Task.objects.filter(done=True).delete()
Project.objects.all().delete()
```

---

## 5️⃣ Consultas Avanzadas

```python
from django.db.models import Q

# Tareas urgentes
Task.objects.filter(
    Q(project__name="Web Proiektua") | Q(project__name="Mugikorrerako Proiektua"),
    done=False
)

# Ordenar
Task.objects.order_by('-project__name', 'title')

# Contar
Project.objects.count()
Task.objects.filter(done=True).count()

# Primero y último
Project.objects.first()
Task.objects.last()

# Comprobar existencia
Project.objects.filter(name="Web Proiektua").exists()

# Crear o actualizar
Project.objects.get_or_create(name="Proiektu Berria")
Task.objects.update_or_create(
    title="Existitzen den Ataza",
    defaults={'done': True, 'description': 'Eguneratuta'}
)
```

---

## 6️⃣ Panel de Administración

### Crear superusuario

```bash
python manage.py createsuperuser
```

### Registrar modelos

```python
from django.contrib import admin
from .models import Project, Task

admin.site.register(Project)
admin.site.register(Task)
```

### Acceso

👉 http://127.0.0.1:8000/admin

---

## 7️⃣ Parámetros en URL

### urls.py

```python
from django.urls import path
from . import views

urlpatterns = [
    path('hello/<str:username>/', views.hello, name='hello'),
    path('tasks/<int:id>/', views.tasks, name='tasks'),
]
```

### views.py

```python
from django.shortcuts import get_object_or_404
from django.http import HttpResponse
from .models import Task

def hello(request, username):
    return HttpResponse(f"<h2>Hello {username}</h2>")

def tasks(request, id):
    task = get_object_or_404(Task, id=id)
    return HttpResponse(f"Task: {task.title}")
```

---

## 8️⃣ Templates y Herencia

### Base

`templates/layouts/base.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <title>{% block title %}My Django App{% endblock %}</title>
  </head>
  <body>
    <nav>
      <a href="/">Home</a> | <a href="/projects/">Projects</a> |
      <a href="/tasks/">Tasks</a>
    </nav>
    <main>{% block content %}{% endblock %}</main>
    <footer>
      <small>Django App</small>
    </footer>
  </body>
</html>
```

### Herencia

`templates/projects.html`

```html
{% extends "layouts/base.html" %} {% block title %}Proiektuak{% endblock %} {%
block content %}
<h1>Proiektu Zerrenda</h1>
<ul>
  {% for project in projects %}
  <li>{{ project.name }}</li>
  {% endfor %}
</ul>
{% endblock %}
```

---

## 9️⃣ Formularios y CRUD Vistas

### views.py

```python
from django.views.generic import CreateView, UpdateView, DeleteView
from django.urls import reverse_lazy
from .models import Task

class AtazaCreateView(CreateView):
    model = Task
    template_name = 'task_form.html'
    fields = ['title', 'description', 'project']
    success_url = reverse_lazy('tasks')

class AtazaUpdateView(UpdateView):
    model = Task
    template_name = 'task_form.html'
    fields = ['title', 'description', 'project', 'done']
    success_url = reverse_lazy('tasks')

class AtazaDeleteView(DeleteView):
    model = Task
    template_name = 'task_confirm_delete.html'
    success_url = reverse_lazy('tasks')
```

### urls.py

```python
path('tasks/create/', AtazaCreateView.as_view(), name='task_create'),
path('tasks/<int:pk>/edit/', AtazaUpdateView.as_view(), name='task_update'),
path('tasks/<int:pk>/delete/', AtazaDeleteView.as_view(), name='task_delete'),
```

---

## 🔟 Archivos Estáticos

### Estructura

```
myapp/
 └── static/
      ├── css/
      ├── js/
      └── img/
```

### Uso en templates

```html
{% load static %}
<link rel="stylesheet" href="{% static 'css/styles.css' %}" />
<img src="{% static 'img/logo.png' %}" alt="logo" />
```

---

## ▶️ Ejecución Final

1. Activar entorno virtual
2. Lanzar servidor:

```bash
python manage.py runserver
```

3. Entrar en:
   - http://127.0.0.1:8000/ → App
   - http://127.0.0.1:8000/admin/ → Admin

---
