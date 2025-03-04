+++
title = "Building a SimpleTicket, an example CRUD API made with Django"
tags = ["coding", "python", "django"]
date = "2022-02-01"
draft = false
+++

# Introduction

In this tutorial, we will walk you through creating a simple ticket system using Django and Django REST Framework (DRF). This system will allow users to create, view, and manage tickets. We'll cover setting up the Django project, creating models, serializers, views, and URLs, as well as implementing user authentication using JSON Web Tokens (JWT).

## Project Structure

Here's the structure of the project:

```
simpleticket/
│
├── manage.py
├── simpleticket/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   ├── simpleticketapp/
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── migrations/
│   │   │   ├── 0001_initial.py
│   │   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── tests.py
│   │   ├── views.py
```

## Step 1: Setting Up the Django Project

First, let's set up the basic structure of the Django project. We'll initialize the project and create the main application.

### 1.1. Creating the Django Project

Open your terminal and run the following commands to create a new Django project:

```bash
django-admin startproject simpleticket
cd simpleticket
```

### 1.2. Creating the Django App

Next, create a new Django app within the project:

```bash
python manage.py startapp simpleticketapp
```

### 1.3. Installing Required Packages

Install the required packages by running:

```bash
pip install djangorestframework djangorestframework-simplejwt drf-spectacular psycopg2
```

## Step 2: Configuring the Django Project

Let's configure the Django project by setting up the database, installed apps, middleware, and other settings.

### 2.1. Settings (settings.py)

Update the `settings.py` file with the following configuration:

```python
from pathlib import Path
from datetime import timedelta

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = 'django-insecure-...'

DEBUG = True

ALLOWED_HOSTS = ['*']

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'simpleticket.simpleticketapp',
    'corsheaders',
    'rest_framework_simplejwt',
    'drf_spectacular',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'corsheaders.middleware.CorsMiddleware',
]

ROOT_URLCONF = 'simpleticket.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'simpleticket.wsgi.application'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'simpleticket',
        'USER': 'postgres',
        'PASSWORD': 'postgres',
        'HOST': 'db',
        'PORT': '5432',
    }
}

AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'UTC'

USE_I18N = True

USE_TZ = False

STATIC_URL = 'static/'

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

CORS_ORIGIN_ALLOW_ALL = True
ALLOWED_HOSTS = ['*']

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=15),
}

SPECTACULAR_SETTINGS = {
    'TITLE': 'simpleticket API',
    'DESCRIPTION': 'simple trouble ticketing system',
    'VERSION': '1.0.0',
    'SERVE_INCLUDE_SCHEMA': False,
}
```

## Step 3: Creating the Ticket Model

Let's create the `Ticket` model to represent a ticket in the system.

### 3.1. Models (models.py)

Update the `models.py` file with the following code:

```python
from django.db import models
from django.contrib.auth.models import User

class Ticket(models.Model):
    title = models.CharField(max_length=150)
    description = models.TextField(max_length=1000)
    creationDate = models.DateTimeField()
    dueDate = models.DateTimeField()
    createdBy = models.ForeignKey(User, related_name="createdBys", related_query_name="createdBy", on_delete=models.CASCADE)
    requestedBy = models.ForeignKey(User, related_name="requestedBys", related_query_name="requestedBy", on_delete=models.CASCADE)
    requestedFor = models.ForeignKey(User, related_name="requestedFors", related_query_name="requestedFor", on_delete=models.CASCADE)
    completed = models.BooleanField()
```

### 3.2. Running Migrations

Run the following commands to create and apply the migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

## Step 4: Creating Serializers

Serializers are used to convert complex data types, such as querysets and model instances, to native Python data types that can then be easily rendered into JSON, XML, or other content types.

### 4.1. Serializers (serializers.py)

Update the `serializers.py` file with the following code:

```python
from django.contrib.auth.models import User, Group
from simpleticket.simpleticketapp.models import Ticket
from rest_framework import serializers

class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ['url', 'id', 'username', 'email', 'groups']

class GroupSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Group
        fields = ['url', 'id', 'name']

class TicketSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Ticket
        fields = ['url', 'id', 'title', 'description', 'creationDate', 'dueDate', 'createdBy', 'requestedBy', 'requestedFor', 'completed']

class TokenObtainPairResponseSerializer(serializers.Serializer):
    access = serializers.CharField()
    refresh = serializers.CharField()

    def create(self, validated_data):
        raise NotImplementedError()

    def update(self, instance, validated_data):
        raise NotImplementedError()

class TokenRefreshResponseSerializer(serializers.Serializer):
    access = serializers.CharField()

    def create(self, validated_data):
        raise NotImplementedError()

    def update(self, instance, validated_data):
        raise NotImplementedError()

class RegisterSerializer(serializers.ModelSerializer):
    email = serializers.EmailField(
        required=True,
        validators=[UniqueValidator(queryset=User.objects.all())]
    )
    password = serializers.CharField(write_only=True, required=True)
    password2 = serializers.CharField(write_only=True, required=True)

    class Meta:
        model = User
        fields = ('username', 'password', 'password2', 'email')

    def validate(self, attrs):
        if attrs['password'] != attrs['password2']:
            raise serializers.ValidationError({"password": "Password fields didn't match."})
        return attrs

    def create(self, validated_data):
        user = User.objects.create(
            username=validated_data['username'],
            email=validated_data['email'],
        )
        user.set_password(validated_data['password'])
        user.save()
        return user
```

## Step 5: Creating Views

Views are used to handle the logic for processing requests and returning responses.

### 5.1. Views (views.py)

Update the `views.py` file with the following code:

```python
from django.contrib.auth.models import User, Group
from simpleticket.simpleticketapp.models import Ticket
from rest_framework import viewsets
from rest_framework import permissions
from simpleticket.simpleticketapp.serializers import UserSerializer, GroupSerializer, TicketSerializer, TokenObtainPairResponseSerializer, TokenRefreshResponseSerializer
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView
from rest_framework.permissions import AllowAny
from rest_framework.views import APIView
from rest_framework.response import Response
from .serializers import UserSerializer, RegisterSerializer
from django.contrib.auth.models import User
from rest_framework import generics

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated]

class GroupViewSet(viewsets.ModelViewSet):
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
    permission_classes = [permissions.IsAuthenticated]

class TicketViewSet(viewsets.ModelViewSet):
    queryset = Ticket.objects.all()
    serializer_class = TicketSerializer
    permission_classes = [permissions.IsAuthenticated]

class RegisterUserAPIView(generics.CreateAPIView):
    permission_classes = []
    serializer_class = RegisterSerializer
```

## Step 6: Setting Up URLs

Let's set up the URLs for the API endpoints.

### 6.1. URLs (urls.py)

Update the `urls.py` file with the following code:

```python
from django.contrib import admin
from django.urls import include, path
from rest_framework import routers
from simpleticket.simpleticketapp import views
from rest_framework import permissions
from drf_spectacular.views import SpectacularAPIView, SpectacularRedocView, SpectacularSwaggerView
from django.contrib.staticfiles.urls import staticfiles_urlpatterns

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)
router.register(r'groups', views.GroupViewSet)
router.register(r'tickets', views.TicketViewSet)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include(router.urls)),
    path('api-auth/', include('rest_framework.urls', namespace='rest_framework')),
    path('api/token/', views.TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', views.TokenRefreshView.as_view(), name='token_refresh'),
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    path('api/schema/swagger-ui/', SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'),
    path('api/schema/redoc/', SpectacularRedocView.as_view(url_name='schema'), name='redoc'),
    path('register', views.RegisterUserAPIView.as_view()),
]

urlpatterns += staticfiles_urlpatterns()
```

## Step 7: Running the Server

Finally, run the development server to test the API:

```bash
python manage.py runserver
```

## Conclusion

You've now created a simple ticket system using Django and Django REST Framework. This system allows users to create, view, and manage tickets. You can further enhance the system by adding more features or improving the existing ones.

### Resources

- Check out the code for this tutorial [on my GitHub](https://github.com/raphael2692/simpleticket)