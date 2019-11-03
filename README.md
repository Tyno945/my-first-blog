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
```