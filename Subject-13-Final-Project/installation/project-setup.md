# Final Project Development Environment Setup

## Overview
This guide covers setting up a comprehensive development environment for the final project that integrates all course concepts: Git workflows, Docker, FastAPI, PostgreSQL, web crawling, and AI-assisted frontend development.

## Prerequisites
- Completion of all previous subjects
- All tools from Subjects 1-13 installed
- GitHub account with repository access
- Understanding of all course technologies

---

## Project Planning & Architecture

### Project Requirements Analysis
The final project requires building a complete web application that includes:

1. **Data Collection**: Web crawling to gather data
2. **Data Storage**: PostgreSQL database with optimized schema
3. **Data Processing**: ETL pipelines for data transformation
4. **API Layer**: FastAPI REST API for data access
5. **Frontend**: AI-generated modern web interface
6. **DevOps**: Docker containerization and CI/CD
7. **Version Control**: Professional Git workflow

### Technology Stack Selection
Based on course subjects, use:
- **Backend**: FastAPI (Python)
- **Database**: PostgreSQL
- **Crawling**: Crawlee Python
- **Containerization**: Docker & Docker Compose
- **Frontend**: React/Next.js with AI assistance
- **CI/CD**: GitHub Actions
- **Version Control**: Git with Git Flow

---

## Complete Development Environment Setup

### 1. Project Repository Setup
```bash
# Create project directory
mkdir final-project
cd final-project

# Initialize Git repository
git init

# Create initial structure
mkdir -p \
  backend/{app,tests,scripts} \
  frontend/{src,public,tests} \
  crawler/{src,tests,scripts} \
  database/{migrations,seeds} \
  docker \
  docs \
  .github/workflows

# Create essential files
touch README.md docker-compose.yml .env.example
touch backend/requirements.txt frontend/package.json
```

### 2. Git Flow Setup
```bash
# Install git-flow
sudo apt install git-flow  # Linux
brew install git-flow      # macOS

# Initialize git-flow
git flow init

# Create develop branch
git checkout -b develop
git push -u origin develop

# Create feature branches
git flow feature start setup-project-structure
```

---

## Backend Setup (FastAPI)

### Requirements & Dependencies
```bash
# backend/requirements.txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlalchemy==2.0.23
asyncpg==0.29.0
alembic==1.12.1
pydantic[email]==2.5.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
redis==5.0.1
celery==5.3.4
```

### FastAPI Application Structure
```python
# backend/app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api.v1.api import api_router
from app.core.config import settings

app = FastAPI(
    title=settings.PROJECT_NAME,
    openapi_url=f"{settings.API_V1_STR}/openapi.json"
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.BACKEND_CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include API router
app.include_router(api_router, prefix=settings.API_V1_STR)

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

### Database Configuration
```python
# backend/app/core/config.py
from pydantic import BaseSettings

class Settings(BaseSettings):
    PROJECT_NAME: str = "Final Project API"
    VERSION: str = "1.0.0"
    API_V1_STR: str = "/api/v1"

    # Database
    POSTGRES_SERVER: str = "localhost"
    POSTGRES_USER: str = "postgres"
    POSTGRES_PASSWORD: str = "password"
    POSTGRES_DB: str = "finalproject"
    DATABASE_URL: str = f"postgresql://{POSTGRES_USER}:{POSTGRES_PASSWORD}@{POSTGRES_SERVER}/{POSTGRES_DB}"

    # CORS
    BACKEND_CORS_ORIGINS: list = ["http://localhost:3000", "http://localhost:8080"]

    class Config:
        env_file = ".env"

settings = Settings()
```

---

## Database Setup (PostgreSQL)

### Docker Compose for Development
```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=finalproject
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  api:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db/finalproject
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    volumes:
      - ./backend:/app
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  crawler:
    build: ./crawler
    environment:
      - DATABASE_URL=postgresql://postgres:password@db/finalproject
    depends_on:
      - db
    volumes:
      - ./crawler:/app
      - ./data:/app/data
    command: python src/main.py

volumes:
  postgres_data:
```

### Database Schema Design
```sql
-- database/init.sql
-- Create extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements";

-- Create database roles
CREATE ROLE readonly;
CREATE ROLE readwrite;

-- Grant permissions
GRANT CONNECT ON DATABASE finalproject TO readonly;
GRANT CONNECT ON DATABASE finalproject TO readwrite;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT USAGE ON SCHEMA public TO readwrite;

-- Create tables (will be migrated via Alembic)
-- Tables will be created by backend migrations
```

---

## Crawler Setup (Crawlee Python)

### Crawler Dependencies
```bash
# crawler/requirements.txt
crawlee==0.1.0
sqlalchemy==2.0.23
asyncpg==0.29.0
pydantic==2.5.0
loguru==0.7.2
aiofiles==23.2.1
beautifulsoup4==4.12.2
lxml==4.9.3
selenium==4.15.2
webdriver-manager==4.0.1
```

### Crawler Application Structure
```python
# crawler/src/main.py
import asyncio
import os
from loguru import logger
from crawlers.news_crawler import NewsCrawler
from processors.article_processor import ArticleProcessor
from database.connection import init_db, save_articles

async def main():
    """Main ETL pipeline"""
    logger.info("Starting final project ETL pipeline")

    # Initialize database
    await init_db()

    # Configure crawler
    crawler = NewsCrawler(
        delay=float(os.getenv("CRAWL_DELAY", "1.0")),
        max_concurrent=int(os.getenv("MAX_CONCURRENT", "3"))
    )

    # Define target websites
    urls = [
        "https://news.ycombinator.com/",
        "https://techcrunch.com/",
        # Add more relevant sources
    ]

    # Crawl websites
    logger.info(f"Starting crawl of {len(urls)} websites")
    raw_data = await crawler.crawl_and_extract(urls)

    # Process data
    processor = ArticleProcessor()
    processed_data = processor.process_batch(raw_data)

    # Save to database
    await save_articles(processed_data)

    logger.info(f"ETL pipeline completed. Processed {len(processed_data)} articles.")

if __name__ == "__main__":
    asyncio.run(main())
```

---

## Frontend Setup (React + AI)

### Frontend Dependencies
```json
// frontend/package.json
{
  "name": "final-project-frontend",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@testing-library/jest-dom": "^5.16.4",
    "@testing-library/react": "^13.3.0",
    "@testing-library/user-event": "^13.5.0",
    "axios": "^1.4.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.14.2",
    "react-scripts": "5.0.1",
    "styled-components": "^6.0.7",
    "tailwindcss": "^3.3.3",
    "web-vitals": "^2.1.4"
  },
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-scripts eject"
  }
}
```

### React Application Structure
```jsx
// frontend/src/App.js
import React from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Header from './components/Header';
import Home from './pages/Home';
import Articles from './pages/Articles';
import Dashboard from './pages/Dashboard';

function App() {
  return (
    <Router>
      <div className="min-h-screen bg-gray-50">
        <Header />
        <main className="container mx-auto px-4 py-8">
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/articles" element={<Articles />} />
            <Route path="/dashboard" element={<Dashboard />} />
          </Routes>
        </main>
      </div>
    </Router>
  );
}

export default App;
```

---

## CI/CD Setup (GitHub Actions)

### GitHub Actions Workflow
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test-backend:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - uses: actions/checkout@v4
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    - name: Install dependencies
      run: |
        cd backend
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Run tests
      run: |
        cd backend
        python -m pytest tests/ -v --cov=app --cov-report=xml
    - name: Upload coverage
      uses: codecov/codecov-action@v3

  test-frontend:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Set up Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
    - name: Install dependencies
      run: |
        cd frontend
        npm ci
    - name: Run tests
      run: |
        cd frontend
        npm test -- --coverage --watchAll=false
    - name: Build
      run: |
        cd frontend
        npm run build

  deploy:
    needs: [test-backend, test-frontend]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - name: Deploy to production
      run: echo "Deployment steps here"
```

---

## Docker Configuration

### Backend Dockerfile
```dockerfile
# backend/Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Run the application
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Frontend Dockerfile
```dockerfile
# frontend/Dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Expose port
EXPOSE 3000

# Start the application
CMD ["npm", "start"]
```

---

## Environment Configuration

### .env.example
```bash
# Database
POSTGRES_SERVER=localhost
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
POSTGRES_DB=finalproject
DATABASE_URL=postgresql://postgres:password@localhost/finalproject

# Redis
REDIS_URL=redis://localhost:6379

# API
SECRET_KEY=your-super-secret-key-here
API_V1_STR=/api/v1
BACKEND_CORS_ORIGINS=["http://localhost:3000", "http://localhost:8080"]

# Crawler
CRAWL_DELAY=1.0
MAX_CONCURRENT_REQUESTS=3
USER_AGENT=FinalProject/1.0

# Frontend
REACT_APP_API_URL=http://localhost:8000/api/v1
```

---

## Development Scripts

### Project Management Scripts
```bash
# scripts/setup.sh - Complete environment setup
#!/bin/bash
echo "Setting up final project environment..."

# Create virtual environments
python -m venv backend/venv
source backend/venv/bin/activate
pip install -r backend/requirements.txt

# Frontend setup
cd frontend
npm install
cd ..

# Copy environment file
cp .env.example .env

echo "Environment setup complete!"
```

```bash
# scripts/run-dev.sh - Run development environment
#!/bin/bash
echo "Starting development environment..."

# Start Docker services
docker-compose up -d db redis

# Wait for services
sleep 10

# Run backend
cd backend
source venv/bin/activate
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000 &
BACKEND_PID=$!

# Run frontend
cd ../frontend
npm start &
FRONTEND_PID=$!

# Run crawler (optional)
cd ../crawler
python src/main.py &
CRAWLER_PID=$!

echo "Development environment running!"
echo "Backend: http://localhost:8000"
echo "Frontend: http://localhost:3000"
echo "API Docs: http://localhost:8000/docs"

# Wait for processes
wait $BACKEND_PID $FRONTEND_PID $CRAWLER_PID
```

---

## Testing Strategy

### Backend Testing
```python
# backend/tests/test_main.py
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_health_check():
    response = client.get("/health")
    assert response.status_code == 200
    assert response.json() == {"status": "healthy"}

def test_api_v1_exists():
    response = client.get("/api/v1/")
    assert response.status_code == 200
```

### Frontend Testing
```jsx
// frontend/src/components/Header.test.js
import { render, screen } from '@testing-library/react';
import Header from './Header';

test('renders header with navigation', () => {
  render(<Header />);
  expect(screen.getByText('Final Project')).toBeInTheDocument();
  expect(screen.getByText('Articles')).toBeInTheDocument();
  expect(screen.getByText('Dashboard')).toBeInTheDocument();
});
```

---

## Documentation Setup

### Project Documentation
```markdown
# Final Project: Complete Web Application

## Overview
This project demonstrates the integration of all course concepts into a complete web application featuring data collection, processing, storage, and presentation.

## Architecture
- **Data Layer**: PostgreSQL with optimized schemas
- **API Layer**: FastAPI with async operations
- **Processing Layer**: ETL pipelines with Crawlee
- **Presentation Layer**: React frontend with AI assistance
- **Infrastructure**: Docker containerization
- **CI/CD**: GitHub Actions automation

## Quick Start
1. Clone the repository
2. Run `./scripts/setup.sh`
3. Start development with `./scripts/run-dev.sh`
4. Access frontend at http://localhost:3000

## API Documentation
Available at http://localhost:8000/docs when running.
```

---

## Next Steps

1. **Planning Phase**: Define project requirements and scope
2. **Architecture Design**: Create detailed system architecture
3. **Development Phase**: Implement components iteratively
4. **Integration Phase**: Connect all components
5. **Testing Phase**: Comprehensive testing and validation
6. **Deployment Phase**: Production deployment and monitoring

---

## Success Criteria

- ✅ Complete ETL pipeline from crawling to API
- ✅ Modern web interface with AI-generated components
- ✅ Docker containerization for all services
- ✅ CI/CD pipeline with automated testing
- ✅ Professional Git workflow and documentation
- ✅ Performance optimization and monitoring
- ✅ Security best practices implementation

---

## Resources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [React Documentation](https://reactjs.org/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker Documentation](https://docs.docker.com/)
- [GitHub Actions](https://docs.github.com/en/actions)

---

*This final project represents the culmination of all course learning, demonstrating the ability to build a complete, production-ready web application using modern development practices.*