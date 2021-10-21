# CLASS BASED VIEWS PRECLASS SETUP

```bash
# CREATING VIRTUAL ENVIRONMENT
# windows
py -m venv env
# windows other option
python -m venv env
# linux / Mac OS
vitualenv env

# ACTIVATING ENVIRONMENT
# windows
.\env\Scripts\activate
# linux / Mac OS
source env/bin/activate

# PACKAGE INSTALLATION
# if pip does not work try pip3 in linux/Mac OS
pip install django
# alternatively python -m pip install django
pip install python-decouple
django-admin --version
django-admin startproject main .
```

go to terminal

```bash
py manage.py migrate
py manage.py runserver
```

click the link with CTRL key pressed in the terminal and see django rocket.

go to terminal, stop project, add app

```
py manage.py startapp fscohort
```

go to settings.py and add 'fscohort' app to installed apps and add below lines

```python
import os
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
MEDIA_URL = '/media/'
```

create these folders at project level as /media/student

go to fscohort/models.py

```python
from django.db import models

# Create your models here.

class Student(models.Model):
    first_name = models.CharField(max_length=50)
    last_name = models.CharField(max_length=50)

    email = models.EmailField(max_length=154, unique=True, blank=True, null=True)
    phone = models.CharField(max_length=50, unique=True, blank=True, null=True)

    GENDER =(
        ("1", "Female"),
        ("2", "Male"),
        ("3", "Other"),
        ("4", "Prefer Not Say"),
    )

    gender = models.CharField(max_length=50, choices=GENDER)
    number = models.IntegerField(unique=True, blank=True, null=True)
    image = models.ImageField(upload_to="student/", default="avatar.png")

    def __str__(self):
        return f"{self.number} {self.first_name} {self.last_name}"
```

go to terminal

```bash
pip install pillow
pip freeze > requirements.txt
py manage.py makemigrations
py manage.py migrate
```

go to students/forms.py

```python
from django import  forms
from django.forms import fields
from .models import Student


class StudentForm(forms.ModelForm):
    class Meta:
        model = Student
        fields = "__all__"
```

create template folder as fscohort/templates/fscohort

base.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CW Student App</title>
  </head>

  <body>
    <h1>
      <hr />
      <center>Clarusway</center>
      <hr />
    </h1>
    <a href="{% url 'home' %}">HOME</a> {% block content %} {% endblock content
    %}
  </body>
</html>
```

home.html

```html
{% extends 'fscohort/base.html' %} {% block content %}

<h2>
  <center>
    Welcome To Student App <br />
    (Backend Team)
    <hr />
    <br />
    <a href="{% url 'list' %}">LIST</a>
    <a href="{% url 'add' %}">ADD</a>
  </center>
</h2>

{% endblock content %}
```

student_add.html

```html
{% extends 'fscohort/base.html' %} {% block content %}

<h2>Student Add</h2>

<form action="" method="POST" enctype="multipart/form-data">
  {% csrf_token %} {{form.as_p}}

  <input type="submit" value="Add" />
</form>

{% endblock content %}
```

student_delete.html

```html
{% extends 'fscohort/base.html' %} {% block content %}
<h3>Delete Student</h3>
<div>
  <form action="" , method="POST">
    {% csrf_token %} Are you sure delete {{student}}?<br />
    <input type="submit" value="Yes" />
  </form>

  <a href="{% url 'list'%}"> <button>No</button></a>
</div>
{% endblock content %}
```

student_detail.html

```html
{% extends 'fscohort/base.html' %} {% block content %}

<h2>Student Detail</h2>

<div>
  <img src="{{student.image.url}}" alt="" />
</div>
<div>{{student.first_name}}</div>
<div>{{student.last_name}}</div>
<div>{{student.email}}</div>
<div>{{student.phone}}</div>
<div>{{student.number}}</div>

<div>
  <a href="{% url 'update' student.id %}">Edit</a>
  <a href="{% url 'delete' student.id %}">Delete</a>
</div>

{% endblock content %}
```

student_list.html

```html
{% extends 'fscohort/base.html' %} {% block content %}

<h2>Student List</h2>

<ul>
  {% for student in students %}
  <a href="{% url 'detail' student.id  %}">
    <li>{{student}}</li>
  </a>
  {% endfor %}
</ul>
{% endblock content %}
```

student_update.html

```html
{% extends 'fscohort/base.html' %} {% block content %}

<h2>Student Update</h2>

<form action="" method="POST" enctype="multipart/form-data">
  {% csrf_token %} {{form.as_p}}

  <input type="submit" value="Update" />
</form>

{% endblock content %}
```

go to fscohort/views.py

```python
from django.shortcuts import redirect, render
from django.http import HttpResponse
from .forms import StudentForm
from .models import Student
# Create your views here.

def home(request):
    return render(request, "fscohort/home.html")


def student_list(request):

    students = Student.objects.all()

    context = {
        "students":students
    }

    return render(request, "fscohort/student_list.html", context)

def student_add(request):
    form = StudentForm()

    if request.method == "POST":
        form = StudentForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect("list")


    context = {

       "form":form
    }

    return render(request, "fscohort/student_add.html", context)

def student_detail(request,id):
    student = Student.objects.get(id=id)
    context = {
        "student":student
    }

    return render(request, "fscohort/student_detail.html", context)

def student_update(request, id):

    student = Student.objects.get(id=id)

    form = StudentForm(instance=student)

    if request.method == "POST":
        form = StudentForm(request.POST, request.FILES, instance=student)
        if form.is_valid():
            form.save()
            return redirect("list")

    context= {

        "student":student,
        "form":form
    }

    return render(request, "fscohort/student_update.html", context)

def student_delete(request, id):

    student = Student.objects.get(id=id)

    if request.method == "POST":


        student.delete()
        return redirect("list")

    context= {
        "student":student
    }
    return render(request, "fscohort/student_delete.html",context)

```

go to main/urls.py

```python
from django.contrib import admin
from django.urls import path, include
from django.conf.urls.static import static
from django.conf import settings

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include("fscohort.urls")),
] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT) + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

go to fscohort/urls.py

```python
from django.urls import path
from .views import home,student_list, student_add, student_detail, student_update,student_delete

urlpatterns = [
    path('', home, name="home"),
    path('student_list/', student_list, name="list"),
    path('student_add/', student_add, name="add"),
    path('detail/<int:id>/', student_detail, name="detail"),
    path('update/<int:id>/', student_update, name="update"),
    path('delete/<int:id>/', student_delete, name="delete"),
]
```

go to terminal

```bash
py manage.py createsuperuser
```

go to fscohort/admin.py

```python
from django.contrib import admin
from .models import Student

# Register your models here.
admin.site.register(Student)

```

go to terminal and run server

```bash
py manage.py runserver
```

# INCLASS STARTS

## gitignore

add a gitignore file at same level as env folder, and check that it includes .env and /env lines

## Python Decouple

create a new file and name as .env at same level as env folder

copy your SECRET_KEY from settings.py into this .env file. Don't forget to remove quotation marks from SECRET_KEY

```
SECRET_KEY = django-insecure-)=b-%-w+0_^slb(exmy*mfiaj&wz6_fb4m&s=az-zs!#1^ui7j
```

go to settings.py, make amendments below

```python
from decouple import config

SECRET_KEY = config('SECRET_KEY')
```

## Class Based Views

Navigate to https://docs.djangoproject.com/en/3.2/topics/class-based-views/
Explain Documentation

## TemplateView

### Using TemplateView directly in urls.py

go to fscohort/urls.py and amend lines

```python
from django.urls import path
from .views import home,student_list, student_add, student_detail, student_update,student_delete

from django.views.generic import TemplateView # new line

urlpatterns = [
    # path('', home, name="home"), # comment out this line
    # rendering template in urls.py
    path('', TemplateView.as_view(template_name= "fscohort/home.html"), name="home"), # new line

    path('student_list/', student_list, name="list"),
    path('student_add/', student_add, name="add"),
    path('detail/<int:id>/', student_detail, name="detail"),
    path('update/<int:id>/', student_update, name="update"),
    path('delete/<int:id>/', student_delete, name="delete"),
]
```

### Using TemplateView in views.py

go to fscohort/views.py and add below lines

```python
from django.views.generic import TemplateView

class HomeView(TemplateView):
    template_name = "fscohort/home.html"
```

go to fscohort/urls.py and amend lines

```python
from django.urls import path
from .views import home,student_list, student_add, student_detail, student_update,student_delete
# from django.views.generic import TemplateView # commented
from .views import HomeView # new line

urlpatterns = [
    # path('', home, name="home"),
    # path('', TemplateView.as_view(template_name= "fscohort/home.html"), name="home"), # commented
    path('', HomeView.as_view(), name="home"), # new line
    path('student_list/', student_list, name="list"),
    path('student_add/', student_add, name="add"),
    path('detail/<int:id>/', student_detail, name="detail"),
    path('update/<int:id>/', student_update, name="update"),
    path('delete/<int:id>/', student_delete, name="delete"),
]
```

## ListView

navigate to https://docs.djangoproject.com/en/3.2/ref/class-based-views/generic-display/#listview

go to fscohort/views.py and add below lines

```python
from django.views.generic import ListView
class StudentListView(ListView):
    model = Student
    # default template name : # app/modelname_list.html
    # this fits our template name no need to use this time
    # template_name = "fscohort/student_list.html"
    context_object_name = 'students' # default context name : object_list
    paginate_by = 10
```

go to fscohort/urls.py and amend lines

```python
from django.urls import path
from .views import home,student_list, student_add, student_detail, student_update,student_delete
# from django.views.generic import TemplateView
from .views import HomeView, StudentListView # newly added

urlpatterns = [
    # path('', home, name="home"),
    # path('', TemplateView.as_view(template_name= "fscohort/home.html"), name="home"),
    path('', HomeView.as_view(), name="home"),
    # path('student_list/', student_list, name="list"), # commented
    path('student_list/', StudentListView.as_view(), name="list"), # new line
    path('student_add/', student_add, name="add"),
    path('detail/<int:id>/', student_detail, name="detail"),
    path('update/<int:id>/', student_update, name="update"),
    path('delete/<int:id>/', student_delete, name="delete"),
]
```

## DetailView

go to fscohort/views.py and add below lines

```python
from django.views.generic import DetailView
class StudentDetailView(DetailView):
    model = Student
    pk_url_kwarg = 'id'
```

go to fscohort/urls.py and amend lines

```python
from django.urls import path
from .views import home,student_list, student_add, student_detail, student_update,student_delete
# from django.views.generic import TemplateView
from .views import HomeView, StudentListView, StudentDetailView # newly added

urlpatterns = [
    # path('', home, name="home"),
    # path('', TemplateView.as_view(template_name= "fscohort/home.html"), name="home"),
    path('', HomeView.as_view(), name="home"),
    # path('student_list/', student_list, name="list"),
    path('student_list/', StudentListView.as_view(), name="list"),
    path('student_add/', student_add, name="add"),
    # path('detail/<int:id>/', student_detail, name="detail"), # commented
    path('detail/<int:id>/', StudentDetailView.as_view(), name="detail"), # new line
    path('update/<int:id>/', student_update, name="update"),
    path('delete/<int:id>/', student_delete, name="delete"),
]
```

## CreateView

go to fscohort/views.py and add below lines

```python
from django.views.generic import CreateView
from django.urls import reverse_lazy

class StudentCreateView(CreateView):
    model = Student
    form_class = StudentForm
    template_name = "fscohort/student_add.html" # default name app/modelname_form.html
    success_url = reverse_lazy("list")
```

go to fscohort/urls.py and amend lines

```python
from django.urls import path
from .views import home,student_list, student_add, student_detail, student_update,student_delete
# from django.views.generic import TemplateView
from .views import HomeView, StudentListView, StudentDetailView, StudentCreateView # newly added

urlpatterns = [
    # path('', home, name="home"),
    # path('', TemplateView.as_view(template_name= "fscohort/home.html"), name="home"),
    path('', HomeView.as_view(), name="home"),
    # path('student_list/', student_list, name="list"),
    path('student_list/', StudentListView.as_view(), name="list"),
    # path('student_add/', student_add, name="add"), # commented
    path('student_add/', StudentCreateView.as_view(), name="add"), # newline
    # path('detail/<int:id>/', student_detail, name="detail"),
    path('detail/<int:id>/', StudentDetailView.as_view(), name="detail"),
    path('update/<int:id>/', student_update, name="update"),
    path('delete/<int:id>/', student_delete, name="delete"),
]
```

## UpdateView

go to fscohort/views.py and add below lines

```python
from django.views.generic import UpdateView
class StudentUpdateView(UpdateView):
    model = Student
    form_class = StudentForm
    template_name = "fscohort/student_update.html" # default app/modelname_form.html
    success_url = '/student_list/' #'reverse_lazy("list")
    # pk_url_kwarg = 'id'
```

go to fscohort/urls.py and amend lines

```python
from django.urls import path
from .views import home,student_list, student_add, student_detail, student_update,student_delete
# from django.views.generic import TemplateView
from .views import HomeView, StudentListView, StudentDetailView, StudentCreateView, StudentUpdateView # newly added

urlpatterns = [
    # path('', home, name="home"),
    # path('', TemplateView.as_view(template_name= "fscohort/home.html"), name="home"),
    path('', HomeView.as_view(), name="home"),
    # path('student_list/', student_list, name="list"),
    path('student_list/', StudentListView.as_view(), name="list"),
    # path('student_add/', student_add, name="add"),
    path('student_add/', StudentCreateView.as_view(), name="add"),
    # path('detail/<int:id>/', student_detail, name="detail"),
    path('detail/<int:id>/', StudentDetailView.as_view(), name="detail"),
    # path('update/<int:id>/', student_update, name="update"), # commented
    path('update/<int:pk>/', StudentUpdateView.as_view(), name="update"), # new line
    path('delete/<int:id>/', student_delete, name="delete"),
]
```
