# Django Girls Tutorial

## Your first Django project!

### 创建项目

```bash
# Django installation
$ mkdir djangogirls
$ cd djangogirls
# 创建虚拟环境
$ python -m venv myvenv
# 激活虚拟环境
$ . myvenv/Scripts/activate
# 更新pip
$ python -m pip install -i https://mirrors.aliyun.com/pypi/simple/ --upgrade pip
# 安装django
$ pip install -i https://mirrors.aliyun.com/pypi/simple/ django
# 创建新项目
$ django-admin startproject mysite .
```

### Changing settings

```python
# mysite/settings.py

STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')

ALLOWED_HOSTS = ['127.0.0.1', '.pythonanywhere.com']
```

### Set up a database

```bash
# 默认使用sqlite3
$ python manage.py migrate
```

### Starting the web server

```bash
# 启动
$ python manage.py runserver
```

## Creating an blog application

创建app

```
$ python manage.py startapp blog
```

告诉Django使用app

```python
# mysite/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog.apps.BlogConfig',
]
```

### Django Models

```python
# blog/models.py
from django.conf import settings
from django.db import models
from django.utils import timezone


class Post(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(default=timezone.now)
    published_date = models.DateTimeField(blank=True, null=True)

    def publish(self):
        self.published_date = timezone.now()
        self.save()

    def __str__(self):
        return self.title
```

当创建或更新model时，都要执行以下两步命令

```
$ python manage.py makemigrations blog
$ python manage.py migrate blog
```

```python
# blog/admin.py
from django.contrib import admin
from .models import Post

admin.site.register(Post)
```

```bash
# 创建后台超级用户
$ python manage.py createsuperuser
# 启动后进入http://127.0.0.1:8000/admin/
$ python manage.py runserver
```

### Django URLs

```python
# mysite/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls')),
]
```

```python
# blog/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.post_list, name='post_list'),
]
```

### Django views

A view is a place where we put the "logic" of our application. It will request information from the `model` you created before and pass it to a `template`. 

```py
# blog/views.py
def post_list(request):
    return render(request, 'blog/post_list.html', {})
```

### Django templates

A template is a file that we can re-use to present different information in a consistent format

创建`blog/templates/blog/post_list.html`

```html
<!-- blog/templates/blog/post_list.html -->
<html>
    <head>
        <title>Django Girls blog</title>
    </head>
    <body>
        <div>
            <h1><a href="/">Django Girls Blog</a></h1>
        </div>

        <div>
            <p>published: 14.06.2014, 12:14</p>
            <h2><a href="">My first post</a></h2>
            <p>Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis vestibulum. Donec id elit non mi porta gravida at eget metus. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.</p>
        </div>

        <div>
            <p>published: 14.06.2014, 12:14</p>
            <h2><a href="">My second post</a></h2>
            <p>Aenean eu leo quam. Pellentesque ornare sem lacinia quam venenatis vestibulum. Donec id elit non mi porta gravida at eget metus. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut f.</p>
        </div>
    </body>
</html>
```

### Dynamic data in templates

This is exactly what views are supposed to do: connect models and templates.

```py
# blog/views.py
from django.shortcuts import render
from django.utils import timezone
from .models import Post

def post_list(request):
    posts = Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
    return render(request, 'blog/post_list.html', {'posts': posts})
```

### Django template tags

Django template tags allow us to transfer Python-like things into HTML, so you can build dynamic websites faster. 

```html
<!-- blog/templates/blog/post_list.html -->
<!-- body -->
<div>
    <h1><a href="/">Django Girls Blog</a></h1>
</div>

{% for post in posts %}
    <div>
        <p>published: {{ post.published_date }}</p>
        <h2><a href="">{{ post.title }}</a></h2>
        <p>{{ post.text|linebreaksbr }}</p>
    </div>
{% endfor %}
```

## Deploy

### Git

创建`.gitignore`

```
*.pyc
*~
__pycache__
myvenv
db.sqlite3
/static
.DS_Store
.vscode
```

```bash
$ git init
# Initialized empty Git repository in ~/djangogirls/.git/
$ git config --global user.name "Your Name"
$ git config --global user.email you@example.com
$ git status
$ git add --all .
$ git commit -m "My Django Girls app, first commit"
```

### Github

注册Github，并创建新仓库

```
$ git remote add origin https://github.com/Tyno945/my-first-blog.git
$ git push -u origin master
```

### PythonAnywhere

- 注册[PythonAnywhere](www.pythonanywhere.com)
- Creating a PythonAnywhere API token
- 打开PythonAnywhere "Bash" console

```bash
# PythonAnywhere command-line
$ pip3.7 install --user pythonanywhere
$ pa_autoconfigure_django.py --python=3.7 https://github.com/Tyno945/my-first-blog.git
$ python manage.py createsuperuser
# Pull your new code down to PythonAnywhere, and reload your web app
$ cd ~/<your-pythonanywhere-domain>.pythonanywhere.com
$ git pull
```

## Django ORM and QuerySets

```bash
# Django shell
$ python manage.py shell
# All objects
>>> from blog.models import Post
>>> Post.objects.all()
# Create object
>>> from django.contrib.auth.models import User
>>> User.objects.all()
>>> me = User.objects.get(username='felix')
>>> Post.objects.create(author=me, title='Sample title', text='Test')
# Filter objects
>>> Post.objects.filter(author=me)
>>> Post.objects.filter(title__contains='title')
>>> from django.utils import timezone
>>> Post.objects.filter(published_date__lte=timezone.now())
>>> post = Post.objects.get(title="Sample title")
>>> post.publish()
# Ordering objects
>>> Post.objects.order_by('created_date')
>>> Post.objects.order_by('-created_date')
# Complex queries through method-chaining
>>> Post.objects.filter(published_date__lte=timezone.now()).order_by('published_date')
# close the shell
>>> exit()
```

## CSS – make it pretty!

Cascading Style Sheets (CSS) is a language used for describing the look and formatting of a website written in a markup language (like HTML). Treat it as make-up for our web page.

创建CSS文件`blog/static/css/blog.css`

详细代码见`blog/templates/blog/post_list.html`和`blog/static/css/blog.css`

## Template extending

创建`blog/templates/blog/base.html`

```html
{% load static %}
<html>
    <head>
		<title>Django Girls blog</title>
		<link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css">
		<link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css">
		<link href="//fonts.googleapis.com/css?family=Lobster&subset=latin,latin-ext" rel="stylesheet" type="text/css">
		<link rel="stylesheet" href="{% static 'css/blog.css' %}">
    </head>
    <body>
		<div class="page-header">
			<h1><a href="/">Django Girls Blog</a></h1>
		</div>
		<div class="content container">
			<div class="row">
				<div class="col-md-8">
				{% block content %}
				{% endblock %}
				</div>
			</div>
		</div>
	</body>
</html>
```

```html
{% extends 'blog/base.html' %}

{% block content %}
    {% for post in posts %}
        <div class="post">
            <div class="date">
                {{ post.published_date }}
            </div>
            <h2><a href="">{{ post.title }}</a></h2>
            <p>{{ post.text|linebreaksbr }}</p>
        </div>
    {% endfor %}
{% endblock %}
```

## Create post's detail

- Create a template link to a post's detail
- Create a URL to a post's detail
- Add a post's detail view
- Create a template for the post details
