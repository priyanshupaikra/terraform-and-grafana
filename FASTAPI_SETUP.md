# FastAPI Complete Setup Guide

## Overview
FastAPI is a modern, fast (high-performance), web framework for building APIs with Python 3.7+ based on standard Python type hints.

---

## Folder Structure for FastAPI

```
fastapi-app/
├── README.md
├── requirements.txt
├── .env.example
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── pytest.ini
├── main.py                    # Entry point
├── app/
│   ├── __init__.py
│   ├── main.py               # App factory
│   ├── config.py             # Settings
│   ├── database.py           # Database connection
│   ├── dependencies.py       # Dependency injection
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   ├── base.py           # Base model
│   │   └── item.py           # Item model
│   │
│   ├── schemas/              # Pydantic models
│   │   ├── __init__.py
│   │   ├── item.py
│   │   └── response.py
│   │
│   ├── routes/               # API endpoints
│   │   ├── __init__.py
│   │   ├── health.py
│   │   └── items.py
│   │
│   ├── services/             # Business logic
│   │   ├── __init__.py
│   │   └── item_service.py
│   │
│   ├── utils/                # Helper functions
│   │   ├── __init__.py
│   │   ├── logger.py
│   │   └── exceptions.py
│   │
│   └── middleware/
│       ├── __init__.py
│       └── cors.py
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py           # Pytest config
│   ├── test_main.py
│   └── routes/
│       └── test_items.py
│
├── scripts/
│   └── init_db.py
│
└── alembic/                  # Database migrations
    ├── versions/
    ├── env.py
    └── script.py.mako
```

---

## Step 1: Project Initialization

### 1.1 Create Project Folder

```bash
mkdir fastapi-app
cd fastapi-app
git init
```

### 1.2 Create Virtual Environment

```bash
# Create virtual environment
python -m venv venv

# Activate (Linux/Mac)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate

# Verify activation
python --version  # Should show your Python version
```

### 1.3 Create requirements.txt

```txt
# Web Framework
fastapi==0.104.1
uvicorn[standard]==0.24.0
starlette==0.27.0

# Database
sqlalchemy==2.0.23
alembic==1.13.0
psycopg2-binary==2.9.9

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
httpx==0.25.0

# Code Quality
black==23.12.0
flake8==6.1.0
isort==5.13.2
mypy==1.7.1
pylint==3.0.3

# Logging
python-json-logger==2.0.7

# Security
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
bcrypt==4.1.1

# Monitoring
opentelemetry-api==1.21.0
opentelemetry-sdk==1.21.0
```

### 1.4 Install Dependencies

```bash
pip install -r requirements.txt
```

---

## Step 2: Create Folder Structure

```bash
# Create main app folder structure
mkdir -p app/{models,schemas,routes,services,utils,middleware}
mkdir -p tests/routes
mkdir scripts
mkdir alembic/versions

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
touch scripts/__init__.py
```

---

## Step 3: Create Core Application Files

### 3.1 app/config.py

```python
"""Application configuration."""
import os
from functools import lru_cache
from typing import Optional

from pydantic_settings import BaseSettings


class Settings(BaseSettings):
    """Application settings."""
    
    # App
    APP_NAME: str = "FastAPI Application"
    APP_VERSION: str = "1.0.0"
    DEBUG: bool = os.getenv("DEBUG", "False").lower() == "true"
    ENVIRONMENT: str = os.getenv("ENVIRONMENT", "development")
    
    # Database
    DATABASE_URL: str = os.getenv(
        "DATABASE_URL",
        "sqlite:///./test.db"
    )
    
    # Security
    SECRET_KEY: str = os.getenv(
        "SECRET_KEY",
        "your-secret-key-change-in-production"
    )
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    
    # CORS
    CORS_ORIGINS: list = [
        "http://localhost:3000",
        "http://localhost:8000",
        "http://127.0.0.1:3000",
        "http://127.0.0.1:8000",
    ]
    CORS_ALLOW_CREDENTIALS: bool = True
    CORS_ALLOW_METHODS: list = ["*"]
    CORS_ALLOW_HEADERS: list = ["*"]
    
    # Logging
    LOG_LEVEL: str = "INFO"
    
    class Config:
        env_file = ".env"
        case_sensitive = True


@lru_cache()
def get_settings() -> Settings:
    """Get cached settings instance."""
    return Settings()
```

### 3.2 app/database.py

```python
"""Database configuration and session management."""
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from typing import Generator

from app.config import get_settings

settings = get_settings()

# Create engine
engine = create_engine(
    settings.DATABASE_URL,
    connect_args={"check_same_thread": False} if "sqlite" in settings.DATABASE_URL else {},
    echo=settings.DEBUG,
)

# Create session factory
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Create base class for models
Base = declarative_base()


def get_db() -> Generator[Session, None, None]:
    """Get database session."""
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


def init_db():
    """Initialize database tables."""
    Base.metadata.create_all(bind=engine)
```

### 3.3 app/dependencies.py

```python
"""Dependency injection setup."""
from sqlalchemy.orm import Session
from app.database import get_db

# Database dependency
async def get_database() -> Session:
    """Get database session."""
    async for db in get_db():
        yield db
```

### 3.4 app/models/base.py

```python
"""Base model for all database models."""
from datetime import datetime
from sqlalchemy import Column, Integer, DateTime
from app.database import Base


class BaseModel(Base):
    """Base model with common fields."""
    
    __abstract__ = True
    
    id = Column(Integer, primary_key=True, index=True)
    created_at = Column(DateTime, default=datetime.utcnow, nullable=False)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow, nullable=False)
```

### 3.5 app/models/item.py

```python
"""Item model."""
from sqlalchemy import Column, String, Float, Text
from app.models.base import BaseModel


class Item(BaseModel):
    """Item database model."""
    
    __tablename__ = "items"
    
    name = Column(String(255), nullable=False, index=True)
    description = Column(Text, nullable=True)
    price = Column(Float, nullable=False, default=0.0)
    tax = Column(Float, nullable=False, default=0.0)
```

### 3.6 app/schemas/item.py

```python
"""Item Pydantic schemas."""
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime


class ItemBase(BaseModel):
    """Base item schema."""
    
    name: str = Field(..., min_length=1, max_length=255)
    description: Optional[str] = None
    price: float = Field(..., ge=0)
    tax: float = Field(default=0.0, ge=0)


class ItemCreate(ItemBase):
    """Item creation schema."""
    pass


class ItemUpdate(BaseModel):
    """Item update schema."""
    
    name: Optional[str] = Field(None, min_length=1, max_length=255)
    description: Optional[str] = None
    price: Optional[float] = Field(None, ge=0)
    tax: Optional[float] = Field(None, ge=0)


class Item(ItemBase):
    """Item response schema."""
    
    id: int
    created_at: datetime
    updated_at: datetime
    
    class Config:
        from_attributes = True
```

### 3.7 app/schemas/response.py

```python
"""Response schemas."""
from typing import Generic, TypeVar, Optional
from pydantic import BaseModel

T = TypeVar('T')


class SuccessResponse(BaseModel, Generic[T]):
    """Generic success response."""
    
    success: bool = True
    message: str = "Success"
    data: Optional[T] = None
    

class ErrorResponse(BaseModel):
    """Error response."""
    
    success: bool = False
    message: str
    details: Optional[dict] = None
```

### 3.8 app/utils/exceptions.py

```python
"""Custom exceptions."""
from fastapi import HTTPException, status


class AppException(HTTPException):
    """Base application exception."""
    
    def __init__(
        self,
        status_code: int = status.HTTP_400_BAD_REQUEST,
        detail: str = "An error occurred",
    ):
        super().__init__(status_code=status_code, detail=detail)


class ItemNotFound(AppException):
    """Item not found exception."""
    
    def __init__(self):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Item not found"
        )


class InvalidInput(AppException):
    """Invalid input exception."""
    
    def __init__(self, detail: str = "Invalid input"):
        super().__init__(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
            detail=detail
        )
```

### 3.9 app/utils/logger.py

```python
"""Logging configuration."""
import logging
import sys
from pythonjsonlogger import jsonlogger
from app.config import get_settings

settings = get_settings()


def setup_logger(name: str) -> logging.Logger:
    """Setup logger with JSON formatting."""
    
    logger = logging.getLogger(name)
    logger.setLevel(settings.LOG_LEVEL)
    
    # Console handler
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setLevel(settings.LOG_LEVEL)
    
    # JSON formatter
    formatter = jsonlogger.JsonFormatter(
        '%(timestamp)s %(level)s %(name)s %(message)s'
    )
    console_handler.setFormatter(formatter)
    
    logger.addHandler(console_handler)
    
    return logger


logger = setup_logger(__name__)
```

### 3.10 app/services/item_service.py

```python
"""Item service - business logic."""
from sqlalchemy.orm import Session
from typing import List, Optional
from app.models.item import Item
from app.schemas.item import ItemCreate, ItemUpdate
from app.utils.exceptions import ItemNotFound


class ItemService:
    """Service for item operations."""
    
    @staticmethod
    def get_item(db: Session, item_id: int) -> Item:
        """Get item by ID."""
        item = db.query(Item).filter(Item.id == item_id).first()
        if not item:
            raise ItemNotFound()
        return item
    
    @staticmethod
    def get_items(db: Session, skip: int = 0, limit: int = 10) -> List[Item]:
        """Get all items with pagination."""
        return db.query(Item).offset(skip).limit(limit).all()
    
    @staticmethod
    def create_item(db: Session, item: ItemCreate) -> Item:
        """Create new item."""
        db_item = Item(**item.model_dump())
        db.add(db_item)
        db.commit()
        db.refresh(db_item)
        return db_item
    
    @staticmethod
    def update_item(db: Session, item_id: int, item_update: ItemUpdate) -> Item:
        """Update item."""
        db_item = ItemService.get_item(db, item_id)
        
        update_data = item_update.model_dump(exclude_unset=True)
        for field, value in update_data.items():
            setattr(db_item, field, value)
        
        db.add(db_item)
        db.commit()
        db.refresh(db_item)
        return db_item
    
    @staticmethod
    def delete_item(db: Session, item_id: int) -> Item:
        """Delete item."""
        db_item = ItemService.get_item(db, item_id)
        db.delete(db_item)
        db.commit()
        return db_item
```

### 3.11 app/routes/health.py

```python
"""Health check endpoints."""
from fastapi import APIRouter
from app.config import get_settings

router = APIRouter(
    prefix="/health",
    tags=["health"]
)

settings = get_settings()


@router.get("")
async def health_check():
    """Health check endpoint."""
    return {
        "status": "healthy",
        "app": settings.APP_NAME,
        "version": settings.APP_VERSION,
        "environment": settings.ENVIRONMENT,
    }
```

### 3.12 app/routes/items.py

```python
"""Item endpoints."""
from fastapi import APIRouter, Depends, status
from sqlalchemy.orm import Session
from typing import List

from app.database import get_db
from app.schemas.item import Item, ItemCreate, ItemUpdate
from app.services.item_service import ItemService

router = APIRouter(
    prefix="/items",
    tags=["items"]
)


@router.get("", response_model=List[Item])
async def list_items(
    skip: int = 0,
    limit: int = 10,
    db: Session = Depends(get_db)
):
    """Get all items."""
    return ItemService.get_items(db, skip=skip, limit=limit)


@router.get("/{item_id}", response_model=Item)
async def get_item(
    item_id: int,
    db: Session = Depends(get_db)
):
    """Get item by ID."""
    return ItemService.get_item(db, item_id)


@router.post("", response_model=Item, status_code=status.HTTP_201_CREATED)
async def create_item(
    item: ItemCreate,
    db: Session = Depends(get_db)
):
    """Create new item."""
    return ItemService.create_item(db, item)


@router.put("/{item_id}", response_model=Item)
async def update_item(
    item_id: int,
    item_update: ItemUpdate,
    db: Session = Depends(get_db)
):
    """Update item."""
    return ItemService.update_item(db, item_id, item_update)


@router.delete("/{item_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_item(
    item_id: int,
    db: Session = Depends(get_db)
):
    """Delete item."""
    ItemService.delete_item(db, item_id)
```

### 3.13 app/middleware/cors.py

```python
"""CORS middleware configuration."""
from fastapi.middleware.cors import CORSMiddleware
from app.config import get_settings

settings = get_settings()


def setup_cors(app):
    """Setup CORS middleware."""
    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.CORS_ORIGINS,
        allow_credentials=settings.CORS_ALLOW_CREDENTIALS,
        allow_methods=settings.CORS_ALLOW_METHODS,
        allow_headers=settings.CORS_ALLOW_HEADERS,
    )
```

### 3.14 app/main.py

```python
"""Application factory."""
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager

from app.config import get_settings
from app.database import init_db
from app.routes import health, items

settings = get_settings()


@asynccontextmanager
async def lifespan(app: FastAPI):
    """Lifespan context manager."""
    # Startup
    init_db()
    yield
    # Shutdown


def create_app() -> FastAPI:
    """Create and configure FastAPI app."""
    
    app = FastAPI(
        title=settings.APP_NAME,
        description="A FastAPI application",
        version=settings.APP_VERSION,
        lifespan=lifespan
    )
    
    # Add CORS middleware
    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.CORS_ORIGINS,
        allow_credentials=settings.CORS_ALLOW_CREDENTIALS,
        allow_methods=settings.CORS_ALLOW_METHODS,
        allow_headers=settings.CORS_ALLOW_HEADERS,
    )
    
    # Include routers
    app.include_router(health.router)
    app.include_router(items.router, prefix="/api")
    
    # Root endpoint
    @app.get("/")
    async def root():
        return {
            "message": f"Welcome to {settings.APP_NAME}",
            "version": settings.APP_VERSION
        }
    
    return app
```

### 3.15 main.py (Entry Point)

```python
"""Main entry point."""
import uvicorn
from app.main import create_app

app = create_app()

if __name__ == "__main__":
    uvicorn.run(
        app,
        host="0.0.0.0",
        port=8000,
        reload=True
    )
```

---

## Step 4: Testing Setup

### 4.1 tests/conftest.py

```python
"""Pytest configuration and fixtures."""
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session

from app.main import create_app
from app.database import Base, get_db


@pytest.fixture(scope="function")
def test_db():
    """Create test database."""
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(bind=engine)
    
    TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
    
    def override_get_db():
        try:
            db = TestingSessionLocal()
            yield db
        finally:
            db.close()
    
    app = create_app()
    app.dependency_overrides[get_db] = override_get_db
    
    yield TestingSessionLocal()
    
    Base.metadata.drop_all(bind=engine)


@pytest.fixture
def client(test_db):
    """Create test client."""
    from fastapi.testclient import TestClient
    
    app = create_app()
    return TestClient(app)
```

### 4.2 tests/test_main.py

```python
"""Test main endpoints."""
from fastapi.testclient import TestClient


def test_root(client: TestClient):
    """Test root endpoint."""
    response = client.get("/")
    assert response.status_code == 200
    assert "Welcome" in response.json()["message"]


def test_health_check(client: TestClient):
    """Test health check endpoint."""
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json()["status"] == "healthy"
```

### 4.3 tests/routes/test_items.py

```python
"""Test item endpoints."""
from fastapi.testclient import TestClient


def test_create_item(client: TestClient):
    """Test creating an item."""
    response = client.post(
        "/api/items",
        json={
            "name": "Test Item",
            "description": "A test item",
            "price": 99.99,
            "tax": 10.0
        }
    )
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "Test Item"
    assert data["price"] == 99.99


def test_get_items(client: TestClient):
    """Test listing items."""
    response = client.get("/api/items")
    assert response.status_code == 200
    assert isinstance(response.json(), list)


def test_get_item_not_found(client: TestClient):
    """Test getting non-existent item."""
    response = client.get("/api/items/999")
    assert response.status_code == 404
```

---

## Step 5: Configuration Files

### 5.1 .env.example

```
ENVIRONMENT=development
DEBUG=True
SECRET_KEY=your-secret-key-here

DATABASE_URL=sqlite:///./test.db

LOG_LEVEL=INFO
```

### 5.2 .gitignore

```
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
.venv

# IDEs
.vscode/
.idea/
*.swp

# Testing
.pytest_cache/
.coverage
htmlcov/

# Database
*.db
*.sqlite

# Environment
.env
.env.local
```

### 5.3 Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 5.4 docker-compose.yml

```yaml
version: '3.8'

services:
  fastapi:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=sqlite:///./test.db
      - DEBUG=True
    volumes:
      - .:/app
    command: uvicorn main:app --host 0.0.0.0 --reload

  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: fastapi_db
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

## Running the Application

### Development Mode

```bash
# Activate virtual environment
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Copy .env file
cp .env.example .env

# Run development server
python main.py

# Or use uvicorn directly
uvicorn main:app --reload
```

### Access API Documentation

- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc
- API: http://localhost:8000

### Run Tests

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

# Combined check
black . && isort . && flake8 . && mypy app
```

---

## Best Practices

✅ **Always use async functions** for I/O operations  
✅ **Validate input** with Pydantic models  
✅ **Use dependency injection** for database and other services  
✅ **Implement proper error handling** with custom exceptions  
✅ **Test thoroughly** with pytest  
✅ **Document endpoints** with docstrings  
✅ **Use type hints** everywhere  
✅ **Separate concerns** - models, schemas, routes, services  
✅ **Use environment variables** for configuration  
✅ **Implement logging** for debugging  

---

## Useful Commands Summary

```bash
# Setup
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Development
python main.py
uvicorn main:app --reload

# Testing
pytest
pytest --cov

# Code Quality
black .
flake8 .
isort .
mypy app

# Database
alembic init alembic
alembic revision --autogenerate -m "message"
alembic upgrade head

# Docker
docker-compose up -d
docker-compose down
```

---

This structure follows FastAPI best practices and is production-ready!
