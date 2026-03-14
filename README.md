# Learning Documentation: Grafana, Terraform & Python Web Frameworks

![Status](https://img.shields.io/badge/Status-Active-brightgreen) ![Version](https://img.shields.io/badge/Version-1.0-blue) ![License](https://img.shields.io/badge/License-MIT-green)

## Table of Contents

- [Introduction](#introduction)
- [Grafana](#grafana)
- [Terraform](#terraform)
- [Python Web Frameworks](#python-web-frameworks)
  - [FastAPI](#fastapi)
  - [Django](#django)
  - [Flask](#flask)
- [Project Structure](#project-structure)
- [Installation & Setup](#installation--setup)
- [Best Practices](#best-practices)
- [Resources](#resources)

---

## Introduction

This documentation serves as a comprehensive guide to learning and implementing:
- **Grafana**: Open-source visualization and monitoring platform
- **Terraform**: Infrastructure as Code (IaC) tool for managing cloud resources
- **Python Web Frameworks**: FastAPI, Django, and Flask for building robust web applications

Each section includes concepts, setup instructions, code examples, and best practices.

---

## Grafana

### Overview
Grafana is an open-source platform for real-time data visualization and monitoring. It allows you to query, visualize, alert, and explore metrics regardless of where the data is stored.

### Key Concepts

#### 1. **Dashboards**
- Visual representation of your data
- Composed of panels showing different metrics
- Can be organized by teams or use cases

#### 2. **Data Sources**
- Prometheus
- InfluxDB
- Elasticsearch
- Graphite
- Cloud monitoring services (AWS CloudWatch, Azure Monitor, GCP Stackdriver)

#### 3. **Alerts**
- Threshold-based alerting
- Multiple notification channels (Email, Slack, PagerDuty, etc.)
- Alert rules and conditions

#### 4. **Panels**
- Graph/Timeseries
- Stat (single value)
- Gauge
- Table
- Heatmap
- Bar gauge

### Basic Setup

```bash
# Using Docker
docker run -d -p 3000:3000 --name grafana grafana/grafana:latest

# Default credentials
# Username: admin
# Password: admin
```

### Creating Your First Dashboard

#### Step 1: Access Grafana
```
http://localhost:3000
```

#### Step 2: Add Data Source
1. Go to Configuration → Data Sources
2. Click "Add data source"
3. Select your database type (e.g., Prometheus)
4. Configure connection details

#### Step 3: Create Dashboard
1. Click "+" → Dashboard
2. Add a new panel
3. Configure the query from your data source
4. Customize visualization options
5. Save dashboard

### Example Prometheus Query

```promql
# CPU Usage
node_cpu_seconds_total{mode="user"}

# Memory Usage
node_memory_MemFree_bytes / node_memory_MemTotal_bytes * 100

# Request Rate
rate(http_requests_total[5m])
```

### Dashboard JSON Example

```json
{
  "dashboard": {
    "title": "System Monitoring",
    "panels": [
      {
        "title": "CPU Usage",
        "type": "timeseries",
        "targets": [
          {
            "expr": "rate(node_cpu_seconds_total[5m])"
          }
        ]
      }
    ]
  }
}
```

### Alert Configuration

```yaml
# Alert Rule Example
alert: HighCPUUsage
expr: avg(node_cpu_seconds_total) > 0.8
for: 5m
annotations:
  summary: "High CPU usage detected"
  description: "CPU usage is above 80%"
```

### Key Plugins & Extensions
- Worldmap Panel
- Pie Chart
- Status Panel
- Singlestat
- Custom metrics visualization

### Grafana Best Practices
- Use consistent naming conventions
- Organize dashboards by team/service
- Implement proper RBAC (Role-Based Access Control)
- Version control your dashboard JSON
- Use variables for reusable dashboards
- Set up meaningful alerts with proper thresholds
- Document your dashboards

---

## Terraform

### Overview
Terraform is an Infrastructure as Code tool that allows you to define, preview, and deploy infrastructure using a declarative configuration language.

### Key Concepts

#### 1. **State Management**
- Terraform maintains a state file tracking resource status
- Remote state backends: S3, Terraform Cloud, Azure Storage, etc.
- State locking prevents concurrent modifications

#### 2. **Providers**
- AWS, Azure, GCP, Kubernetes, Docker, etc.
- Plugins that interact with cloud platforms
- Multiple providers in single configuration

#### 3. **Resources**
- Infrastructure components (EC2, RDS, VPC, etc.)
- Each resource has attributes and data sources

#### 4. **Variables & Outputs**
- Input variables for dynamic configurations
- Output values to share resource information

### Directory Structure

```
terraform/
├── main.tf              # Primary configuration
├── variables.tf         # Input variable definitions
├── outputs.tf           # Output definitions
├── terraform.tfvars     # Variable values
├── provider.tf          # Provider configuration
├── backend.tf           # State backend configuration
├── modules/             # Reusable modules
│   ├── vpc/
│   ├── rds/
│   └── ec2/
└── environments/        # Environment-specific configs
    ├── dev/
    ├── staging/
    └── prod/
```

### Basic Syntax

#### Provider Configuration

```hcl
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Environment = var.environment
      ManagedBy   = "Terraform"
      Project     = var.project_name
    }
  }
}
```

#### Variables Definition

```hcl
# variables.tf

variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 2
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "tags" {
  description = "Common tags"
  type        = map(string)
  default = {
    Project = "MyProject"
    Owner   = "DevTeam"
  }
}
```

#### Basic AWS Resources

```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true

  tags = {
    Name = "${var.environment}-vpc"
  }
}

# Subnet
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "${var.aws_region}a"
  map_public_ip_on_launch = true

  tags = {
    Name = "${var.environment}-public-subnet"
  }
}

# Security Group
resource "aws_security_group" "web" {
  name        = "web-security-group"
  description = "Security group for web servers"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.environment}-web-sg"
  }
}

# EC2 Instance
resource "aws_instance" "web" {
  count                = var.instance_count
  ami                  = data.aws_ami.ubuntu.id
  instance_type        = "t3.micro"
  subnet_id            = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]

  tags = {
    Name = "${var.environment}-web-${count.index + 1}"
  }
}
```

#### Outputs

```hcl
# outputs.tf

output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "instance_ids" {
  description = "EC2 instance IDs"
  value       = aws_instance.web[*].id
}

output "instance_ips" {
  description = "EC2 instance public IPs"
  value       = aws_instance.web[*].public_ip
}
```

### Terraform Commands

```bash
# Initialize Terraform working directory
terraform init

# Validate configuration syntax
terraform validate

# Format HCL files
terraform fmt -recursive

# Generate and review execution plan
terraform plan -out=tfplan

# Apply changes to infrastructure
terraform apply tfplan

# Destroy all infrastructure
terraform destroy

# Show current state
terraform show

# Refresh state from real infrastructure
terraform refresh

# Target specific resource
terraform apply -target=aws_instance.web[0]

# Get output values
terraform output
terraform output instance_ids
```

### Modules

```hcl
# modules/vpc/main.tf

resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = var.vpc_name
  }
}

# modules/vpc/variables.tf
variable "vpc_cidr" {
  type = string
}

variable "vpc_name" {
  type = string
}

# modules/vpc/outputs.tf
output "vpc_id" {
  value = aws_vpc.this.id
}
```

```hcl
# Using modules in main.tf

module "vpc" {
  source = "./modules/vpc"

  vpc_cidr = "10.0.0.0/16"
  vpc_name = "${var.environment}-vpc"
}
```

### Terraform Best Practices
- Use remote state storage with locking
- Implement state file encryption
- Use variable validation
- Organize code with modules
- Version your providers and Terraform
- Use `terraform fmt` for consistency
- Implement tagging strategy
- Use workspaces for multiple environments
- Keep secrets out of version control
- Document your infrastructure
- Use `depends_on` carefully
- Implement CI/CD for Terraform changes

---

## Python Web Frameworks

### Comparison Table

| Aspect | FastAPI | Django | Flask |
|--------|---------|--------|-------|
| **Type** | Modern, async-first | Full-featured framework | Lightweight, flexible |
| **Learning Curve** | Moderate | Steep | Gentle |
| **Async Support** | Native (async/await) | Limited (3.1+) | Limited |
| **ORM** | None (use SQLAlchemy) | Built-in ORM | None (use SQLAlchemy) |
| **Admin Panel** | None | Built-in | Third-party |
| **Project Size** | Small to Medium | Large enterprise | Small to Medium |
| **Performance** | Very Fast | Good | Good |
| **Documentation** | Excellent | Excellent | Good |

---

## FastAPI

### Overview
FastAPI is a modern, fast web framework for building APIs with Python 3.7+. It uses Starlette for the web framework and Pydantic for data validation.

### Key Features
- Automatic API documentation (Swagger UI, ReDoc)
- Type hints for validation
- Async/await support
- High performance
- Easy testing

### Installation

```bash
pip install fastapi uvicorn pydantic sqlalchemy python-dotenv
```

### Basic Application

```python
# main.py
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI(
    title="My API",
    description="A sample FastAPI application",
    version="1.0.0"
)

# Data Models
class Item(BaseModel):
    id: int
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

class ItemUpdate(BaseModel):
    name: Optional[str] = None
    description: Optional[str] = None
    price: Optional[float] = None
    tax: Optional[float] = None

# In-memory database
items_db = {}

# Routes
@app.get("/", tags=["Root"])
async def read_root():
    return {"message": "Welcome to FastAPI"}

@app.post("/items/", response_model=Item, status_code=201, tags=["Items"])
async def create_item(item: Item):
    items_db[item.id] = item
    return item

@app.get("/items/{item_id}", response_model=Item, tags=["Items"])
async def read_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return items_db[item_id]

@app.get("/items/", response_model=list[Item], tags=["Items"])
async def list_items(skip: int = 0, limit: int = 10):
    return list(items_db.values())[skip:skip+limit]

@app.put("/items/{item_id}", response_model=Item, tags=["Items"])
async def update_item(item_id: int, item: ItemUpdate):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    
    stored_item = items_db[item_id]
    update_data = item.dict(exclude_unset=True)
    updated_item = stored_item.copy(update={**update_data})
    items_db[item_id] = updated_item
    return updated_item

@app.delete("/items/{item_id}", status_code=204, tags=["Items"])
async def delete_item(item_id: int):
    if item_id not in items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    del items_db[item_id]
```

### Database Integration

```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

DATABASE_URL = "sqlite:///./test.db"

engine = create_engine(
    DATABASE_URL, 
    connect_args={"check_same_thread": False}
)

SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# models.py
from sqlalchemy import Column, Integer, String, Float
from database import Base

class ItemModel(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    description = Column(String, index=True)
    price = Column(Float)
    tax = Column(Float, default=0.0)

# schemas.py
from pydantic import BaseModel

class ItemCreate(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

class Item(ItemCreate):
    id: int

    class Config:
        from_attributes = True
```

### Running FastAPI

```bash
# Development server
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Production server
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker
```

### Testing

```python
# test_main.py
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_read_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"message": "Welcome to FastAPI"}

def test_create_item():
    response = client.post(
        "/items/",
        json={
            "id": 1,
            "name": "Test Item",
            "price": 99.99,
            "description": "A test item"
        }
    )
    assert response.status_code == 201
    assert response.json()["name"] == "Test Item"

def test_read_item():
    response = client.get("/items/1")
    assert response.status_code == 200
```

### FastAPI Best Practices
- Use type hints for all parameters
- Implement proper error handling
- Use dependency injection for common tasks
- Leverage async for I/O operations
- Validate input with Pydantic
- Use background tasks for long operations
- Implement proper logging
- Secure endpoints with authentication/authorization
- Use middleware for cross-cutting concerns

---

## Django

### Overview
Django is a high-level, batteries-included Python web framework that encourages rapid development and clean, pragmatic design.

### Installation

```bash
pip install django djangorestframework django-cors-headers python-dotenv
```

### Project Setup

```bash
# Create project
django-admin startproject myproject

# Create app
cd myproject
python manage.py startapp items

# Run migrations
python manage.py makemigrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Run development server
python manage.py runserver
```

### Project Structure

```
myproject/
├── myproject/
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── items/
│   ├── migrations/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── tests.py
│   ├── urls.py
│   ├── views.py
│   └── serializers.py
└── manage.py
```

### Models

```python
# items/models.py
from django.db import models
from django.core.validators import MinValueValidator

class Item(models.Model):
    name = models.CharField(max_length=255)
    description = models.TextField(blank=True, null=True)
    price = models.DecimalField(max_digits=10, decimal_places=2, validators=[MinValueValidator(0)])
    tax = models.DecimalField(max_digits=10, decimal_places=2, default=0, validators=[MinValueValidator(0)])
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['name']),
        ]

    def __str__(self):
        return self.name

    def total_price(self):
        return self.price + self.tax
```

### Serializers

```python
# items/serializers.py
from rest_framework import serializers
from .models import Item

class ItemSerializer(serializers.ModelSerializer):
    total_price = serializers.SerializerMethodField()

    class Meta:
        model = Item
        fields = ['id', 'name', 'description', 'price', 'tax', 'total_price', 'created_at', 'updated_at']
        read_only_fields = ['id', 'created_at', 'updated_at']

    def get_total_price(self, obj):
        return obj.total_price()
```

### Views (Class-Based)

```python
# items/views.py
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from .models import Item
from .serializers import ItemSerializer

class ItemViewSet(viewsets.ModelViewSet):
    queryset = Item.objects.all()
    serializer_class = ItemSerializer
    permission_classes = [IsAuthenticated]

    @action(detail=False, methods=['get'])
    def by_price(self, request):
        """Get items within price range"""
        min_price = request.query_params.get('min', 0)
        max_price = request.query_params.get('max', float('inf'))
        
        items = self.queryset.filter(price__gte=min_price, price__lte=max_price)
        serializer = self.get_serializer(items, many=True)
        return Response(serializer.data)

    def perform_create(self, serializer):
        """Save additional data on create"""
        serializer.save()
```

### URLs

```python
# items/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ItemViewSet

router = DefaultRouter()
router.register(r'items', ItemViewSet)

urlpatterns = [
    path('api/', include(router.urls)),
]

# myproject/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('items.urls')),
]
```

### Settings Configuration

```python
# myproject/settings.py
import os
from pathlib import Path
from dotenv import load_dotenv

load_dotenv()

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = os.getenv('SECRET_KEY', 'your-secret-key')
DEBUG = os.getenv('DEBUG', False) == 'True'
ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', 'localhost,127.0.0.1').split(',')

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'corsheaders',
    'items',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
    'DEFAULT_FILTER_BACKENDS': [
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ],
}

CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "http://localhost:8000",
]

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
```

### Django Admin

```python
# items/admin.py
from django.contrib import admin
from .models import Item

@admin.register(Item)
class ItemAdmin(admin.ModelAdmin):
    list_display = ('name', 'price', 'tax', 'created_at')
    list_filter = ('created_at', 'price')
    search_fields = ('name', 'description')
    readonly_fields = ('created_at', 'updated_at')
    fieldsets = (
        ('Basic Info', {
            'fields': ('name', 'description')
        }),
        ('Pricing', {
            'fields': ('price', 'tax')
        }),
        ('Timestamps', {
            'fields': ('created_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )
```

### Testing

```python
# items/tests.py
from django.test import TestCase, Client
from django.contrib.auth.models import User
from rest_framework.test import APITestCase
from rest_framework.test import APIClient
from .models import Item

class ItemAPITestCase(APITestCase):
    def setUp(self):
        self.client = APIClient()
        self.user = User.objects.create_user('testuser', password='123456')
        self.client.force_authenticate(user=self.user)

    def test_create_item(self):
        data = {
            'name': 'Test Item',
            'price': 99.99,
            'description': 'A test item'
        }
        response = self.client.post('/api/items/', data, format='json')
        self.assertEqual(response.status_code, 201)
        self.assertEqual(Item.objects.count(), 1)

    def test_list_items(self):
        Item.objects.create(name='Item 1', price=10.0)
        Item.objects.create(name='Item 2', price=20.0)
        response = self.client.get('/api/items/')
        self.assertEqual(response.status_code, 200)
        self.assertEqual(len(response.data), 2)
```

### Django Best Practices
- Use environment variables for configuration
- Implement proper authentication and permissions
- Use signals carefully (can impact performance)
- Leverage Django ORM querysets efficiently
- Implement caching strategies
- Use middleware for cross-cutting concerns
- Optimize database queries (select_related, prefetch_related)
- Implement proper logging
- Use management commands for recurring tasks
- Version your API

---

## Flask

### Overview
Flask is a lightweight, flexible micro-framework for building web applications. It gives developers maximum control while keeping code simple.

### Installation

```bash
pip install flask flask-sqlalchemy flask-marshmallow flask-cors python-dotenv
```

### Basic Application

```python
# app.py
from flask import Flask, request, jsonify
from flask_cors import CORS
from dotenv import load_dotenv
import os

load_dotenv()

app = Flask(__name__)
CORS(app)
app.config['JSON_SORT_KEYS'] = False

# Configuration
app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv(
    'DATABASE_URL',
    'sqlite:///items.db'
)
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

# In-memory database
items_db = {}

@app.route('/', methods=['GET'])
def home():
    return jsonify({"message": "Welcome to Flask API"}), 200

@app.route('/items', methods=['GET'])
def get_items():
    skip = request.args.get('skip', 0, type=int)
    limit = request.args.get('limit', 10, type=int)
    
    items_list = list(items_db.values())[skip:skip+limit]
    return jsonify(items_list), 200

@app.route('/items/<int:item_id>', methods=['GET'])
def get_item(item_id):
    if item_id not in items_db:
        return jsonify({"error": "Item not found"}), 404
    
    return jsonify(items_db[item_id]), 200

@app.route('/items', methods=['POST'])
def create_item():
    data = request.get_json()
    
    # Validation
    if not data or 'id' not in data or 'name' not in data:
        return jsonify({"error": "Missing required fields"}), 400
    
    item_id = data['id']
    if item_id in items_db:
        return jsonify({"error": "Item already exists"}), 409
    
    items_db[item_id] = {
        'id': item_id,
        'name': data['name'],
        'description': data.get('description'),
        'price': data.get('price', 0),
        'tax': data.get('tax', 0)
    }
    
    return jsonify(items_db[item_id]), 201

@app.route('/items/<int:item_id>', methods=['PUT'])
def update_item(item_id):
    if item_id not in items_db:
        return jsonify({"error": "Item not found"}), 404
    
    data = request.get_json()
    
    # Update fields
    for key in ['name', 'description', 'price', 'tax']:
        if key in data:
            items_db[item_id][key] = data[key]
    
    return jsonify(items_db[item_id]), 200

@app.route('/items/<int:item_id>', methods=['DELETE'])
def delete_item(item_id):
    if item_id not in items_db:
        return jsonify({"error": "Item not found"}), 404
    
    deleted_item = items_db.pop(item_id)
    return jsonify(deleted_item), 200

# Error handlers
@app.errorhandler(404)
def not_found(error):
    return jsonify({"error": "Resource not found"}), 404

@app.errorhandler(500)
def server_error(error):
    return jsonify({"error": "Internal server error"}), 500

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

### Database Integration

```python
# models.py
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

db = SQLAlchemy()

class Item(db.Model):
    __tablename__ = 'items'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(255), nullable=False, index=True)
    description = db.Column(db.Text, nullable=True)
    price = db.Column(db.Float, nullable=False, default=0)
    tax = db.Column(db.Float, nullable=False, default=0)
    created_at = db.Column(db.DateTime, nullable=False, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, nullable=False, default=datetime.utcnow, onupdate=datetime.utcnow)

    def to_dict(self):
        return {
            'id': self.id,
            'name': self.name,
            'description': self.description,
            'price': self.price,
            'tax': self.tax,
            'created_at': self.created_at.isoformat(),
            'updated_at': self.updated_at.isoformat()
        }

# app.py (updated)
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from models import db, Item

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///items.db'
db.init_app(app)

with app.app_context():
    db.create_all()
```

### Schema Validation

```python
# schemas.py
from marshmallow import Schema, fields, validate

class ItemSchema(Schema):
    id = fields.Int(dump_only=True)
    name = fields.Str(required=True, validate=validate.Length(min=1, max=255))
    description = fields.Str(allow_none=True)
    price = fields.Float(required=True, validate=validate.Range(min=0))
    tax = fields.Float(required=True, validate=validate.Range(min=0))
    created_at = fields.DateTime(dump_only=True)
    updated_at = fields.DateTime(dump_only=True)

item_schema = ItemSchema()
items_schema = ItemSchema(many=True)
```

### Blueprints (Modular Structure)

```python
# routes/items.py
from flask import Blueprint, request, jsonify
from flask_marshmallow import Marshmallow
from models import db, Item
from schemas import item_schema, items_schema

items_bp = Blueprint('items', __name__, url_prefix='/api/items')
ma = Marshmallow()

@items_bp.route('', methods=['GET'])
def list_items():
    skip = request.args.get('skip', 0, type=int)
    limit = request.args.get('limit', 10, type=int)
    
    items = Item.query.offset(skip).limit(limit).all()
    return jsonify(items_schema.dump(items)), 200

@items_bp.route('/<int:item_id>', methods=['GET'])
def get_item(item_id):
    item = Item.query.get_or_404(item_id)
    return jsonify(item_schema.dump(item)), 200

@items_bp.route('', methods=['POST'])
def create_item():
    errors = item_schema.validate(request.get_json())
    if errors:
        return jsonify(errors), 400
    
    item = Item(**request.get_json())
    db.session.add(item)
    db.session.commit()
    
    return jsonify(item_schema.dump(item)), 201

@items_bp.route('/<int:item_id>', methods=['PUT'])
def update_item(item_id):
    item = Item.query.get_or_404(item_id)
    errors = item_schema.validate(request.get_json(), partial=True)
    if errors:
        return jsonify(errors), 400
    
    for key, value in request.get_json().items():
        setattr(item, key, value)
    
    db.session.commit()
    return jsonify(item_schema.dump(item)), 200

@items_bp.route('/<int:item_id>', methods=['DELETE'])
def delete_item(item_id):
    item = Item.query.get_or_404(item_id)
    db.session.delete(item)
    db.session.commit()
    
    return '', 204

# app.py
from flask import Flask
from routes.items import items_bp
from models import db

app = Flask(__name__)
app.register_blueprint(items_bp)
db.init_app(app)
```

### Testing

```python
# test_app.py
import pytest
from app import app, db
from models import Item

@pytest.fixture
def client():
    app.config['TESTING'] = True
    app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'
    
    with app.app_context():
        db.create_all()
        yield app.test_client()
        db.session.remove()
        db.drop_all()

def test_create_item(client):
    response = client.post('/api/items', json={
        'name': 'Test Item',
        'price': 99.99,
        'tax': 10.0,
        'description': 'Test'
    })
    
    assert response.status_code == 201
    data = response.get_json()
    assert data['name'] == 'Test Item'

def test_list_items(client):
    # Create test items
    client.post('/api/items', json={
        'name': 'Item 1',
        'price': 10.0,
        'tax': 1.0
    })
    
    response = client.get('/api/items')
    assert response.status_code == 200
    data = response.get_json()
    assert len(data) == 1
```

### Flask Best Practices
- Use blueprints to organize code
- Implement proper error handling
- Use SQLAlchemy ORM for database operations
- Validate input with marshmallow
- Implement logging
- Use environment variables for configuration
- Implement authentication/authorization
- Use middleware for common tasks
- Test your endpoints thoroughly
- Document your API

---

## Project Structure

Here's a recommended structure for managing all three technologies:

```
my-learning-project/
├── README.md                    # This file
├── .gitignore
├── .env.example
│
├── terraform/                   # Infrastructure as Code
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── provider.tf
│   ├── backend.tf
│   ├── terraform.tfvars
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── prod/
│   └── modules/
│       ├── vpc/
│       ├── rds/
│       └── ec2/
│
├── grafana/                     # Monitoring & Visualization
│   ├── dashboards/
│   │   ├── system-monitoring.json
│   │   ├── application-metrics.json
│   │   └── infrastructure.json
│   ├── alerts/
│   │   └── alert-rules.yaml
│   ├── provisioning/
│   │   ├── datasources/
│   │   └── dashboards/
│   └── docker-compose.yml
│
├── fastapi-app/                 # FastAPI Project
│   ├── main.py
│   ├── requirements.txt
│   ├── .env
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── database.py
│   │   ├── models/
│   │   ├── schemas/
│   │   ├── routes/
│   │   ├── dependencies.py
│   │   └── utils/
│   ├── tests/
│   ├── Dockerfile
│   └── docker-compose.yml
│
├── django-app/                  # Django Project
│   ├── manage.py
│   ├── requirements.txt
│   ├── .env
│   ├── myproject/
│   ├── apps/
│   │   ├── items/
│   │   ├── users/
│   │   └── ...
│   ├── tests/
│   ├── Dockerfile
│   └── docker-compose.yml
│
├── flask-app/                   # Flask Project
│   ├── app.py
│   ├── requirements.txt
│   ├── .env
│   ├── app/
│   │   ├── __init__.py
│   │   ├── models/
│   │   ├── routes/
│   │   ├── schemas/
│   │   └── utils/
│   ├── tests/
│   ├── Dockerfile
│   └── docker-compose.yml
│
└── scripts/                     # Utility Scripts
    ├── setup.sh
    ├── deploy.sh
    └── backup.sh
```

---

## Installation & Setup

### Prerequisites
- Python 3.8+
- pip or conda
- Docker (optional)
- Terraform 1.0+
- Git

### Initial Setup

```bash
# Clone repository
git clone <your-repo-url>
cd my-learning-project

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Linux/Mac:
source venv/bin/activate
# On Windows:
venv\Scripts\activate

# Install dependencies for each framework
pip install -r fastapi-app/requirements.txt
pip install -r django-app/requirements.txt
pip install -r flask-app/requirements.txt
```

### Environment Configuration

Create `.env` files in each project:

```bash
# FastAPI .env
DATABASE_URL=sqlite:///./test.db
SECRET_KEY=your-secret-key
DEBUG=True

# Django .env
SECRET_KEY=your-secret-key
DEBUG=True
DB_ENGINE=django.db.backends.postgresql
DB_NAME=mydb
DB_USER=postgres
DB_PASSWORD=password
DB_HOST=localhost
DB_PORT=5432

# Flask .env
DATABASE_URL=sqlite:///items.db
SECRET_KEY=your-secret-key
FLASK_ENV=development
```

### Running Applications

```bash
# FastAPI
cd fastapi-app
uvicorn main:app --reload

# Django
cd django-app
python manage.py runserver

# Flask
cd flask-app
python app.py
```

### Terraform Deployment

```bash
cd terraform
terraform init
terraform plan
terraform apply
```

### Grafana Setup

```bash
cd grafana
docker-compose up -d
```

---

## Best Practices

### Code Quality
- Use linting tools: `flake8`, `pylint`, `black`
- Type checking: `mypy`
- Testing: Aim for 80%+ coverage
- Documentation: Docstrings for all functions

### Security
- Never commit secrets (use `.env`)
- Use HTTPS in production
- Implement CORS carefully
- Validate all inputs
- Use parameterized queries
- Keep dependencies updated

### Performance
- Use caching strategies
- Optimize database queries
- Implement pagination
- Use async where appropriate
- Monitor with Grafana

### DevOps
- Version control all code
- Use CI/CD pipelines
- Implement automated testing
- Use Terraform for infrastructure
- Monitor and alert with Grafana

---

## Resources

### Grafana
- [Official Documentation](https://grafana.com/docs/)
- [Grafana Dashboard Library](https://grafana.com/grafana/dashboards/)
- [PromQL Documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/)

### Terraform
- [Official Documentation](https://www.terraform.io/docs/)
- [AWS Provider Docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Registry](https://registry.terraform.io/)

### FastAPI
- [Official Documentation](https://fastapi.tiangolo.com/)
- [Starlette Documentation](https://www.starlette.io/)
- [Pydantic Documentation](https://docs.pydantic.dev/)

### Django
- [Official Documentation](https://docs.djangoproject.com/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Django Best Practices](https://docs.djangoproject.com/en/stable/misc/design-philosophies/)

### Flask
- [Official Documentation](https://flask.palletsprojects.com/)
- [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/)
- [Flask-CORS](https://flask-cors.readthedocs.io/)

### General Resources
- [Python Official Docs](https://docs.python.org/3/)
- [Real Python Tutorials](https://realpython.com/)
- [DevOps Best Practices](https://aws.amazon.com/devops/what-is-devops/)

---

## Contributing

To contribute to this documentation:

1. Create a feature branch
2. Make your changes
3. Test thoroughly
4. Submit a pull request
5. Ensure all tests pass

## License

This documentation is licensed under the MIT License - see the LICENSE file for details.

## Author

Created as comprehensive learning documentation for Grafana, Terraform, and Python Web Frameworks.

---

**Last Updated**: March 15, 2026

**Version**: 1.0.0

**Status**: Active Development

---

## Quick Command Reference

### Terraform
```bash
terraform init                  # Initialize
terraform plan                  # Review changes
terraform apply                 # Deploy
terraform destroy               # Remove resources
```

### FastAPI
```bash
uvicorn main:app --reload       # Dev server
pytest                          # Run tests
black .                         # Format code
```

### Django
```bash
python manage.py runserver      # Dev server
python manage.py makemigrations # Create migrations
python manage.py migrate        # Apply migrations
python manage.py createsuperuser # Create admin user
python manage.py test           # Run tests
```

### Flask
```bash
python app.py                   # Dev server
pytest                          # Run tests
black .                         # Format code
```

---

**Happy Learning!** 🚀
