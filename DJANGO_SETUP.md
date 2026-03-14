# Django Complete Setup Guide

## Overview
Django is a high-level Python web framework that encourages rapid development and clean, pragmatic design. It's batteries-included with ORM, admin panel, and authentication built-in.

---

## Folder Structure for Django

```
django-app/
├── README.md
├── manage.py
├── requirements.txt
├── .env.example
├── .gitignore
├── pytest.ini
├── Dockerfile
├── docker-compose.yml
│
├── config/                    # Project settings folder
│   ├── __init__.py
│   ├── settings.py           # Main settings (or points to settings/)
│   ├── settings/
│   │   ├── base.py           # Base settings
│   │   ├── development.py    # Dev settings
│   │   ├── production.py     # Prod settings
│   │   └── testing.py        # Test settings
│   ├── urls.py               # Root URL configuration
│   ├── asgi.py               # ASGI entry point
│   └── wsgi.py               # WSGI entry point
│
├── apps/                      # Django applications
│   ├── __init__.py
│   │
│   ├── items/
│   │   ├── migrations/
│   │   │   └── __init__.py
│   │   ├── __init__.py
│   │   ├── admin.py          # Admin configuration
│   │   ├── apps.py           # App configuration
│   │   ├── models.py         # Database models
│   │   ├── views.py          # API views
│   │   ├── serializers.py    # DRF serializers
│   │   ├── urls.py           # App URL routes
│   │   ├── filters.py        # Query filters
│   │   ├── permissions.py    # Custom permissions
│   │   ├── pagination.py     # Pagination classes
│   │   └── tests.py          # App tests
│   │
│   ├── users/
│   │   ├── migrations/
│   │   │   └── __init__.py
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── urls.py
│   │   └── tests.py
│   │
│   └── api/                   # API utilities
│       ├── __init__.py
│       ├── pagination.py      # Global pagination
│       ├── renderers.py       # Custom renderers
│       ├── versioning.py      # API versioning
│       └── exceptions.py      # Custom exceptions
│
├── static/
│   └── .gitkeep
│
├── media/
│   └── .gitkeep
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py           # Pytest configuration
│   ├── factories.py          # Test factories
│   └── test_api.py
│
├── scripts/
│   ├── manage_db.py
│   └── seed_data.py
│
└── logs/
    └── .gitkeep
```

---

## Step 1: Project Initialization

### 1.1 Create Project Folder and Django Project

```bash
mkdir django-app
cd django-app
git init

# Create virtual environment
python -m venv venv

# Activate (Linux/Mac)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate
```

### 1.2 Install Django and Dependencies

```bash
pip install django==4.2.7 djangorestframework django-cors-headers
pip install python-dotenv psycopg2-binary
pip install pytest pytest-django pytest-cov factory-boy
pip install black flake8 isort mypy
```

### 1.3 Create Django Project

```bash
# Create Django project with custom structure
django-admin startproject config .

# Rename myproject to config (if not already done)
# This will create:
# - config/
# - manage.py
```

### 1.4 Create Apps

```bash
python manage.py startapp items
python manage.py startapp users

# Move apps to apps folder (optional but organized)
mkdir apps
mv items apps/
mv users apps/
```

---

## Step 2: Create Folder Structure

```bash
# Create folders
mkdir -p apps/{items,users}
mkdir -p apps/items/migrations
mkdir -p apps/users/migrations
mkdir -p config/settings
mkdir -p tests
mkdir -p scripts
mkdir -p logs
mkdir -p static media

# Add __init__.py files
touch apps/__init__.py
touch config/settings/__init__.py
touch apps/items/__init__.py
touch apps/users/__init__.py
touch tests/__init__.py
```

---

## Step 3: Configure Settings

### 3.1 config/settings/base.py

```python
"""Base Django settings."""
import os
from pathlib import Path
from dotenv import load_dotenv

load_dotenv()

# Build paths inside the project
BASE_DIR = Path(__file__).resolve().parent.parent.parent

# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.getenv('SECRET_KEY', 'django-insecure-change-this-in-production')

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = os.getenv('DEBUG', 'False').lower() == 'true'

ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', 'localhost,127.0.0.1').split(',')

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Third party
    'rest_framework',
    'corsheaders',
    
    # Local apps
    'apps.items',
    'apps.users',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'config.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
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

WSGI_APPLICATION = 'config.wsgi.application'

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DB_NAME', 'mydb'),
        'USER': os.getenv('DB_USER', 'postgres'),
        'PASSWORD': os.getenv('DB_PASSWORD', 'password'),
        'HOST': os.getenv('DB_HOST', 'localhost'),
        'PORT': os.getenv('DB_PORT', '5432'),
    }
}

# Password validation
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

# Internationalization
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

# Static files (CSS, JavaScript, Images)
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

# Media files
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# Default primary key field type
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# REST Framework Configuration
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
    'DEFAULT_FILTER_BACKENDS': [
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
        'django_filters.rest_framework.DjangoFilterBackend',
    ],
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
    ],
}

# CORS Configuration
CORS_ALLOWED_ORIGINS = os.getenv(
    'CORS_ALLOWED_ORIGINS',
    'http://localhost:3000,http://localhost:8000'
).split(',')

CORS_ALLOW_CREDENTIALS = True

# Logging
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': BASE_DIR / 'logs' / 'django.log',
        },
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
        },
    },
    'root': {
        'handlers': ['console', 'file'],
        'level': 'INFO',
    },
}
```

### 3.2 config/settings/development.py

```python
"""Development settings."""
from .base import *

DEBUG = True
ALLOWED_HOSTS = ['*']

# Database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# CORS for development
CORS_ALLOW_ALL_ORIGINS = True

# Logging
LOGGING['loggers'] = {
    'django': {
        'handlers': ['console'],
        'level': 'DEBUG',
    },
}
```

### 3.3 config/settings/production.py

```python
"""Production settings."""
from .base import *

DEBUG = False
ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', '').split(',')

# Security settings for production
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_SECURITY_POLICY = {
    'default-src': ("'self'",),
}

# Database - Use environment variables
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DB_NAME'),
        'USER': os.getenv('DB_USER'),
        'PASSWORD': os.getenv('DB_PASSWORD'),
        'HOST': os.getenv('DB_HOST'),
        'PORT': os.getenv('DB_PORT', '5432'),
    }
}

# CORS
CORS_ALLOWED_ORIGINS = os.getenv('CORS_ALLOWED_ORIGINS', '').split(',')
```

### 3.4 config/settings/testing.py

```python
"""Testing settings."""
from .base import *

DEBUG = True

# Use SQLite for testing
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': ':memory:',
    }
}

# Disable migrations for faster testing
class DisableMigrations:
    def __contains__(self, item):
        return True

    def __getitem__(self, item):
        return None

MIGRATION_MODULES = DisableMigrations()

# Use simple password hasher for tests
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.MD5PasswordHasher',
]
```

### 3.5 config/settings/__init__.py

```python
"""Settings initialization."""
import os

# Import settings based on environment
environment = os.getenv('ENVIRONMENT', 'development')

if environment == 'production':
    from .production import *
elif environment == 'testing':
    from .testing import *
else:
    from .development import *
```

---

## Step 4: Create Models

### 4.1 apps/items/models.py

```python
"""Item models."""
from django.db import models
from django.core.validators import MinValueValidator
from django.utils.timezone import now


class Item(models.Model):
    """Item model."""
    
    name = models.CharField(max_length=255, db_index=True)
    description = models.TextField(blank=True, null=True)
    price = models.DecimalField(
        max_digits=10,
        decimal_places=2,
        validators=[MinValueValidator(0)]
    )
    tax = models.DecimalField(
        max_digits=10,
        decimal_places=2,
        default=0,
        validators=[MinValueValidator(0)]
    )
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    is_active = models.BooleanField(default=True)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['name']),
            models.Index(fields=['created_at']),
        ]
        verbose_name = 'Item'
        verbose_name_plural = 'Items'
    
    def __str__(self):
        return self.name
    
    def total_price(self):
        """Calculate total price with tax."""
        return self.price + self.tax
```

### 4.2 apps/users/models.py

```python
"""User models."""
from django.db import models
from django.contrib.auth.models import AbstractUser


class CustomUser(AbstractUser):
    """Custom user model."""
    
    bio = models.TextField(blank=True, null=True)
    avatar = models.ImageField(upload_to='avatars/', blank=True, null=True)
    phone = models.CharField(max_length=20, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = 'User'
        verbose_name_plural = 'Users'
    
    def __str__(self):
        return self.username
```

---

## Step 5: Create Serializers

### 5.1 apps/items/serializers.py

```python
"""Item serializers."""
from rest_framework import serializers
from .models import Item


class ItemSerializer(serializers.ModelSerializer):
    """Item serializer."""
    
    total_price = serializers.SerializerMethodField()
    
    class Meta:
        model = Item
        fields = [
            'id',
            'name',
            'description',
            'price',
            'tax',
            'total_price',
            'is_active',
            'created_at',
            'updated_at',
        ]
        read_only_fields = ['id', 'created_at', 'updated_at']
    
    def get_total_price(self, obj):
        """Get total price."""
        return obj.total_price()


class ItemListSerializer(serializers.ModelSerializer):
    """Simplified item serializer for list view."""
    
    class Meta:
        model = Item
        fields = ['id', 'name', 'price', 'tax']


class ItemCreateUpdateSerializer(serializers.ModelSerializer):
    """Serializer for creating/updating items."""
    
    class Meta:
        model = Item
        fields = ['name', 'description', 'price', 'tax', 'is_active']
```

---

## Step 6: Create Views

### 6.1 apps/items/views.py

```python
"""Item views."""
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from rest_framework.filters import SearchFilter, OrderingFilter
from django_filters.rest_framework import DjangoFilterBackend

from .models import Item
from .serializers import (
    ItemSerializer,
    ItemListSerializer,
    ItemCreateUpdateSerializer
)


class ItemViewSet(viewsets.ModelViewSet):
    """ViewSet for Item model."""
    
    queryset = Item.objects.all()
    serializer_class = ItemSerializer
    permission_classes = [IsAuthenticated]
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_fields = ['is_active', 'created_at']
    search_fields = ['name', 'description']
    ordering_fields = ['created_at', 'price', 'name']
    ordering = ['-created_at']
    
    def get_serializer_class(self):
        """Get serializer based on action."""
        if self.action == 'list':
            return ItemListSerializer
        elif self.action in ['create', 'update', 'partial_update']:
            return ItemCreateUpdateSerializer
        return ItemSerializer
    
    @action(detail=False, methods=['get'])
    def by_price(self, request):
        """Get items within price range."""
        min_price = request.query_params.get('min', 0)
        max_price = request.query_params.get('max', float('inf'))
        
        items = self.queryset.filter(price__gte=min_price, price__lte=max_price)
        serializer = self.get_serializer(items, many=True)
        return Response(serializer.data)
    
    @action(detail=True, methods=['post'])
    def activate(self, request, pk=None):
        """Activate an item."""
        item = self.get_object()
        item.is_active = True
        item.save()
        return Response({'status': 'Item activated'})
    
    @action(detail=True, methods=['post'])
    def deactivate(self, request, pk=None):
        """Deactivate an item."""
        item = self.get_object()
        item.is_active = False
        item.save()
        return Response({'status': 'Item deactivated'})
```

---

## Step 7: Create Admin

### 7.1 apps/items/admin.py

```python
"""Item admin configuration."""
from django.contrib import admin
from .models import Item


@admin.register(Item)
class ItemAdmin(admin.ModelAdmin):
    """Item admin."""
    
    list_display = ('id', 'name', 'price', 'tax', 'is_active', 'created_at')
    list_filter = ('is_active', 'created_at', 'price')
    search_fields = ('name', 'description')
    readonly_fields = ('created_at', 'updated_at')
    ordering = ['-created_at']
    
    fieldsets = (
        ('Basic Info', {
            'fields': ('name', 'description')
        }),
        ('Pricing', {
            'fields': ('price', 'tax')
        }),
        ('Status', {
            'fields': ('is_active',)
        }),
        ('Timestamps', {
            'fields': ('created_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )
```

---

## Step 8: Create URLs

### 8.1 apps/items/urls.py

```python
"""Item URLs."""
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ItemViewSet

router = DefaultRouter()
router.register(r'items', ItemViewSet, basename='item')

urlpatterns = [
    path('', include(router.urls)),
]
```

### 8.2 config/urls.py

```python
"""Project URLs."""
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/v1/', include('apps.items.urls')),
    path('api-auth/', include('rest_framework.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

---

## Step 9: Testing Setup

### 9.1 tests/conftest.py

```python
"""Pytest configuration."""
import os
import django
from django.conf import settings

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings.testing')
django.setup()


import pytest
from rest_framework.test import APIClient
from django.contrib.auth.models import User


@pytest.fixture
def api_client():
    """API client fixture."""
    return APIClient()


@pytest.fixture
def user():
    """Create test user."""
    return User.objects.create_user(
        username='testuser',
        email='test@example.com',
        password='testpass123'
    )


@pytest.fixture
def authenticated_client(api_client, user):
    """Authenticated API client."""
    api_client.force_authenticate(user=user)
    return api_client
```

### 9.2 tests/test_api.py

```python
"""API tests."""
import pytest
from apps.items.models import Item


@pytest.mark.django_db
class TestItemAPI:
    """Item API tests."""
    
    def test_create_item(self, authenticated_client):
        """Test creating an item."""
        response = authenticated_client.post(
            '/api/v1/items/',
            {
                'name': 'Test Item',
                'description': 'Test description',
                'price': '99.99',
                'tax': '10.00'
            },
            format='json'
        )
        assert response.status_code == 201
        assert Item.objects.count() == 1
    
    def test_list_items(self, authenticated_client):
        """Test listing items."""
        Item.objects.create(name='Item 1', price=10.0)
        Item.objects.create(name='Item 2', price=20.0)
        
        response = authenticated_client.get('/api/v1/items/')
        assert response.status_code == 200
        assert len(response.data['results']) == 2
    
    def test_get_item(self, authenticated_client):
        """Test getting single item."""
        item = Item.objects.create(name='Test Item', price=99.99)
        response = authenticated_client.get(f'/api/v1/items/{item.id}/')
        
        assert response.status_code == 200
        assert response.data['name'] == 'Test Item'
    
    def test_update_item(self, authenticated_client):
        """Test updating item."""
        item = Item.objects.create(name='Old Name', price=10.0)
        
        response = authenticated_client.patch(
            f'/api/v1/items/{item.id}/',
            {'name': 'New Name'},
            format='json'
        )
        
        assert response.status_code == 200
        item.refresh_from_db()
        assert item.name == 'New Name'
    
    def test_delete_item(self, authenticated_client):
        """Test deleting item."""
        item = Item.objects.create(name='Test Item', price=10.0)
        response = authenticated_client.delete(f'/api/v1/items/{item.id}/')
        
        assert response.status_code == 204
        assert Item.objects.count() == 0
```

---

## Step 10: Configuration Files

### 10.1 .env.example

```
ENVIRONMENT=development
DEBUG=True
SECRET_KEY=django-insecure-your-secret-key-here

DB_ENGINE=django.db.backends.postgresql
DB_NAME=mydb
DB_USER=postgres
DB_PASSWORD=password
DB_HOST=localhost
DB_PORT=5432

ALLOWED_HOSTS=localhost,127.0.0.1
CORS_ALLOWED_ORIGINS=http://localhost:3000,http://localhost:8000

EMAIL_BACKEND=django.core.mail.backends.console.EmailBackend
```

### 10.2 requirements.txt

```
Django==4.2.7
djangorestframework==3.14.0
django-cors-headers==4.3.1
django-filter==23.4
psycopg2-binary==2.9.9
python-dotenv==1.0.0
pytest==7.4.3
pytest-django==4.7.0
pytest-cov==4.1.0
factory-boy==3.3.0
black==23.12.0
flake8==6.1.0
isort==5.13.2
mypy==1.7.1
djangorestframework-simplejwt==5.3.2
django-admin-interface==0.25.2
Pillow==10.1.0
```

### 10.3 pytest.ini

```ini
[pytest]
DJANGO_SETTINGS_MODULE = config.settings.testing
python_files = tests.py test_*.py *_tests.py
addopts = --cov=apps --cov-report=html
```

### 10.4 Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN apt-get update && apt-get install -y postgresql-client

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN python manage.py collectstatic --noinput

CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000"]
```

### 10.5 docker-compose.yml

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  django:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DEBUG=True
      - DB_HOST=postgres
      - ENVIRONMENT=development
    volumes:
      - .:/app
    command: python manage.py runserver 0.0.0.0:8000
    depends_on:
      - postgres

volumes:
  postgres_data:
```

---

## Running Django

### Initial Setup

```bash
# Activate virtual environment
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Copy environment file
cp .env.example .env

# Make migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Run development server
python manage.py runserver
```

### Access Django

- Admin: http://localhost:8000/admin/
- API: http://localhost:8000/api/v1/
- API Browsable: http://localhost:8000/api/v1/items/

### Testing

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=apps

# Run specific test file
pytest tests/test_api.py

# Run with verbose output
pytest -v
```

### Common Commands

```bash
# Create new app
python manage.py startapp appname

# Create migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Run shell
python manage.py shell

# Collect static files
python manage.py collectstatic

# Check for issues
python manage.py check

# Format code
black .
isort .
flake8 .
```

---

## Best Practices

✅ **Use custom user model** if you need custom fields  
✅ **Separate settings** for dev, staging, production  
✅ **Use environment variables** for sensitive data  
✅ **Implement proper permissions** on views  
✅ **Use select_related** and **prefetch_related** to optimize queries  
✅ **Test thoroughly** with pytest  
✅ **Use signals carefully** - they can impact performance  
✅ **Document API endpoints** with docstrings  
✅ **Implement logging** for debugging  
✅ **Use migrations** for database changes  

---

This Django setup is production-ready with DRF, proper structure, and testing!
