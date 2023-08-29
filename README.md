# Steps/Notes for Django Tutorial

## Part 1 - Initial set up and include method in urls.py

1. In desired folder, run `django-admin startproject <project_name>`
2. cd into project, then `python manage.py runserver`
3. To create and application, run `python manage.py startapp <app_name>`
4. In <app_name>/views.py, create a function that returns an HttpResponse
```python
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
5. In <app_name>/views.py, add the following: 
```python
from django.urls import path

from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```
6. In <project_name>/urls.py, add the following:
```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
]
```
About the include above from tutorial source.\
The include() function allows referencing other URLconfs. Whenever Django encounters include(), it chops off whatever part of the URL matched up to that point and sends the remaining string to the included URLconf for further processing.

The idea behind include() is to make it easy to plug-and-play URLs. Since polls are in their own URLconf (polls/urls.py), they can be placed under “/polls/”, or under “/fun_polls/”, or under “/content/polls/”, or any other path root, and the app will still work.

>When to use include():
>* You should always use include() when you include other URL patterns. admin.site.urls is the only exception to this.

The path() function is passed four arguments, two required: route and view, and two optional: kwargs, and name. At this point, it’s worth reviewing what these arguments are for.

path() argument: route¶
route is a string that contains a URL pattern. When processing a request, Django starts at the first pattern in urlpatterns and makes its way down the list, comparing the requested URL against each pattern until it finds one that matches.

Patterns don’t search GET and POST parameters, or the domain name. For example, in a request to https://www.example.com/myapp/, the URLconf will look for myapp/. In a request to https://www.example.com/myapp/?page=3, the URLconf will also look for myapp/.

path() argument: view¶
When Django finds a matching pattern, it calls the specified view function with an HttpRequest object as the first argument and any “captured” values from the route as keyword arguments. We’ll give an example of this in a bit.

path() argument: kwargs¶
Arbitrary keyword arguments can be passed in a dictionary to the target view. We aren’t going to use this feature of Django in the tutorial.

path() argument: name¶
Naming your URL lets you refer to it unambiguously from elsewhere in Django, especially from within templates. This powerful feature allows you to make global changes to the URL patterns of your project while only touching a single file.

When you’re comfortable with the basic request and response flow, read part 2 of this tutorial to start working with the database.

## Part 2.1 - Models

**Note: Django comes with SQLite, so it is recommended to switch to PostgreSQL for production.**
Text below is from to tutorial source.
'''
If you wish to use another database, install the appropriate database bindings and change the following keys in the DATABASES 'default' item to match your database connection settings:

* ENGINE – Either 'django.db.backends.sqlite3', 'django.db.backends.postgresql', 'django.db.backends.mysql', or 'django.db.backends.oracle'. Other backends are also available.
* NAME – The name of your database. If you’re using SQLite, the database will be a file on your computer; in that case, NAME should be the full absolute path, including filename, of that file. The default value, BASE_DIR / 'db.sqlite3', will store the file in your project directory.

[Info on databases](https://docs.djangoproject.com/en/4.2/ref/settings/#std-setting-DATABASES)

1. Run the command `python manage.py migrate` to create the models defined in <app_name>/models.py.
2. In <app_name>/models.py, insert representation of your models. For example:
```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```
3. In <project_name>/settings.py, add a reference to the app in the INSTALLED_APPS list - `'<app_name>.apps.<App_name>Config'`. For example:
```python
INSTALLED_APPS = [
    "polls.apps.PollsConfig", # Object <App_name>Config was created when you ran `python manage.py startapp <app_name>`.
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
```
4. Next run `python manage.py makemigrations polls`. This will 
Migrations for 'polls':
```
  polls/migrations/0001_initial.py
    - Create model Question
    - Create model Choice
```
5. Run migrate again to create those model tables in your database `$ python manage.py migrate`

Quick Summary
1. Change your models (in models.py).
2. Run `python manage.py makemigrations` to create migrations for those changes
3. Run `python manage.py migrate` to apply those changes to the database.

## Part 2.2 - Playing with the API

1. Run `python manage.py shell` to open the Django shell. This is for testing purposes according to AI.
2. Update <app_name>/models.py to include a __str__() method for each model. For this demo, also add was_published_recently. For example:
```python
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)


class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

## Part 2.3 - Create an admin user and register models

1. Run `python manage.py createsuperuser` to create an admin user.
2. Enter desired username, email(optional), and password.
3. Run `python manage.py runserver` to start the server.
4. Navigate to e.g., http://127.0.0.1:8000/*admin/*
5. Login with the admin user you created.
6. Tell the admin that Questions has admin interface. In <app_name>/admin.py, add, for example, the following:
```python 
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```
Cool
7. Now you can add questions and choices in the admin interface.

## Part 3.1 - Views

In Django, web pages and other content are delivered by views. Each view is represented by a Python function (or method, in the case of class-based views). Django will choose a view by examining the URL that’s requested (to be precise, the part of the URL after the domain name).

(DISPATCHER REFERENCES)[https://docs.djangoproject.com/en/4.2/topics/http/urls/]

1. Use <app_name>/views.py to add functions that endpoints from <app_name>/polls.py will call. Fox example:
```python
<app_name>/views.py
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)


def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)


def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```
2. In <app_name>/urls.py, add the following:
```python
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path("", views.index, name="index"),
    # ex: /polls/5/
    path("<int:question_id>/", views.detail, name="detail"),
    # ex: /polls/5/results/
    path("<int:question_id>/results/", views.results, name="results"),
    # ex: /polls/5/vote/
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```
 When somebody requests a page from your website – say, “/polls/34/”, Django will load the <project_name>.urls Python module because it’s pointed to by the ROOT_URLCONF setting. It finds the variable named urlpatterns and traverses the patterns in order. After finding the match at 'polls/', it strips off the matching text ("polls/") and sends the remaining text – "34/" – to the ‘polls.urls’ URLconf for further processing. 
> View definitions can do anything, CRUD databases, product files, etc. **All Django wants is that HttpResponse. Or an exception such as Http404.**

## Part 3.2 - Templates

A Template is a text file. It can generate any text-based format (HTML, XML, CSV, etc.). A template contains variables, which get replaced with values when the template is evaluated, and tags, which control the logic of the template.

1. Create a templates directory in <app_name>/templates/<app_name>/ and add a template file. For example:
```html
 {% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```
2. In <app_name>/views.py, update the index function to use the template. For example:
```python
from django.shortcuts import render


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {
        "latest_question_list": latest_question_list,
    }
    return render(request, "polls/index.html", context)
```

## Part 3.3 - Raising a 404 error

1. In <app_name>/views.py, update the detail function to raise a 404 error if the question does not exist. For example:
```python
from django.http import Http404
from django.shortcuts import render

from .models import Question

def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})
```
Django also has a shortcut for this, `get_object_or_404()`. See below.
```python
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
```
