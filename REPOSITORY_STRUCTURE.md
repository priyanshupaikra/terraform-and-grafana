# Complete Repository Structure Guide

This guide shows you how to organize your Terraform, Grafana, and Python web frameworks projects in a professional, scalable way.

## Final Repository Structure

```
terraform-and-grafana/
│
├── README.md                           # Main project readme
├── .gitignore
├── .env.example
├── LICENSE
│
├── docs/                               # Documentation folder
│   ├── SETUP.md
│   ├── DEPLOYMENT.md
│   └── TROUBLESHOOTING.md
│
├── terraform/                          # Infrastructure as Code
│   ├── README.md
│   ├── .gitignore
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── provider.tf
│   ├── backend.tf
│   ├── terraform.tfvars.example
│   ├── environments/
│   │   ├── dev/
│   │   │   ├── terraform.tfvars
│   │   │   └── backend.tf
│   │   ├── staging/
│   │   │   ├── terraform.tfvars
│   │   │   └── backend.tf
│   │   └── prod/
│   │       ├── terraform.tfvars
│   │       └── backend.tf
│   └── modules/
│       ├── vpc/
│       │   ├── main.tf
│       │   ├── variables.tf
│       │   └── outputs.tf
│       ├── rds/
│       │   ├── main.tf
│       │   ├── variables.tf
│       │   └── outputs.tf
│       ├── ec2/
│       │   ├── main.tf
│       │   ├── variables.tf
│       │   └── outputs.tf
│       └── security/
│           ├── main.tf
│           ├── variables.tf
│           └── outputs.tf
│
├── grafana/                            # Monitoring & Visualization
│   ├── README.md
│   ├── docker-compose.yml
│   ├── .env.example
│   ├── dashboards/
│   │   ├── system-monitoring.json
│   │   ├── application-metrics.json
│   │   └── infrastructure-overview.json
│   ├── alerts/
│   │   └── alert-rules.yaml
│   ├── provisioning/
│   │   ├── datasources/
│   │   │   └── prometheus.yml
│   │   └── dashboards/
│   │       └── dashboard-provider.yml
│   └── scripts/
│       └── setup-grafana.sh
│
├── fastapi-app/                        # FastAPI Project
│   ├── README.md
│   ├── requirements.txt
│   ├── .env.example
│   ├── .gitignore
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── pytest.ini
│   ├── main.py                         # Entry point
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py                     # App factory
│   │   ├── config.py                   # Configuration
│   │   ├── database.py                 # Database setup
│   │   ├── dependencies.py             # Dependency injection
│   │   │
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── base.py                 # Base model
│   │   │   └── item.py                 # Item model
│   │   │
│   │   ├── schemas/
│   │   │   ├── __init__.py
│   │   │   ├── item.py                 # Item schemas
│   │   │   └── response.py             # Response schemas
│   │   │
│   │   ├── routes/
│   │   │   ├── __init__.py
│   │   │   └── items.py                # Item endpoints
│   │   │
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   └── item_service.py         # Business logic
│   │   │
│   │   ├── utils/
│   │   │   ├── __init__.py
│   │   │   ├── logger.py               # Logging
│   │   │   └── exceptions.py           # Custom exceptions
│   │   │
│   │   └── middleware/
│   │       ├── __init__.py
│   │       └── cors.py                 # CORS middleware
│   │
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── conftest.py                 # Pytest fixtures
│   │   ├── test_main.py
│   │   └── routes/
│   │       ├── __init__.py
│   │       └── test_items.py
│   │
│   ├── scripts/
│   │   └── init_db.py
│   │
│   └── alembic/                        # Database migrations (optional)
│       ├── versions/
│       ├── env.py
│       └── script.py.mako
│
├── django-app/                         # Django Project
│   ├── README.md
│   ├── requirements.txt
│   ├── .env.example
│   ├── .gitignore
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── pytest.ini
│   ├── manage.py
│   │
│   ├── config/                         # Project settings
│   │   ├── __init__.py
│   │   ├── settings.py
│   │   ├── settings/
│   │   │   ├── base.py                 # Base settings
│   │   │   ├── development.py
│   │   │   ├── production.py
│   │   │   └── testing.py
│   │   ├── urls.py
│   │   ├── asgi.py
│   │   └── wsgi.py
│   │
│   ├── apps/
│   │   ├── __init__.py
│   │   │
│   │   ├── items/
│   │   │   ├── migrations/
│   │   │   │   └── __init__.py
│   │   │   ├── __init__.py
│   │   │   ├── admin.py
│   │   │   ├── apps.py
│   │   │   ├── models.py
│   │   │   ├── views.py
│   │   │   ├── serializers.py
│   │   │   ├── urls.py
│   │   │   ├── filters.py
│   │   │   ├── permissions.py
│   │   │   └── tests.py
│   │   │
│   │   ├── users/
│   │   │   ├── migrations/
│   │   │   │   └── __init__.py
│   │   │   ├── __init__.py
│   │   │   ├── admin.py
│   │   │   ├── apps.py
│   │   │   ├── models.py
│   │   │   ├── views.py
│   │   │   ├── serializers.py
│   │   │   ├── urls.py
│   │   │   └── tests.py
│   │   │
│   │   └── api/
│   │       ├── __init__.py
│   │       ├── pagination.py
│   │       ├── renderers.py
│   │       └── versioning.py
│   │
│   ├── static/
│   │   └── .gitkeep
│   │
│   ├── media/
│   │   └── .gitkeep
│   │
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── conftest.py
│   │   ├── factories.py
│   │   └── test_api.py
│   │
│   ├── scripts/
│   │   ├── manage_db.py
│   │   └── seed_data.py
│   │
│   └── logs/
│       └── .gitkeep
│
├── flask-app/                          # Flask Project
│   ├── README.md
│   ├── requirements.txt
│   ├── .env.example
│   ├── .gitignore
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── pytest.ini
│   ├── wsgi.py                         # Entry point
│   │
│   ├── app/
│   │   ├── __init__.py                 # App factory
│   │   ├── config.py                   # Configuration
│   │   │
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   └── item.py
│   │   │
│   │   ├── schemas/
│   │   │   ├── __init__.py
│   │   │   └── item.py
│   │   │
│   │   ├── routes/
│   │   │   ├── __init__.py
│   │   │   ├── items.py
│   │   │   └── health.py
│   │   │
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   └── item_service.py
│   │   │
│   │   ├── utils/
│   │   │   ├── __init__.py
│   │   │   ├── logger.py
│   │   │   └── decorators.py
│   │   │
│   │   └── extensions.py                # Flask extensions setup
│   │
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── conftest.py
│   │   ├── test_main.py
│   │   └── routes/
│   │       ├── __init__.py
│   │       └── test_items.py
│   │
│   ├── migrations/
│   │   └── versions/
│   │
│   ├── logs/
│   │   └── .gitkeep
│   │
│   └── scripts/
│       └── init_db.py
│
└── scripts/                            # Root level scripts
    ├── setup.sh                        # Initial setup
    ├── docker-up.sh                    # Start all containers
    ├── docker-down.sh                  # Stop all containers
    ├── test-all.sh                     # Run all tests
    └── deploy.sh                       # Deployment script
```

---

## Step-by-Step Setup Instructions

### Step 1: Create Root Directory

```bash
# Create root directory
mkdir terraform-and-grafana
cd terraform-and-grafana

# Initialize git
git init
```

### Step 2: Create Individual Project Folders

```bash
# Create main folders
mkdir terraform grafana fastapi-app django-app flask-app docs scripts

# Create subfolders for each framework
cd fastapi-app
mkdir app tests scripts
mkdir app/models app/schemas app/routes app/services app/utils app/middleware
cd ..

cd django-app
mkdir config apps static media tests scripts logs
mkdir apps/items apps/items/migrations
mkdir apps/users apps/users/migrations
cd ..

cd flask-app
mkdir app tests migrations logs scripts
mkdir app/models app/schemas app/routes app/services app/utils
cd ..
```

### Step 3: Create .gitignore File

Create root `./.gitignore`:

```
# Python
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
MANIFEST

# Virtual environments
.venv
venv/
env/
.env
.env.local
.env.*.local

# IDEs
.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store
.project
.pydevproject
.settings/
*.sublime-project
*.sublime-workspace

# Testing
.pytest_cache/
.coverage
htmlcov/
.tox/
.hypothesis/

# Logs
*.log
logs/

# Database
*.db
*.sqlite
*.sqlite3

# Terraform
.terraform/
.terraform.lock.hcl
terraform.tfstate
terraform.tfstate.backup
*.tfvars
!terraform.tfvars.example
crash.log

# Grafana
grafana/data/
grafana/logs/

# Docker
docker-compose.override.yml

# Environment files
.env
.env.local
.env.*.local

# OS
.DS_Store
Thumbs.db
```

### Step 4: Create Individual README Files

#### FastAPI README:

Create `fastapi-app/README.md`:

```markdown
# FastAPI Application

A modern, fast, and high-performance API built with FastAPI.

## Setup

```bash
cd fastapi-app
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
```

## Run Development Server

```bash
uvicorn main:app --reload
```

## Run Tests

```bash
pytest
```

## Project Structure

- `app/` - Application code
  - `models/` - SQLAlchemy models
  - `schemas/` - Pydantic schemas
  - `routes/` - API endpoints
  - `services/` - Business logic
  - `utils/` - Helper functions
- `tests/` - Test files
- `scripts/` - Utility scripts
```

#### Django README:

Create `django-app/README.md`:

```markdown
# Django Application

A full-featured Django REST API application.

## Setup

```bash
cd django-app
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
python manage.py migrate
python manage.py createsuperuser
```

## Run Development Server

```bash
python manage.py runserver
```

## Run Tests

```bash
pytest
```

## Project Structure

- `config/` - Project settings
- `apps/` - Django applications
  - `items/` - Items app
  - `users/` - Users app
- `tests/` - Test files
- `scripts/` - Utility scripts
```

#### Flask README:

Create `flask-app/README.md`:

```markdown
# Flask Application

A lightweight and flexible Flask API application.

## Setup

```bash
cd flask-app
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
```

## Run Development Server

```bash
python wsgi.py
```

## Run Tests

```bash
pytest
```

## Project Structure

- `app/` - Application code
  - `models/` - Database models
  - `schemas/` - Marshmallow schemas
  - `routes/` - Blueprints and endpoints
  - `services/` - Business logic
  - `utils/` - Helper functions
- `tests/` - Test files
- `scripts/` - Utility scripts
```

### Step 5: Create requirements.txt Files

#### FastAPI requirements.txt:

Create `fastapi-app/requirements.txt`:

```
# Web Framework
fastapi==0.104.1
uvicorn[standard]==0.24.0
starlette==0.27.0

# Database
sqlalchemy==2.0.23
alembic==1.13.0

# Data Validation
pydantic==2.5.0
pydantic-settings==2.1.0

# Environment Variables
python-dotenv==1.0.0

# API Documentation
python-multipart==0.0.6

# Async
httpx==0.25.0

# Testing
pytest==7.4.3
pytest-asyncio==0.21.1
pytest-cov==4.1.0

# Code Quality
black==23.12.0
flake8==6.1.0
isort==5.13.2
mypy==1.7.1

# Logging
python-json-logger==2.0.7

# Security
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
```

#### Django requirements.txt:

Create `django-app/requirements.txt`:

```
# Framework
Django==4.2.7
djangorestframework==3.14.0
django-cors-headers==4.3.1
django-filter==23.4

# Database
psycopg2-binary==2.9.9
sqlparse==0.4.4

# Environment Variables
python-dotenv==1.0.0

# Database Migrations
alembic==1.13.0

# Testing
pytest==7.4.3
pytest-django==4.7.0
pytest-cov==4.1.0
factory-boy==3.3.0

# Code Quality
black==23.12.0
flake8==6.1.0
isort==5.13.2
mypy==1.7.1

# API
djangorestframework-simplejwt==5.3.2

# Logging
python-json-logger==2.0.7

# Admin
django-admin-interface==0.25.2
```

#### Flask requirements.txt:

Create `flask-app/requirements.txt`:

```
# Web Framework
Flask==3.0.0
Werkzeug==3.0.1

# Database
Flask-SQLAlchemy==3.1.1
SQLAlchemy==2.0.23
alembic==1.13.0

# Data Validation
marshmallow==3.20.1
Flask-Marshmallow==0.15.0

# Environment Variables
python-dotenv==1.0.0

# CORS
Flask-CORS==4.0.0

# Testing
pytest==7.4.3
pytest-flask==1.3.0
pytest-cov==4.1.0

# Code Quality
black==23.12.0
flake8==6.1.0
isort==5.13.2
mypy==1.7.1

# Logging
python-json-logger==2.0.7

# Database Fixtures
pytest-factoryboy==2.5.1
```

### Step 6: Create Environment Example Files

#### FastAPI `.env.example`:

```
# FastAPI Environment Variables
ENVIRONMENT=development
DEBUG=True
SECRET_KEY=your-secret-key-here

# Database
DATABASE_URL=sqlite:///./test.db

# API
API_TITLE=FastAPI Application
API_VERSION=1.0.0
API_DESCRIPTION=A FastAPI application

# CORS
CORS_ORIGINS=["http://localhost:3000", "http://localhost:8000"]
```

#### Django `.env.example`:

```
# Django Environment Variables
ENVIRONMENT=development
DEBUG=True
SECRET_KEY=your-secret-key-here

# Database
DB_ENGINE=django.db.backends.postgresql
DB_NAME=mydb
DB_USER=postgres
DB_PASSWORD=password
DB_HOST=localhost
DB_PORT=5432

# Email
EMAIL_BACKEND=django.core.mail.backends.console.EmailBackend

# Allowed Hosts
ALLOWED_HOSTS=localhost,127.0.0.1

# CORS
CORS_ALLOWED_ORIGINS=http://localhost:3000,http://localhost:8000
```

#### Flask `.env.example`:

```
# Flask Environment Variables
FLASK_ENV=development
FLASK_DEBUG=True
SECRET_KEY=your-secret-key-here

# Database
DATABASE_URL=sqlite:///app.db

# API
API_TITLE=Flask Application
API_VERSION=1.0.0

# CORS
CORS_ORIGINS=http://localhost:3000,http://localhost:8000
```

### Step 7: Create Docker Files

#### FastAPI `docker-compose.yml`:

```yaml
version: '3.8'

services:
  fastapi:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=sqlite:///./test.db
    volumes:
      - .:/app
    command: uvicorn main:app --host 0.0.0.0 --reload
```

#### Django `docker-compose.yml`:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  django:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DB_HOST=postgres
    volumes:
      - .:/app
    command: python manage.py runserver 0.0.0.0:8000
    depends_on:
      - postgres

volumes:
  postgres_data:
```

#### Flask `docker-compose.yml`:

```yaml
version: '3.8'

services:
  flask:
    build: .
    ports:
      - "5000:5000"
    environment:
      - FLASK_ENV=development
    volumes:
      - .:/app
    command: python wsgi.py
```

---

## How to Set Up from Scratch

### 1. Clone and Initialize

```bash
git clone <your-repo>
cd terraform-and-grafana
git config user.email "your.email@example.com"
git config user.name "Your Name"
```

### 2. Set Up Each Framework

```bash
# FastAPI
cd fastapi-app
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
cd ..

# Django
cd django-app
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
python manage.py migrate
cd ..

# Flask
cd flask-app
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
cd ..
```

### 3. Run All Services with Docker

```bash
# FastAPI
cd fastapi-app && docker-compose up -d

# Django
cd ../django-app && docker-compose up -d

# Flask
cd ../flask-app && docker-compose up -d

# Grafana
cd ../grafana && docker-compose up -d
```

### 4. Test All Applications

```bash
# FastAPI
cd fastapi-app && pytest

# Django
cd ../django-app && pytest

# Flask
cd ../flask-app && pytest
```

---

## Best Practices for Each Framework

### FastAPI
✅ Use async/await for I/O operations  
✅ Leverage Pydantic for validation  
✅ Organize by features (routes, models, schemas)  
✅ Use dependency injection  
✅ Implement proper error handling  

### Django
✅ Use apps to organize features  
✅ Keep business logic in services  
✅ Use Django ORM efficiently  
✅ Implement proper permissions  
✅ Use signals sparingly  

### Flask
✅ Use blueprints for organization  
✅ Keep views lightweight  
✅ Use extensions properly  
✅ Implement middleware for common tasks  
✅ Keep app factory pattern  

---

## Git Workflow

```bash
# Create feature branch
git checkout -b feature/add-user-endpoints

# Make changes
git add .
git commit -m "feat: add user endpoints"

# Push to remote
git push origin feature/add-user-endpoints

# Create Pull Request
```

---

## Common Commands

```bash
# Setup all projects
bash scripts/setup.sh

# Start all services
bash scripts/docker-up.sh

# Stop all services
bash scripts/docker-down.sh

# Run all tests
bash scripts/test-all.sh

# Deploy
bash scripts/deploy.sh
```

---

This structure provides:
- ✅ Clear separation of concerns
- ✅ Easy scalability
- ✅ Professional organization
- ✅ Simple docker integration
- ✅ Testing infrastructure
- ✅ Environment management
- ✅ Documentation
