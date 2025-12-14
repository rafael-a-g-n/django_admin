# Django Admin - Online Course Management System

A Django-based web application for managing online courses, instructors, learners, and course content. This project demonstrates Django's powerful admin interface with custom configurations for managing educational content.

## 📋 Table of Contents

- [Features](#features)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [Local Development](#local-development)
  - [Docker Deployment](#docker-deployment)
- [Configuration](#configuration)
- [Running the Application](#running-the-application)
- [Admin Interface](#admin-interface)
- [Models Overview](#models-overview)
- [Docker Guide](#docker-guide)
- [Deployment](#deployment)
- [Contributing](#contributing)

## ✨ Features

- **Course Management**: Create and manage online courses with descriptions, images, and publication dates
- **Instructor Management**: Manage instructors with user associations and employment status
- **Learner Profiles**: Track learners with occupation types and social links
- **Lesson Management**: Organize course content into structured lessons
- **Enrollment System**: Track course enrollments with different modes (Audit, Honor, BETA)
- **Question & Quiz System**: Create questions with multiple choice answers
- **Custom Django Admin**: Enhanced admin interface with inline editing and custom field arrangements
- **PostgreSQL Database**: Production-ready database configuration
- **Environment Variable Management**: Secure credential management using `.env` files
- **Docker Support**: Complete Docker and Docker Compose setup for easy deployment

## 📁 Project Structure

```
django_admin/
├── adminsite/              # Main application
│   ├── migrations/         # Database migrations
│   ├── admin.py           # Custom admin configurations
│   ├── models.py          # Database models
│   ├── views.py           # Views
│   ├── apps.py            # App configuration
│   └── tests.py           # Tests
├── myproject/             # Project settings
│   ├── settings.py        # Django settings
│   ├── urls.py            # URL routing
│   ├── wsgi.py            # WSGI configuration
│   └── asgi.py            # ASGI configuration
├── .env                   # Environment variables (not in git)
├── .env.example           # Environment variables template
├── .env.docker            # Docker environment template
├── .gitignore             # Git ignore rules
├── Dockerfile             # Docker image definition
├── docker-compose.yml     # Docker orchestration
├── nginx.conf             # Nginx configuration
├── entrypoint.sh          # Docker startup script
├── manage.py              # Django management script
├── requirements.txt       # Python dependencies
├── runtime.txt            # Python version specification          # Environment setup documentation
├── DOCKER.md             # Docker deployment guide
└── README.md             # This file
```

## 🔧 Prerequisites

### For Local Development
- **Python**: 3.8 or higher
- **PostgreSQL**: 12 or higher (optional, can use SQLite for development)
- **pip**: Python package manager
- **virtualenv**: For creating isolated Python environments

### For Docker Deployment
- **Docker**: 20.10 or higher
- **Docker Compose**: 1.29 or higher

## 📦 Installation

You can run this project either locally or using Docker. Choose the method that suits your needs.

### Local Development

#### 1. Clone the Repository

```bash
git clone https://github.com/rafael-a-g-n/django_admin.git
cd django_admin
```

#### 2. Create Virtual Environment

**Windows (PowerShell):**
```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**Linux/macOS:**
```bash
python3 -m venv .venv
source .venv/bin/activate
```

#### 3. Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

#### 4. Configure Environment

```bash
# Windows
Copy-Item .env.example .env

# Linux/macOS
cp .env.example .env
```

Edit `.env` with your database credentials.

#### 5. Run Migrations

```bash
python manage.py migrate
python manage.py createsuperuser
```

#### 6. Start Server

```bash
python manage.py runserver
```

Visit: http://127.0.0.1:8000/admin/

### Docker Deployment

#### 1. Clone the Repository

```bash
git clone https://github.com/rafael-a-g-n/django_admin.git
cd django_admin
```

#### 2. Configure Environment

```bash
# Windows
Copy-Item .env.docker .env

# Linux/macOS
cp .env.docker .env
```

Edit `.env` and update credentials (especially `DB_PASSWORD` and `SECRET_KEY`).

#### 3. Build and Start

```bash
docker-compose up -d --build
```

This will:
- Build the Django application
- Start PostgreSQL database
- Run migrations automatically
- Create a default superuser (admin/admin)
- Start Nginx reverse proxy

#### 4. Access Application

- **Via Nginx**: http://localhost
- **Direct Django**: http://localhost:8000
- **Admin Panel**: http://localhost/admin/

Default credentials: `admin` / `admin` (change immediately!)

#### 5. View Logs

```bash
docker-compose logs -f
```

For detailed Docker instructions, see **[DOCKER.md](DOCKER.md)**.

## ⚙️ Configuration

### Environment Variables

Create a `.env` file from the template and configure:

```env
# Django Settings
SECRET_KEY=your-secret-key-here
DEBUG=True

# Database Configuration
DB_NAME=djangodb
DB_USER=djangouser
DB_PASSWORD=your-secure-password
DB_HOST=localhost  # or 'db' for Docker
DB_PORT=5432

# Allowed Hosts (comma-separated)
ALLOWED_HOSTS=localhost,127.0.0.1
```

For detailed environment setup, see **[ENV_SETUP.md](ENV_SETUP.md)**.

## 🚀 Running the Application

### Local Development

```bash
# Activate virtual environment
.venv\Scripts\Activate.ps1  # Windows
source .venv/bin/activate   # Linux/macOS

# Run migrations
python manage.py migrate

# Start server
python manage.py runserver
```

### Docker

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Rebuild after changes
docker-compose up -d --build
```

### Common Django Commands

```bash
# Create migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Collect static files
python manage.py collectstatic

# Run tests
python manage.py test

# Django shell
python manage.py shell
```

### Docker Commands

```bash
# Execute command in container
docker-compose exec web python manage.py migrate

# Access container shell
docker-compose exec web bash

# View database
docker-compose exec db psql -U djangouser djangodb

# Backup database
docker-compose exec db pg_dump -U djangouser djangodb > backup.sql
```

## 👨‍💼 Admin Interface

The Django admin interface has been customized with:

### Course Admin
- **Inline Lesson Management**: Add/edit up to 5 lessons directly from the course page
- **Custom Field Order**: `pub_date`, `name`, `description`
- **Stacked Inline Layout**: Clean, vertical layout for lessons

### Instructor Admin
- **Simplified Fields**: `user`, `full_time`
- **User Association**: Link instructors to Django user accounts

### Accessing the Admin Panel

**Local Development:**
http://127.0.0.1:8000/admin/

**Docker:**
http://localhost/admin/

Log in with your superuser credentials.

## 📊 Models Overview

### Course
- Fields: name, image, description, pub_date, instructors, users, total_enrollment
- Relationships: Many-to-Many with Instructors and Users
- Features: Total score calculation, enrollment tracking

### Instructor
- Fields: user (ForeignKey), full_time, total_learners
- Relationship: One-to-One with User model

### Learner
- Fields: user (ForeignKey), occupation, social_link
- Occupation Types: Student, Developer, Data Scientist, Database Admin

### Lesson
- Fields: title, order, course, content
- Relationship: Many-to-One with Course

### Enrollment
- Fields: user, course, date_enrolled, mode, rating
- Enrollment Modes: Audit, Honor, BETA

### Question & Choice
- Question Fields: lesson, question_text, grade
- Choice Fields: question, choice_text, is_correct
- Features: Automatic grading logic

### Submission
- Fields: enrollment, choices
- Relationship: Many-to-Many with Choices

## 🐳 Docker Guide

This project includes complete Docker support for easy deployment anywhere.

### Quick Start

```bash
# Start everything
docker-compose up -d

# View logs
docker-compose logs -f

# Stop everything
docker-compose down
```

### Services Included

- **PostgreSQL**: Database server
- **Django Web**: Application server (Gunicorn)
- **Nginx**: Reverse proxy and static file server

### Docker Files

- `Dockerfile`: Application container definition
- `docker-compose.yml`: Multi-container orchestration
- `nginx.conf`: Nginx web server configuration
- `entrypoint.sh`: Container startup script
- `.env.docker`: Docker environment template

For complete Docker documentation, see **[DOCKER.md](DOCKER.md)**.

## 🌐 Deployment

### Production Checklist

- [ ] Set `DEBUG=False` in `.env`
- [ ] Update `ALLOWED_HOSTS` with your domain
- [ ] Use strong `SECRET_KEY` and `DB_PASSWORD`
- [ ] Configure static files: `python manage.py collectstatic`
- [ ] Set up HTTPS/SSL certificates
- [ ] Configure proper logging
- [ ] Enable security middleware
- [ ] Set up regular database backups
- [ ] Configure monitoring and alerting

### Deployment Options

**Docker (Recommended):**
```bash
docker-compose -f docker-compose.yml up -d
```

**Traditional VPS:**
- Use Gunicorn as WSGI server
- Nginx as reverse proxy
- PostgreSQL database
- Supervisor for process management

**Cloud Platforms:**
- AWS ECS / Fargate
- Google Cloud Run
- Azure Container Instances
- Heroku
- DigitalOcean App Platform

## 🧪 Testing

```bash
# Run all tests
python manage.py test

# Run specific app tests
python manage.py test adminsite

# With coverage
pip install coverage
coverage run --source='.' manage.py test
coverage report
```

## 🛠️ Common Commands Reference

### Local Development

```bash
# Database
python manage.py makemigrations
python manage.py migrate
python manage.py dbshell

# User Management
python manage.py createsuperuser
python manage.py changepassword username

# Static Files
python manage.py collectstatic
python manage.py findstatic filename

# Development
python manage.py runserver
python manage.py runserver 0.0.0.0:8080
python manage.py shell

# Testing
python manage.py test
python manage.py test --keepdb
```

### Docker

```bash
# Container Management
docker-compose up -d
docker-compose down
docker-compose restart
docker-compose ps
docker-compose logs -f

# Execute Commands
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py createsuperuser
docker-compose exec web bash

# Database
docker-compose exec db psql -U djangouser djangodb
docker-compose exec db pg_dump -U djangouser djangodb > backup.sql
```

## 🐛 Troubleshooting

### Port Already in Use

```bash
# Local
python manage.py runserver 8080

# Docker
# Edit docker-compose.yml ports section
```

### Database Connection Issues

```bash
# Check PostgreSQL is running
# Verify credentials in .env
# For Docker: docker-compose logs db
```

### Static Files Not Loading

```bash
# Local
python manage.py collectstatic --clear

# Docker
docker-compose exec web python manage.py collectstatic --noinput
docker-compose restart nginx
```

### Docker Issues

```bash
# View logs
docker-compose logs -f

# Rebuild
docker-compose up -d --build

# Clean start
docker-compose down -v
docker-compose up -d --build
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

## 📚 Additional Resources

- [Django Documentation](https://docs.djangoproject.com/)
- [Django Admin Documentation](https://docs.djangoproject.com/en/stable/ref/contrib/admin/)
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Gunicorn Documentation](https://docs.gunicorn.org/)
- [Nginx Documentation](https://nginx.org/en/docs/)

## 📄 License

This project is part of the IBM Django Skills Network course. Please refer to the course materials for licensing information.

## 👤 Author

**Rafael Nunes**
- GitHub: [@rafael-a-g-n](https://github.com/rafael-a-g-n)

## 🙏 Acknowledgments

- IBM Skills Network for the course template and guidance
- Django Software Foundation for the excellent framework
- The open-source community

---

**Note**: This is an educational project developed as part of the IBM Django course on Coursera.

For detailed deployment instructions, see:
- **[DOCKER.md](DOCKER.md)** - Complete Docker deployment guide
- **[ENV_SETUP.md](ENV_SETUP.md)** - Environment variable configuration
