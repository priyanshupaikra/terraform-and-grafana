# Flask Complete Setup Guide

## Overview
Flask is a lightweight and flexible Python micro-framework for building web applications. It's minimal by design, allowing developers to choose their own tools and libraries.

---

## Folder Structure for Flask

```
flask-app/
├── README.md
├── wsgi.py                     # Entry point
├── requirements.txt
├── .env.example
├── .gitignore
├── pytest.ini
├── Dockerfile
├── docker-compose.yml
│
├── app/
│   ├── __init__.py             # App factory
│   ├── config.py               # Configuration
│   ├── extensions.py           # Flask extensions
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   └── item.py             # Item model
│   │
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── item.py             # Marshmallow schemas
│   │
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── health.py           # Health check blueprint
│   │   └── items.py            # Items blueprint
│   │
│   ├── services/
│   │   ├── __init__.py
│   │   └── item_service.py     # Business logic
│   │
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── logger.py           # Logging setup
│   │   ├── decorators.py       # Custom decorators
│   │   └── exceptions.py       # Custom exceptions
│   │
│   └── middleware/
│       ├── __init__.py
│       └── cors.py             # CORS setup
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py             # Pytest configuration
│   ├── test_main.py
│   └── routes/
│       ├── __init__.py
│       └── test_items.py
│
├── migrations/
│   └── versions/                # Alembic migrations
│
├── scripts/
│   └── init_db.py
│
├── logs/
│   └── .gitkeep
│
└── instance/                    # Instance folder (local config)
    └── .gitkeep
```

---

## Step 1: Project Initialization

### 1.1 Create Project Folder

```bash
mkdir flask-app
cd flask-app
git init

# Create virtual environment
python -m venv venv

# Activate (Linux/Mac)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate
```

### 1.2 Create requirements.txt

```txt
# Web Framework
Flask==3.0.0
Werkzeug==3.0.1
python-dotenv==1.0.0

# Database
Flask-SQLAlchemy==3.1.1
SQLAlchemy==2.0.23
alembic==1.13.0
psycopg2-binary==2.9.9

# Data Validation & Serialization
marshmallow==3.20.1
Flask-Marshmallow==0.15.0
marshmallow-sqlalchemy==0.30.0

# CORS
Flask-CORS==4.0.0

# API Documentation
flasgger==0.9.7.1

# Testing
pytest==7.4.3
pytest-flask==1.3.0
pytest-cov==4.1.0

# Code Quality
black==23.12.0
flake8==6.1.0
isort==5.13.2
mypy==1.7.1
pylint==3.0.3

# Logging
python-json-logger==2.0.7

# Database Fixtures
pytest-factoryboy==2.5.1

# WSGI Server
gunicorn==21.2.0

# Security
PyJWT==2.8.1
```

### 1.3 Install Dependencies

```bash
pip install -r requirements.txt
```

---

## Step 2: Create Folder Structure

```bash
# Create folders
mkdir -p app/{models,schemas,routes,services,utils,middleware}
mkdir -p tests/routes
mkdir -p migrations/versions
mkdir -p scripts
mkdir -p logs
mkdir instance

# Create __init__.py files
touch app/__init__.py
touch app/models/__init__.py
touch app/schemas/__init__.py
touch app/routes/__init__.py
touch app/services/__init__.py
touch app/utils/__init__.py
touch app/middleware/__init__.py
touch tests/__init__.py
touch tests/routes/__init__.py
```

---

## Step 3: Create Core Application Files

### 3.1 app/config.py

```python
"""Flask configuration."""
import os
from datetime import timedelta


class Config:
    """Base configuration."""
    
    # App
    SECRET_KEY = os.getenv('SECRET_KEY', 'dev-secret-key-change-in-production')
    DEBUG = False
    TESTING = False
    
    # Database
    SQLALCHEMY_DATABASE_URI = os.getenv(
        'DATABASE_URL',
        'sqlite:///app.db'
    )
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SQLALCHEMY_ECHO = False
    
    # Session
    PERMANENT_SESSION_LIFETIME = timedelta(days=7)
    SESSION_COOKIE_SECURE = False
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    
    # CORS
    CORS_ORIGINS = os.getenv(
        'CORS_ORIGINS',
        'http://localhost:3000,http://localhost:5000'
    ).split(',')
    
    # Pagination
    ITEMS_PER_PAGE = 10
    
    # Logging
    LOG_LEVEL = 'INFO'


class DevelopmentConfig(Config):
    """Development configuration."""
    
    DEBUG = True
    SQLALCHEMY_ECHO = True
    TESTING = False
    LOG_LEVEL = 'DEBUG'


class ProductionConfig(Config):
    """Production configuration."""
    
    DEBUG = False
    TESTING = False
    SESSION_COOKIE_SECURE = True
    
    # Database must be set via environment variable
    SQLALCHEMY_DATABASE_URI = os.getenv('DATABASE_URL')
    if not SQLALCHEMY_DATABASE_URI:
        raise ValueError("DATABASE_URL environment variable is required in production")


class TestingConfig(Config):
    """Testing configuration."""
    
    TESTING = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///:memory:'
    WTF_CSRF_ENABLED = False


# Configuration dictionary
config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'testing': TestingConfig,
    'default': DevelopmentConfig
}


def get_config():
    """Get configuration based on environment."""
    env = os.getenv('FLASK_ENV', 'development')
    return config.get(env, config['default'])
```

### 3.2 app/extensions.py

```python
"""Flask extensions initialization."""
from flask_sqlalchemy import SQLAlchemy
from flask_marshmallow import Marshmallow
from flask_cors import CORS

# Initialize extensions
db = SQLAlchemy()
ma = Marshmallow()
cors = CORS()


def init_extensions(app):
    """Initialize all extensions with app."""
    db.init_app(app)
    ma.init_app(app)
    cors.init_app(
        app,
        resources={r"/api/*": {"origins": app.config.get('CORS_ORIGINS', ['*'])}},
        supports_credentials=True
    )
```

### 3.3 app/models/item.py

```python
"""Item model."""
from datetime import datetime
from app.extensions import db


class Item(db.Model):
    """Item database model."""
    
    __tablename__ = 'items'
    
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(255), nullable=False, index=True)
    description = db.Column(db.Text, nullable=True)
    price = db.Column(db.Float, nullable=False, default=0.0)
    tax = db.Column(db.Float, nullable=False, default=0.0)
    is_active = db.Column(db.Boolean, default=True, index=True)
    created_at = db.Column(db.DateTime, nullable=False, default=datetime.utcnow)
    updated_at = db.Column(
        db.DateTime,
        nullable=False,
        default=datetime.utcnow,
        onupdate=datetime.utcnow
    )
    
    def to_dict(self):
        """Convert model to dictionary."""
        return {
            'id': self.id,
            'name': self.name,
            'description': self.description,
            'price': self.price,
            'tax': self.tax,
            'is_active': self.is_active,
            'created_at': self.created_at.isoformat(),
            'updated_at': self.updated_at.isoformat(),
        }
    
    def total_price(self):
        """Calculate total price with tax."""
        return self.price + self.tax
    
    def __repr__(self):
        return f'<Item {self.name}>'
```

### 3.4 app/schemas/item.py

```python
"""Item Marshmallow schemas."""
from marshmallow import Schema, fields, validate, validates_schema, ValidationError
from datetime import datetime


class ItemSchema(Schema):
    """Item schema for serialization/deserialization."""
    
    id = fields.Int(dump_only=True)
    name = fields.Str(
        required=True,
        validate=validate.Length(min=1, max=255)
    )
    description = fields.Str(allow_none=True)
    price = fields.Float(required=True, validate=validate.Range(min=0))
    tax = fields.Float(required=True, validate=validate.Range(min=0))
    is_active = fields.Bool(dump_default=True)
    created_at = fields.DateTime(dump_only=True)
    updated_at = fields.DateTime(dump_only=True)
    
    @validates_schema
    def validate_data(self, data, **kwargs):
        """Validate schema data."""
        if 'price' in data and 'tax' in data:
            if data['price'] < 0 or data['tax'] < 0:
                raise ValidationError('Price and tax must be non-negative')


class ItemListSchema(Schema):
    """Simplified schema for item lists."""
    
    id = fields.Int()
    name = fields.Str()
    price = fields.Float()
    tax = fields.Float()
    is_active = fields.Bool()


class ItemCreateSchema(Schema):
    """Schema for creating items."""
    
    name = fields.Str(required=True, validate=validate.Length(min=1, max=255))
    description = fields.Str(allow_none=True)
    price = fields.Float(required=True, validate=validate.Range(min=0))
    tax = fields.Float(required=True, validate=validate.Range(min=0))


class ItemUpdateSchema(Schema):
    """Schema for updating items."""
    
    name = fields.Str(validate=validate.Length(min=1, max=255))
    description = fields.Str(allow_none=True)
    price = fields.Float(validate=validate.Range(min=0))
    tax = fields.Float(validate=validate.Range(min=0))


# Schema instances
item_schema = ItemSchema()
items_schema = ItemSchema(many=True)
item_list_schema = ItemListSchema()
items_list_schema = ItemListSchema(many=True)
item_create_schema = ItemCreateSchema()
item_update_schema = ItemUpdateSchema()
```

### 3.5 app/utils/exceptions.py

```python
"""Custom exceptions."""
from flask import jsonify


class APIException(Exception):
    """Base API exception."""
    
    def __init__(self, message, status_code=400, payload=None):
        super().__init__()
        self.message = message
        self.status_code = status_code
        self.payload = payload
    
    def to_dict(self):
        """Convert exception to dictionary."""
        rv = dict(self.payload or ())
        rv['message'] = self.message
        rv['success'] = False
        return rv


class ItemNotFound(APIException):
    """Item not found exception."""
    
    def __init__(self):
        super().__init__('Item not found', status_code=404)


class InvalidInput(APIException):
    """Invalid input exception."""
    
    def __init__(self, message='Invalid input'):
        super().__init__(message, status_code=422)


class Unauthorized(APIException):
    """Unauthorized exception."""
    
    def __init__(self, message='Unauthorized'):
        super().__init__(message, status_code=401)


class Forbidden(APIException):
    """Forbidden exception."""
    
    def __init__(self, message='Forbidden'):
        super().__init__(message, status_code=403)
```

### 3.6 app/utils/logger.py

```python
"""Logging configuration."""
import logging
import sys
from pythonjsonlogger import jsonlogger


def setup_logger(app):
    """Setup logging for Flask app."""
    
    if app.debug:
        log_level = logging.DEBUG
    else:
        log_level = logging.INFO
    
    # Console handler
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setLevel(log_level)
    
    # JSON formatter
    formatter = jsonlogger.JsonFormatter(
        '%(timestamp)s %(level)s %(name)s %(message)s'
    )
    console_handler.setFormatter(formatter)
    
    # Get root logger
    logger = logging.getLogger()
    logger.setLevel(log_level)
    logger.addHandler(console_handler)
    
    return logger
```

### 3.7 app/utils/decorators.py

```python
"""Custom decorators."""
from functools import wraps
from flask import request, jsonify
from app.utils.exceptions import APIException


def json_required(f):
    """Decorator to require JSON content type."""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if not request.is_json:
            return jsonify({'message': 'Request must be JSON'}), 400
        return f(*args, **kwargs)
    return decorated_function


def handle_exceptions(f):
    """Decorator to handle API exceptions."""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        try:
            return f(*args, **kwargs)
        except APIException as e:
            return jsonify(e.to_dict()), e.status_code
        except Exception as e:
            return jsonify({'message': 'Internal server error', 'success': False}), 500
    return decorated_function
```

### 3.8 app/services/item_service.py

```python
"""Item service - business logic."""
from app.extensions import db
from app.models.item import Item
from app.utils.exceptions import ItemNotFound


class ItemService:
    """Service for item operations."""
    
    @staticmethod
    def get_all_items(page=1, per_page=10, active_only=True):
        """Get all items with pagination."""
        query = Item.query
        
        if active_only:
            query = query.filter_by(is_active=True)
        
        pagination = query.paginate(page=page, per_page=per_page)
        return pagination
    
    @staticmethod
    def get_item_by_id(item_id):
        """Get item by ID."""
        item = Item.query.get(item_id)
        if not item:
            raise ItemNotFound()
        return item
    
    @staticmethod
    def create_item(data):
        """Create new item."""
        item = Item(
            name=data.get('name'),
            description=data.get('description'),
            price=data.get('price', 0),
            tax=data.get('tax', 0)
        )
        db.session.add(item)
        db.session.commit()
        return item
    
    @staticmethod
    def update_item(item_id, data):
        """Update item."""
        item = ItemService.get_item_by_id(item_id)
        
        for key, value in data.items():
            if hasattr(item, key) and value is not None:
                setattr(item, key, value)
        
        db.session.commit()
        return item
    
    @staticmethod
    def delete_item(item_id):
        """Delete item."""
        item = ItemService.get_item_by_id(item_id)
        db.session.delete(item)
        db.session.commit()
        return item
    
    @staticmethod
    def search_items(query_string, page=1, per_page=10):
        """Search items by name or description."""
        query = Item.query.filter(
            db.or_(
                Item.name.ilike(f'%{query_string}%'),
                Item.description.ilike(f'%{query_string}%')
            )
        )
        pagination = query.paginate(page=page, per_page=per_page)
        return pagination
```

### 3.9 app/routes/health.py

```python
"""Health check endpoints."""
from flask import Blueprint, jsonify, current_app

health_bp = Blueprint('health', __name__, url_prefix='/health')


@health_bp.route('', methods=['GET'])
def health_check():
    """Health check endpoint."""
    return jsonify({
        'status': 'healthy',
        'app': 'Flask Application',
        'version': '1.0.0',
        'environment': current_app.config.get('FLASK_ENV', 'development')
    }), 200
```

### 3.10 app/routes/items.py

```python
"""Item endpoints."""
from flask import Blueprint, request, jsonify
from marshmallow import ValidationError

from app.extensions import db
from app.schemas.item import (
    item_schema,
    items_schema,
    item_list_schema,
    items_list_schema,
    item_create_schema,
    item_update_schema
)
from app.services.item_service import ItemService
from app.utils.exceptions import APIException, ItemNotFound
from app.utils.decorators import json_required, handle_exceptions

items_bp = Blueprint('items', __name__, url_prefix='/api/items')


@items_bp.route('', methods=['GET'])
def get_items():
    """Get all items with pagination."""
    try:
        page = request.args.get('page', 1, type=int)
        per_page = request.args.get('per_page', 10, type=int)
        search = request.args.get('search', None)
        
        if search:
            pagination = ItemService.search_items(search, page, per_page)
        else:
            pagination = ItemService.get_all_items(page, per_page)
        
        return jsonify({
            'success': True,
            'data': items_list_schema.dump(pagination.items),
            'pagination': {
                'page': page,
                'per_page': per_page,
                'total': pagination.total,
                'pages': pagination.pages
            }
        }), 200
    
    except Exception as e:
        return jsonify({'success': False, 'message': str(e)}), 500


@items_bp.route('/<int:item_id>', methods=['GET'])
def get_item(item_id):
    """Get item by ID."""
    try:
        item = ItemService.get_item_by_id(item_id)
        return jsonify({
            'success': True,
            'data': item_schema.dump(item)
        }), 200
    except ItemNotFound as e:
        return jsonify(e.to_dict()), e.status_code


@items_bp.route('', methods=['POST'])
@json_required
def create_item():
    """Create new item."""
    try:
        # Validate input
        data = item_create_schema.load(request.get_json())
        
        # Create item
        item = ItemService.create_item(data)
        
        return jsonify({
            'success': True,
            'message': 'Item created successfully',
            'data': item_schema.dump(item)
        }), 201
    
    except ValidationError as e:
        return jsonify({
            'success': False,
            'message': 'Validation error',
            'errors': e.messages
        }), 422
    except Exception as e:
        return jsonify({
            'success': False,
            'message': str(e)
        }), 500


@items_bp.route('/<int:item_id>', methods=['PUT', 'PATCH'])
@json_required
def update_item(item_id):
    """Update item."""
    try:
        # Validate input
        data = item_update_schema.load(request.get_json(), partial=True)
        
        # Update item
        item = ItemService.update_item(item_id, data)
        
        return jsonify({
            'success': True,
            'message': 'Item updated successfully',
            'data': item_schema.dump(item)
        }), 200
    
    except ValidationError as e:
        return jsonify({
            'success': False,
            'message': 'Validation error',
            'errors': e.messages
        }), 422
    except ItemNotFound as e:
        return jsonify(e.to_dict()), e.status_code
    except Exception as e:
        return jsonify({
            'success': False,
            'message': str(e)
        }), 500


@items_bp.route('/<int:item_id>', methods=['DELETE'])
def delete_item(item_id):
    """Delete item."""
    try:
        ItemService.delete_item(item_id)
        return jsonify({
            'success': True,
            'message': 'Item deleted successfully'
        }), 204
    except ItemNotFound as e:
        return jsonify(e.to_dict()), e.status_code
    except Exception as e:
        return jsonify({
            'success': False,
            'message': str(e)
        }), 500


@items_bp.errorhandler(404)
def not_found(error):
    """Handle 404 errors."""
    return jsonify({
        'success': False,
        'message': 'Resource not found'
    }), 404
```

### 3.11 app/__init__.py

```python
"""Application factory."""
from flask import Flask
from app.config import get_config
from app.extensions import init_extensions, db
from app.utils.logger import setup_logger


def create_app(config=None):
    """Create and configure Flask app."""
    
    app = Flask(__name__, instance_relative_config=True)
    
    # Load configuration
    if config is None:
        config = get_config()
    app.config.from_object(config)
    
    # Initialize extensions
    init_extensions(app)
    
    # Setup logging
    setup_logger(app)
    
    # Register blueprints
    from app.routes import health, items
    app.register_blueprint(health.health_bp)
    app.register_blueprint(items.items_bp)
    
    # Root endpoint
    @app.route('/')
    def root():
        return {
            'message': 'Welcome to Flask API',
            'version': '1.0.0'
        }, 200
    
    # Error handlers
    @app.errorhandler(404)
    def not_found(error):
        return {'success': False, 'message': 'Not found'}, 404
    
    @app.errorhandler(500)
    def server_error(error):
        return {'success': False, 'message': 'Server error'}, 500
    
    # Database initialization
    with app.app_context():
        db.create_all()
    
    return app
```

---

## Step 4: WSGI Entry Point

### 4.1 wsgi.py

```python
"""WSGI entry point."""
import os
from app import create_app

app = create_app()

if __name__ == '__main__':
    app.run(
        host='0.0.0.0',
        port=int(os.getenv('PORT', 5000)),
        debug=os.getenv('FLASK_ENV') == 'development'
    )
```

---

## Step 5: Testing Setup

### 5.1 tests/conftest.py

```python
"""Pytest configuration."""
import pytest
from app import create_app
from app.config import TestingConfig
from app.extensions import db


@pytest.fixture(scope='function')
def app():
    """Create Flask app for testing."""
    app = create_app(TestingConfig)
    
    with app.app_context():
        db.create_all()
        yield app
        db.session.remove()
        db.drop_all()


@pytest.fixture
def client(app):
    """Create test client."""
    return app.test_client()


@pytest.fixture
def runner(app):
    """Create CLI runner."""
    return app.test_cli_runner()
```

### 5.2 tests/test_main.py

```python
"""Test main endpoints."""


def test_root(client):
    """Test root endpoint."""
    response = client.get('/')
    assert response.status_code == 200
    assert 'Welcome' in response.get_json()['message']


def test_health_check(client):
    """Test health check endpoint."""
    response = client.get('/health')
    assert response.status_code == 200
    data = response.get_json()
    assert data['status'] == 'healthy'
```

### 5.3 tests/routes/test_items.py

```python
"""Test item endpoints."""
import pytest
from app.models.item import Item
from app.extensions import db


@pytest.fixture
def sample_item(app):
    """Create sample item."""
    with app.app_context():
        item = Item(name='Test Item', price=99.99, tax=10.0)
        db.session.add(item)
        db.session.commit()
        return item.id


def test_create_item(client):
    """Test creating item."""
    response = client.post(
        '/api/items',
        json={
            'name': 'Test Item',
            'description': 'Test',
            'price': 99.99,
            'tax': 10.0
        }
    )
    assert response.status_code == 201
    data = response.get_json()
    assert data['success'] is True
    assert data['data']['name'] == 'Test Item'


def test_get_items(client):
    """Test getting items."""
    response = client.get('/api/items')
    assert response.status_code == 200
    data = response.get_json()
    assert data['success'] is True
    assert isinstance(data['data'], list)


def test_get_item(client, app, sample_item):
    """Test getting single item."""
    response = client.get(f'/api/items/{sample_item}')
    assert response.status_code == 200
    data = response.get_json()
    assert data['success'] is True


def test_update_item(client, app, sample_item):
    """Test updating item."""
    response = client.patch(
        f'/api/items/{sample_item}',
        json={'name': 'Updated Name'}
    )
    assert response.status_code == 200
    data = response.get_json()
    assert data['data']['name'] == 'Updated Name'


def test_delete_item(client, app, sample_item):
    """Test deleting item."""
    response = client.delete(f'/api/items/{sample_item}')
    assert response.status_code == 204
```

---

## Step 6: Configuration Files

### 6.1 .env.example

```
FLASK_ENV=development
FLASK_DEBUG=True
SECRET_KEY=your-secret-key-here

DATABASE_URL=sqlite:///app.db

CORS_ORIGINS=http://localhost:3000,http://localhost:5000

LOG_LEVEL=DEBUG

PORT=5000
```

### 6.2 requirements.txt (verified)

```
Flask==3.0.0
Werkzeug==3.0.1
python-dotenv==1.0.0

Flask-SQLAlchemy==3.1.1
SQLAlchemy==2.0.23
alembic==1.13.0
psycopg2-binary==2.9.9

marshmallow==3.20.1
Flask-Marshmallow==0.15.0
marshmallow-sqlalchemy==0.30.0

Flask-CORS==4.0.0

flasgger==0.9.7.1

pytest==7.4.3
pytest-flask==1.3.0
pytest-cov==4.1.0

black==23.12.0
flake8==6.1.0
isort==5.13.2
mypy==1.7.1
pylint==3.0.3

python-json-logger==2.0.7

pytest-factoryboy==2.5.1

gunicorn==21.2.0

PyJWT==2.8.1
```

### 6.3 pytest.ini

```ini
[pytest]
testpaths = tests
python_files = tests.py test_*.py *_tests.py
addopts = --cov=app --cov-report=html --cov-report=term
```

### 6.4 Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["gunicorn", "wsgi:app", "--bind", "0.0.0.0:5000"]
```

### 6.5 docker-compose.yml

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: flask_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  flask:
    build: .
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=development
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/flask_db
      - FLASK_DEBUG=True
    volumes:
      - .:/app
    command: python wsgi.py
    depends_on:
      - postgres

volumes:
  postgres_data:
```

### 6.6 .gitignore

```
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

.venv
venv/
env/
.env
.env.local

.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store

.pytest_cache/
.coverage
htmlcov/

*.db
*.sqlite
*.sqlite3

instance/
logs/
```

---

## Running Flask

### Development Mode

```bash
# Activate virtual environment
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Copy .env file
cp .env.example .env

# Initialize database
python wsgi.py

# Run development server
python wsgi.py

# Or use Flask CLI
export FLASK_APP=wsgi.py
export FLASK_ENV=development
flask run
```

### Access Application

- API: http://localhost:5000
- API Docs (Swagger): http://localhost:5000/apidocs/
- Health: http://localhost:5000/health

### Running Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=app

# Run specific test file
pytest tests/routes/test_items.py

# Run with verbose output
pytest -v
```

### Code Quality

```bash
# Format code
black .

# Check linting
flake8 .

# Sort imports
isort .

# Type checking
mypy app

# Combined
black . && isort . && flake8 . && mypy app
```

### Database Migrations (Alembic)

```bash
# Initialize migrations
alembic init migrations

# Create migration
alembic revision --autogenerate -m "Add items table"

# Apply migration
alembic upgrade head

# Rollback migration
alembic downgrade -1
```

---

## Best Practices

✅ **Use blueprints** to organize code  
✅ **Separate concerns** - models, schemas, routes, services  
✅ **Use service layer** for business logic  
✅ **Validate input** with Marshmallow  
✅ **Handle exceptions** gracefully  
✅ **Implement logging** for debugging  
✅ **Test thoroughly** with pytest  
✅ **Use environment variables** for configuration  
✅ **Use dependency injection** patterns  
✅ **Document endpoints** with docstrings  

---

## Useful Commands Summary

```bash
# Setup
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Development
python wsgi.py
flask run

# Testing
pytest
pytest --cov

# Code Quality
black .
flake8 .
isort .
mypy app

# Database
alembic init migrations
alembic revision --autogenerate -m "message"
alembic upgrade head

# Docker
docker-compose up -d
docker-compose down
```

---

This Flask setup is production-ready with proper structure, testing, and best practices!
