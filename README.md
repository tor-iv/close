# Close

Close – Focused on real connections, not noise.

## Overview
Close is a modern social platform that prioritizes meaningful connections over algorithmic feeds and endless scrolling. Built with FastAPI, PostgreSQL, and Redis, it provides a robust and scalable backend infrastructure for authentic social interactions.

## Tech Stack
- **Backend**: FastAPI (Python 3.12)
- **Database**: PostgreSQL 14
- **Caching**: Redis 7
- **Task Queue**: Celery
- **Containerization**: Docker & Docker Compose
- **CI/CD**: GitHub Actions

## Getting Started

### Prerequisites
- Docker and Docker Compose
- Python 3.12 (for local development)
- Git

### Development Setup

1. Clone the repository:
```bash
git clone <repository-url>
cd close
```

2. Create and configure environment variables:
```bash
cp .env.example .env
# Edit .env with your configuration
```

3. Start the development environment:
```bash
docker-compose up -d
```

The API will be available at http://localhost:8000

### Local Development Without Docker

1. Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: .\venv\Scripts\activate
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Start the development server:
```bash
uvicorn app.main:app --reload
```

## Project Structure
```
/app
  /api
    /v1
      /endpoints  # API route handlers
  /core          # Core functionality, config
  /crud          # Database CRUD operations
  /db            # Database models and setup
  /models        # Pydantic models
  /schemas       # Request/Response schemas
  /utils         # Utility functions
/migrations      # Alembic migrations
/tests          # Test suite
```

## Development Workflow

1. Create a new branch for your feature:
```bash
git checkout -b feature/your-feature-name
```

2. Make your changes and ensure tests pass:
```bash
pytest
flake8 .
black .
```

3. Create a pull request to the `develop` branch

## Code Quality
The project uses:
- `flake8` for linting
- `black` for code formatting
- `bandit` for security checks
- `pytest` for testing

CI/CD pipelines automatically check these on pull requests.

## Contributing
1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License
[Add License Information]
