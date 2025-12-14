# Docker Deployment Guide

This guide explains how to deploy the Django Admin project using Docker and Docker Compose.

## 📋 Prerequisites

- Docker (version 20.10 or higher)
- Docker Compose (version 1.29 or higher)

### Install Docker

**Windows:**
- Download and install [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop)

**macOS:**
- Download and install [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop)

**Linux:**
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

## 🚀 Quick Start

### 1. Configure Environment Variables

Copy the Docker environment template:

```bash
# Windows PowerShell
Copy-Item .env.docker .env

# Linux/macOS
cp .env.docker .env
```

Edit `.env` and update the following:

```env
SECRET_KEY=your-unique-secret-key-here
DEBUG=False
DB_PASSWORD=your-secure-database-password
ALLOWED_HOSTS=yourdomain.com,localhost
```

### 2. Build and Start Services

```bash
docker-compose up -d --build
```

This command will:
- Build the Django application image
- Start PostgreSQL database
- Start Django web server
- Start Nginx reverse proxy
- Run migrations automatically
- Collect static files

### 3. Create Superuser (Optional)

If you need to create an admin user manually:

```bash
docker-compose exec web python manage.py createsuperuser
```

### 4. Access the Application

- **Application**: http://localhost
- **Admin Panel**: http://localhost/admin/
- **Direct Django**: http://localhost:8000/admin/

Default superuser (created automatically):
- Username: `admin`
- Password: `admin`

**⚠️ Change these credentials immediately in production!**

## 🛠️ Docker Commands

### View Running Containers

```bash
docker-compose ps
```

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f web
docker-compose logs -f db
docker-compose logs -f nginx
```

### Stop Services

```bash
docker-compose stop
```

### Stop and Remove Containers

```bash
docker-compose down
```

### Stop and Remove Everything (including volumes)

```bash
docker-compose down -v
```

### Restart Services

```bash
docker-compose restart
```

### Execute Commands in Container

```bash
# Django shell
docker-compose exec web python manage.py shell

# Create migrations
docker-compose exec web python manage.py makemigrations

# Apply migrations
docker-compose exec web python manage.py migrate

# Collect static files
docker-compose exec web python manage.py collectstatic --noinput
```

### Access Container Shell

```bash
docker-compose exec web bash
```

### Rebuild After Code Changes

```bash
docker-compose up -d --build
```

## 📁 Docker Files Overview

### Dockerfile
Defines the Django application container:
- Based on Python 3.8
- Installs system and Python dependencies
- Copies application code
- Runs Gunicorn server

### docker-compose.yml
Orchestrates multiple services:
- **db**: PostgreSQL database
- **web**: Django application
- **nginx**: Reverse proxy and static file server

### nginx.conf
Nginx configuration:
- Proxies requests to Django
- Serves static and media files efficiently
- Caching for static assets

### entrypoint.sh
Startup script:
- Waits for database
- Runs migrations
- Collects static files
- Creates default superuser
- Starts Gunicorn

## 🔧 Configuration Options

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `SECRET_KEY` | Django secret key | (required) |
| `DEBUG` | Debug mode | `False` |
| `DB_NAME` | Database name | `djangodb` |
| `DB_USER` | Database user | `djangouser` |
| `DB_PASSWORD` | Database password | (required) |
| `DB_HOST` | Database host | `db` |
| `DB_PORT` | Database port | `5432` |
| `ALLOWED_HOSTS` | Allowed hosts | `localhost` |

### Customize Ports

Edit `docker-compose.yml` to change ports:

```yaml
services:
  web:
    ports:
      - "8080:8000"  # Change 8080 to your desired port
  
  nginx:
    ports:
      - "8081:80"    # Change 8081 to your desired port
```

### Scale Workers

Increase Gunicorn workers in `docker-compose.yml`:

```yaml
command: gunicorn myproject.wsgi:application --bind 0.0.0.0:8000 --workers 5
```

## 🗄️ Database Management

### Backup Database

```bash
docker-compose exec db pg_dump -U djangouser djangodb > backup.sql
```

### Restore Database

```bash
docker-compose exec -T db psql -U djangouser djangodb < backup.sql
```

### Access PostgreSQL CLI

```bash
docker-compose exec db psql -U djangouser djangodb
```

## 🚢 Production Deployment

### 1. Update Settings

In `.env`:
```env
DEBUG=False
SECRET_KEY=generate-a-strong-unique-key
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com
```

### 2. Use Production Dockerfile (Optional)

Create `Dockerfile.prod` for optimized production build:

```dockerfile
FROM python:3.8-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /app/wheels -r requirements.txt

FROM python:3.8-slim
WORKDIR /app
COPY --from=builder /app/wheels /wheels
COPY --from=builder /app/requirements.txt .
RUN pip install --no-cache /wheels/*
COPY . /app
RUN python manage.py collectstatic --noinput
CMD ["gunicorn", "myproject.wsgi:application", "--bind", "0.0.0.0:8000"]
```

### 3. SSL/HTTPS Configuration

Add SSL certificates to nginx.conf:

```nginx
server {
    listen 443 ssl;
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    # ... rest of configuration
}
```

### 4. Deploy to Cloud

**AWS ECS, Google Cloud Run, Azure Container Instances:**
- Push image to container registry
- Configure environment variables
- Deploy container

**DigitalOcean, Linode, VPS:**
- Install Docker and Docker Compose
- Clone repository
- Run `docker-compose up -d`

## 🐛 Troubleshooting

### Port Already in Use

```bash
# Check what's using the port
netstat -ano | findstr :8000  # Windows
lsof -i :8000                 # Linux/macOS

# Change port in docker-compose.yml
```

### Database Connection Error

```bash
# Check if database is running
docker-compose ps

# View database logs
docker-compose logs db

# Restart database
docker-compose restart db
```

### Permission Denied Errors

```bash
# Linux/macOS: Fix entrypoint.sh permissions
chmod +x entrypoint.sh

# Rebuild
docker-compose up -d --build
```

### Static Files Not Loading

```bash
# Recollect static files
docker-compose exec web python manage.py collectstatic --noinput

# Restart nginx
docker-compose restart nginx
```

### Clear Everything and Start Fresh

```bash
docker-compose down -v
docker-compose up -d --build
```

## 📊 Monitoring

### View Resource Usage

```bash
docker stats
```

### View Container Processes

```bash
docker-compose top
```

## 🔐 Security Best Practices

1. **Never use DEBUG=True in production**
2. **Use strong SECRET_KEY**
3. **Change default superuser password**
4. **Use environment variables for secrets**
5. **Keep Docker images updated**
6. **Use non-root user in containers**
7. **Implement SSL/TLS**
8. **Regular backups**
9. **Monitor logs for suspicious activity**

## 📚 Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Django Deployment Checklist](https://docs.djangoproject.com/en/stable/howto/deployment/checklist/)
- [Gunicorn Documentation](https://docs.gunicorn.org/)
- [Nginx Documentation](https://nginx.org/en/docs/)

## 🆘 Support

If you encounter issues:
1. Check the logs: `docker-compose logs -f`
2. Verify environment variables in `.env`
3. Ensure Docker and Docker Compose are up to date
4. Check the troubleshooting section above

---

**Happy Dockerizing! 🐳**

