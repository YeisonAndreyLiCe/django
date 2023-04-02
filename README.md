# Django
Django divides the project into apps. Each app is a self-contained unit of functionality. An app can be reused in different projects. 
Note: Django comes with a few built-in apps, such as admin and auth.
- admin: The admin app allows you to manage content on your site. You can add, change, and delete models.
- auth: The auth app provides a framework for authenticating users. It includes models for users, groups, permissions, and sessions.
- contenttypes: The contenttypes app allows you to assign permissions to models.
- sessions: The sessions app provides a framework for managing user sessions.
- messages: The messages app provides a framework for managing user messages.
- staticfiles: The staticfiles app allows you to collect static files in a single location and serve them via a URL.
- sites: The sites app allows you to manage multiple sites with a single Django installation.

## MVT (Model-View-Template)
Django is an MVT framework. 

### Model
The model is the single, definitive source of information about your data. It contains the essential fields and behaviors of the data you’re storing. Generally, each **model maps to a single database table**.
```python
# app/models.py
from django.db import models

class Question(models.Model):
    id = models.AutoField(primary_key=True)
    info = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text
```
### View
The view is responsible for displaying the data (as HTML) and responding to user input. **It contains the business logic of your application**.
```python
# app/views.py
from django.shortcuts import render

from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'forum/templates/index.html', context)
```
### Template
The template contains the HTML for the view. **It displays the data passed to it by the view**.
```html
{% extends "templates/base.html" %}

{% block content %}
    <h1>Latest Questions</h1>
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/forum/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% endblock %}
```

## Django Directory Structure
The structure I show you is the one I use in my projects. It allows me to reuse easily apps in other projects, modularize the templates and use the same template names in different apps.
```bash
├───{project_name}
│   ├───{project_name}
│   │   ├───__init__.py
│   │   ├───asgi.py
│   │   ├───settings.py
│   │   ├───urls.py
│   │   └───wsgi.py
│   ├───manage.py
│   ├───{app_name}
│   │   ├───__init__.py
│   │   ├───admin.py
│   │   ├───apps.py
│   │   ├───migrations
│   │   │   └───__init__.py
│   │   ├───models.py
│   │   ├───templates
│   │   │   └───{app_name}
│   │   │       ├───{template_name}.html
│   │   │        └───blocks
│   │   ├───static
│   │   │   └───{app_name}
│   │   │       └───{static_file}
│   │   ├───tests.py
│   │   ├───urls.py
│   │   └───views.py
│   ├───static
│   │   ├───css
│   │   └─── js
│   ├───templates
│   │   ├───base.html
│   │   └───blocks
│   ├───media
│   └───requirements.txt
├───.git
├───.gitignore
└───venv

```
- `asgi.py`: An entry-point for ASGI-compatible web servers to serve your project. That means it’s the interface between Django and your web server, and it supports asynchronous work.
- `wsgi.py`: An entry-point for WSGI-compatible web servers to serve your project. That means it’s the interface between Django and your web server, and it supports synchronous work.
- `manage.py`: A command-line utility that lets you interact with this Django project in various ways. You can read all the details about manage.py in django-admin and manage.py.
- `settings.py`: Settings/configuration for this Django project. Django settings will tell you all about how settings work.
- urls.py: The URL declarations for this Django project; a “table of contents” of your Django-powered site.
- `venv`: A directory that contains a Python virtual environment.


## Start a new project
*Remember to activate the virtual environment* before starting a new project. to learn more about virtual environments click [here](https://github.com/YeisonAndreyLiCe/virtual_environments)
```bash
(venv)$ django-admin startproject {project_name}
```
## Start a new app
```bash
(venv)$ python manage.py startapp {app_name}
```

## Add the app to the project
Add the app to the `INSTALLED_APPS` list in the `settings.py` file.
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    '{app_name}',
]
```
## Routing
As a good practice, you should create a **urls.py file in each app** and then include the app’s urls in the project’s urls.py file. This makes it easier to include the app in other projects. 

- Create a `urls.py` file in the app directory.
```bash
$ touch app/urls.py 
```

- To create a route you must add a function to the `views.py` file.
```python
# app/views.py
from django.shortcuts import render

def index(request):
    return render(request, 'forum/templates/forum/index.html')
```

- Then you must add the route to the `urls.py` file.
```python
# app/urls.py
from django.urls import path

from . import views

app_name = 'forum' # here we add the app's name to the namespace. This is used to avoid conflicts with other apps when using named routes.
urlpatterns = [
    path('', views.index, name='index'), # Note that the name of the route is index
]
```

- then you must add the app to the `urls.py` file in the project directory.
```python
# project/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('forum/', include('forum.urls')), # here we add the app's urls
    path('admin/', admin.site.urls),
]
```

#### Routing with parameters
```python
# app/urls.py
from django.urls import path

from . import views

app_name = 'forum'
urlpatterns = [
    path('questions/<int:question_id>/', views.detail, name='detail'),
]
```

```python
# app/views.py
from django.shortcuts import render

def detail(request, question_id):
    return render(request, 'forum/templates/forum/detail.html', {'question_id': question_id})
```

```html
<!-- app/templates/detail.html -->
<h1>Question {{ question_id }}</h1>
```

---

## Run the server
To run the server you must be in the same directory as the manage.py file.
```bash
(venv)$ python manage.py runserver
```

## Create a superuser
Django comes with a ui to manage the database. To access it you must create a superuser.
```bash
(venv)$ python manage.py createsuperuser
```

## Templates
We can create a template folder at the same level as the manage.py file. This way we can use the same template for all the apps in the project. There we can create a base template that will be used by all the other templates.

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Forum</title>
</head>
<body>
    <div class="container">
        {% block content %}
        {% endblock %}
    </div>
</body>
</html>
```
Note that the `base.html` file is the base template for all the other templates in the app. We are using jinja2 syntax to include the content of the other templates. Something to remark whe using django to access the properties of an object, you must use the dot notation.  
As a good practice, you should **create a templates folder in each app**. To avoid conflicts with other apps it is suggested to create a folder with the name of the app inside the templates folder and add the templates there.

## Rendering and passing data to templates
```python
# app/views.py
from django.shortcuts import render

from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5] # get the last 5 questions from the database
    context = {'latest_question_list': latest_question_list} # create a context dictionary, with the data that will be passed to the template
    return render(request, 'forum/templates/forum/index.html', context)
```

```html
<!-- forum/templates/forum/index.html -->
{% extends "templates/base.html" %}

{% block content %}
    <h1>Latest Questions</h1>
    <ul>
    {% for question in latest_question_list %}
        <li><a href="{% url 'forum:detail' id=question.id %}">{{ question.info }}</a></li>
    {% endfor %}
    </ul>
{% endblock %}
```
Note that we are using the `url` tag to create a link to the detail view of each question. The `url` tag takes the name of the route as the first argument and the parameters as the second argument.  
the format is as follow: `{% url 'app_name:view_name' parameter_name=parameter_value %}`. That is why we named the route when we created it. To be more specific we also named the app in the `urls.py` file. That allows as to use the same name for the route in different apps.


## Static files
- Create a `static` folder at the same level as the `manage.py` file.
- Create a `css` folder in the `static` folder. There we will add styles common to most of the apps.
```css
/* static/css/style.css */
body {
    background-color: #f5f5f5;
}
```
```javascript
// static/js/script.js
console.log('Hello World');
```

- Add the static files to the `settings.py` file.
```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]
```
- Add the static files to the `*.html` file.
```html
<!-- app/templates/app/*.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Forum</title>
    <link rel="stylesheet" href="{% static 'css/style.css' %}">
</head>
<body>
    <div class="container">
        {% block content %}
            <img src="{% static 'img/logo.png' %}" alt="logo" />
        {% endblock %}
    </div>
    <script src="{% static 'js/script.js' %}"></script>
</body>
</html>
```

## POST and GET requests
### POST
```python
# app/views.py
from django.shortcuts import render, redirect

from .models import Question

def index(request):
    if request.method == 'POST':
        question = Question(info=request.POST.get('info'), pub_date=timezone.now())
        question.save()
        return redirect('forum:index')
    else:
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        context = {'latest_question_list': latest_question_list}
        return render(request, 'forum/templates/index.html', context)
```
Note that we redirect the user to the index view after the form is submitted. This's required to avoid the user to resubmit the form if they refresh the page.

```html
<!-- app/templates/index.html -->
{% extends "templates/base.html" %}

{% block content %}
    <h1>Latest Questions</h1>
    <ul>
    {% for question in latest_question_list %}
        <li><a href="{% url 'forum:questions' id=question.id %}">{{ question.info }}</a></li>
    {% endfor %}
    </ul>
    <div class="col-12">
            {% include 'forum/sections/form_new_project.html' %}
        </div>
    <form action="{% url 'forum:new_question' %}" id="form" method="post">
        {% csrf_token %}
        <input type="text" name="info" placeholder="Question">
        <input type="submit" value="Add">
    </form>
{% endblock %}
```
Note the use of the `csrf_token` tag. This is a security measure to prevent **cross-site request forgery**. It is required for any POST request in django. 

### GET
```python
# app/views.py
from django.shortcuts import render, redirect

from .models import Question

def index(request):
    if request.method == 'POST':
        question = Question(info=request.POST.get('info'), pub_date=timezone.now())
        question.save()
        return redirect('forum:index')
    else:
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        context = {'latest_question_list': latest_question_list}
        return render(request, 'forum/templates/index.html', context)
```

```html

<!-- app/templates/index.html -->
{% extends "templates/base.html" %}

{% block content %}
    <h1>Latest Questions</h1>
    <ul>
    {% for question in latest_question_list %}
        <li><a href="{% url 'forum:questions' id=question.id %}">{{ question.info }}</a></li>
    {% endfor %}
    </ul>
    <div class="col-12">
            {% include 'forum/sections/form_new_project.html' %}
        </div>
    <form action="{% url 'forum:new_question' %}" id="form" method="get">
        <input type="text" name="info" placeholder="Question">
        <input type="submit" value="Add">
    </form>
{% endblock %}
```

## Django Admin
### Creating a superuser
```bash
$ python manage.py createsuperuser
```
### Registering models
```python
# app/admin.py
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```
### Customizing the admin interface
```python
# app/admin.py
from django.contrib import admin

from .models import Question

class QuestionAdmin(admin.ModelAdmin):
    list_display = ('info', 'pub_date')

admin.site.register(Question, QuestionAdmin)
```

## Sessions
The request response cycle is stateless. This means that the server does not remember the state of the client between requests. This is a problem when you want to store data that is specific to a user. For example, if you want to store the number of times a user has visited your website.
A session is a way to store data on the server side. It is used to store data that is specific to a user.

> "In a given process (HTTP request / response), we can manage data (search terms and search results, for example) that survive the process that generated it. This data is called state. State allows our site to "know" a lot of useful information". [Coding Dojo](https://www.codingdojo.com)   

For example, if we are logged in, we can display our name and a logout link. If we are not logged in, we can display a login link. We can also use state to store information about the user's shopping cart. We can store the items in the cart in the session, and then retrieve them when the user checks out.

### About cookies
A cookie is a small piece of data that is stored on the user's computer by the web browser while browsing a website. Cookies are created when the user visits a website that uses cookies to keep track of the user's browsing session. Each time the user loads a page on the website, the browser sends the cookie back to the server to notify the website of the user's previous activity. Cookies are primarily used for authentication, tracking and personalization. Django uses the hash-based message authentication code (HMAC) to sign cookies. This means that the user can't modify the contents of the cookie. The cookie is signed using a secret key that is stored in the `settings.py` file. The secret key is used to sign the cookie. If the user tries to modify the cookie, the signature will be invalid and the cookie will be rejected. cookies to store session data.

Since django uses the database to administer sessions, we need to run the following command:
```bash
$ python manage.py migrate
```
### Creating a session
```python
# app/views.py
from django.shortcuts import render, redirect

def some_function(request):
    request.session['name'] = request.POST['name']
    request.session['counter'] = 100
```
Note that django uses a dictionary to store the session data. The key is the name of the session variable, and the value is the value of the session variable.

```python
# app/views.py
def some_function(request):
    request.session['my_list'] = []
    request.session['my_list'].append("new item")
    request.session.save() # save the session data.
```

### Retrieving a session
```python
# app/views.py
def some_function(request):
    name = request.session['name']
    counter = request.session['counter']
```
### Deleting a session
```python
# app/views.py
def some_function(request):
    del request.session['name']
    del request.session['counter']
```
### Clearing all sessions
```python
# app/views.py
def some_function(request):
    request.session.clear()
```
### Checking if a session exists
```python
# app/views.py
def some_function(request):
    if 'name' in request.session: # like a dictionary
        name = request.session['name']
    else:
        name = 'No name'
```
### Setting a session expiration
```python
# app/views.py
def some_function(request):
    request.session.set_expiry(60)
```
### Setting a session expiration date
```python
# app/views.py
def some_function(request):
    request.session.set_expiry(datetime.datetime(2019, 1, 1, 12, 0, 0))
```
## Django and PostgreSQL
### Setting up PostgreSQL
    $ sudo apt-get install postgresql postgresql-contrib

### Creating a database
    $ sudo -u postgres psql
    postgres=# CREATE DATABASE project_database;

### Creating a user
    postgres=# CREATE USER project_user WITH PASSWORD 'password';

### Granting privileges
    postgres=# GRANT ALL PRIVILEGES ON DATABASE project_database TO project_user;

### Connecting to the database
    $ sql -h localhost -U project_user project_database

### Configuring Django to use PostgreSQL
```python
# settings.py

DATABASES =  {
    'default': {
        'ENGINE': 'django.db.backends.postgresql', # set postgres as the database engine
        'NAME': 'project_database', # database name
        'USER': 'username', 
        'PASSWORD': 'password',
        'HOST': '127.0.0.1', # IP Address localhost
        'PORT': '5432', # default port. It could be different. to check run: sudo netstat -tulpn | grep :5432
    }
} 
```
## ORM (Object Relational Mapping)
ORM is a technique that lets you query and manipulate data from a database using an object-oriented paradigm. In other words, it allows you to write queries and manipulate data using Python objects and classes. ORM is able to translate a row of a database into a class instance, being each column a class attribute, the same apply for the other way around. If you are familiar with java it is similar to the JDBC API.

## Models
```python
# app/models.py
from django.db import models

# inherit from models.Model allows to use the ORM
class Question(models.Model):
    info = models.CharField(max_length=200)
    pub_date = models.DateTimeField(auto_now_add=True, tex_help='date published')
    update_at = models.DateTimeField(auto_now=True, tex_help='date updated')

    def __str__(self):
        return self.info
```
Notes: id is automatically created by django. It is the primary key of the table. It is an integer field that is automatically incremented. __init__ is not required. Django will create it for you. We can override the __str__ method to return a string representation of the object. The __str__ method is used to display the object in the admin interface.

### "The forward engineering of Django"
    python3 manage.py makemigrations
    python3 manage.py migrate

`migrations.py` is a kind of traceability of changes in the database. Each time we make a change in the models, we need to run the `makemigrations` command. This command will create a new file in the `migrations` folder. This file contains the changes that we need to make to the database. The `migrate` command will apply the changes to the database.

### Django Shell
    python3 manage.py shell

Allows to interact with the models. It is useful to test the models and the ORM. From the shell we can create, retrieve, update and delete objects from the database.
    
    import app.models

## CRUD (Create, Read, Update, Delete)
### Create
```python
# app/views.py
from app.models import Question

def some_function(request):
    question = Question(info="Why?")
    question.save()
```
Alternatively, we can use the `create` method:
```python
# app/views.py
from app.models import Question

def some_function(request):
    # ClassName.objects.create(field1="value", field2="value", etc.)
    question = Question.objects.create(info="Why?")
```
The difference between `save` and `create` is that `save` creates an instance of the model and then saves it to the database. `create` creates an instance of the model and saves it to the database in a single step.

### Read
- `ClassName.objects.all()` Retrieve all objects
- `ClassName.objects.get(id=1)` Retrieve a single object
- `ClassName.objects.filter(field="value", ...)` Retrieve all objects that match the filter
- `ClassName.objects.exclude(field="value", ... )` Retrieve all objects that do not match the filter
- `ClassName.objects.order_by("field")` Retrieve all objects ordered by the field
- `ClassName.objects.order_by("-field")` Retrieve all objects ordered by the field in descending order
- `ClassName.objects.count()` Retrieve the number of objects
- `ClassName.objects.first()` Retrieve the first object
- `ClassName.objects.last()` Retrieve the last object

### Update
```python
# app/views.py
from app.models import Question

def some_function(request):
    question = Question.objects.get(id=1)
    question.info = "Why not?"
    question.save()
```
### Delete
```python
# app/views.py
from app.models import Question

def some_function(request):
    question = Question.objects.get(id=1)
    question.delete()
```

## One to Many Relationship
```python
# app/models.py
from django.db import models


class Question(models.Model):
    info = models.CharField(max_length=200)
    pub_date = models.DateTimeField(auto_now_add=True, tex_help='date published')
    update_at = models.DateTimeField(auto_now=True, tex_help='date updated')

    def __str__(self):
        return self.info
    
class Answer(models.Model):
    info = models.CharField(max_length=200)
    question = models.ForeignKey(Question, related_name="answers" ,on_delete=models.CASCADE) # one to many relationship
    pub_date = models.DateTimeField(auto_now_add=True, tex_help='date published')
    update_at = models.DateTimeField(auto_now=True, tex_help='date updated')

    def __str__(self):
        return self.info
```
- to establish the relationship in views.py we need to import the models
```python
question = Question.objects.get(id=1)
answer = Answer.objects.create(info="Because", question=question)
```
- to retrieve the answers of a question
```python
question = Question.objects.get(id=1)
answers = question.answer_set.all()
```
- to retrieve the question of an answer
```python
answer = Answer.objects.get(id=1)
question = answer.question
```

### Inverse query with related_name
```python
# app/models.py
from django.db import models


class Question(models.Model):
    info = models.TextField()
    pub_date = models.DateTimeField(auto_now_add=True, tex_help='date published')
    update_at = models.DateTimeField(auto_now=True, tex_help='date updated')
    # answers = a list of answers associated with the question.

    def __str__(self):
        return self.info

class Answer(models.Model):
    info = models.TextField()
    question = models.ForeignKey(Question, related_name="answers" ,on_delete=models.CASCADE) # one to many relationship
    pub_date = models.DateTimeField(auto_now_add=True, tex_help='date published')
    update_at = models.DateTimeField(auto_now=True, tex_help='date updated')

    def __str__(self):
        return self.info
```
Having the `related_name` attribute in the `ForeignKey` field, we can use the `answers` attribute to retrieve the answers of a question. This is called an inverse query.

```python
# app/views.py
from app.models import Question, Answer
from django.shortcuts import render

def some_function(request):
    question = Question.objects.get(id=1)
    answers = question.answers.all() # inverse query
```

```html
<h1>Questions List</h1>
<ul>
  {% for question in questions %} # assuming that we have a list of questions
    <li>{{question.info}}
      <ul>
    	<!-- loop over the answers! -->
        {% for answer in question.answers.all %}	
          <li><em>{{answer.info}}</em></li>
        {% endfor %}
      </ul>
    </li>
  {% endfor %}
</ul>
```

## Many to Many Relationship
```python
# app/models.py

from django.db import models

class Book(models.Model):
	title = models.CharField(max_length=255)
	created_at = models.DateTimeField(auto_now_add=True)
	updated_at = models.DateTimeField(auto_now=True)


class Publisher(models.Model):
	name = models.CharField(max_length=255)
	books = models.ManyToManyField(Book, related_name="publishers") # it does not matter which model has the many to many relationship
	created_at = models.DateTimeField(auto_now_add=True)
	updated_at = models.DateTimeField(auto_now=True)

```

### Add a publisher to a book
```python
# app/views.py
from app.models import Book, Publisher

def some_function(request):
    book = Book.objects.get(id=1)
    publisher = Publisher.objects.get(id=1)
    book.publishers.add(publisher)
```
While adding a publisher to a book, django will automatically add the book to the publisher's list of books.

### Remove a publisher from a book
```python
# app/views.py

def some_function(request):
    book = Book.objects.get(id=1)
    publisher = Publisher.objects.get(id=1)
    book.publishers.remove(publisher)
```
We can use reverse query to retrieve the books of a publisher.
```python
# app/views.py
from app.models import Book, Publisher

def some_function(request):
    publisher = Publisher.objects.get(id=1)
    books = publisher.books.all()
```
### Advance Querying
```python
# app/views.py

def some_function(request):
    books = Book.objects.filter(publishers__name="publisher name")
    books = Book.objects.filter(publishers__name__contains="Al")
```

## RESTful Routing
- `GET /books` Retrieve all books
- `GET /books/1` Retrieve a book with id 1
- `POST /books` Create a new book
- `PUT /books/1` Update a book with id 1
- `DELETE /books/1` Delete a book with id 1

Note that the routes for `GET` and `POST` are the same. The difference is in the `method` attribute of the form. The `method` attribute can be `GET` or `POST`.

Note that the `PUT` and `DELETE` methods are not supported by HTML forms. We need to use a library like `axios` to make requests to the server. 

## Validating Inputs before Saving
```python
# app/models.py
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=255)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title

    def save(self, *args, **kwargs):
        if len(self.title) < 5:
            raise Exception("Title must be at least 5 characters long")
        super(Book, self).save(*args, **kwargs)
```
```python
# app/views.py

from app.models import Book

def some_function(request):
    book = Book.objects.get(id=1)
    book.title = "a"
    try:
        book.save()
    except Exception as e:
        print(e)
```
Save method is called when we create a new book or update an existing book. We can use this method to validate the inputs before saving the book. An alternative is to customize the behavior of the manager.

```python
# app/models.py

from django.db import models

class BookManager(models.Manager):
    def create_book(self, title):
        if len(title) < 5:
            raise Exception("Title must be at least 5 characters long")
        book = self.create(title=title)
        return book

class Book(models.Model):
    title = models.CharField(max_length=255)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    objects = BookManager()

    def __str__(self):
        return self.title
```
```python
# app/views.py

from app.models import Book

def some_function(request):
    try:
        book = Book.objects.create_book("a")
    except Exception as e:
        pass
```
We can use the `create_book` method to create a new book. The `create_book` method will validate the inputs before creating the book.

## Securing Passwords
### Encrypting vs Hashing
- **Encryption** is the process of converting plain text into a cipher text. The cipher text is not human readable. The cipher text **can be decrypted back to the plain text**. This kind of technique is used to secure sensitive information like credit card numbers, social security numbers, etc.

- **Hashing** is the process of converting plain text into a cipher text. The cipher text is not human readable. The cipher text **cannot be decrypted back to the plain text**. This kind of technique is used to secure passwords.


### Django's Password Hashing
more info: https://docs.djangoproject.com/en/4.1/topics/auth/passwords/