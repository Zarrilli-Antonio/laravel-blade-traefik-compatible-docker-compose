# Laravel Blade - Docker Setup Guide

This guide will help you configure and start the Laravel project using Docker.

## 📋 Prerequisites

Before starting, make sure you have installed:
- Docker (version 20.10 or higher)
- Docker Compose (version 1.29 or higher)
- Git

## 🚀 Initial Setup

### 1. Navigate to the project directory

```bash
cd /home/antonio/sites/laravel-blade
```

### 2. Configure Docker variables

Create the `.env` file by copying the example file:

```bash
cp .env.example .env
```

Modify `.env` according to your needs. The main variables are:

- **TRAEFIK_DOMAIN**: The domain to access the application via Traefik
- **PORT_NGINX**: Local port to access via browser (default: 8080)
- **PORT_MYSQL**: MySQL port exposed locally (default: 3307)
- **PORT_REDIS**: Redis port exposed locally (default: 6380)
- **PORT_PHPMYADMIN**: phpMyAdmin port (default: 8082)
- **MYSQL_DATABASE**: Database name
- **MYSQL_USER**: Database username
- **MYSQL_PASSWORD**: Database password

**Customization example**:
```env
# Change the domain
TRAEFIK_DOMAIN=my-project.example.com

# Change ports if there are conflicts
PORT_NGINX=9090
PORT_MYSQL=3308
```

### 3. Build and start the Docker containers

```bash
docker compose up -d --build
```

**Explanation**: 
- `up`: starts the containers
- `-d`: runs in background (detached mode)
- `--build`: rebuilds the Docker images if necessary

This command will create and start the following services:
- **app**: PHP-FPM container (port 9000)
- **nginx**: Nginx web server (port 8080)
- **db**: MySQL 8.0 database (port 3307)
- **redis**: Redis cache (port 6380)
- **phpmyadmin**: Web interface for MySQL (port 8082)

### 4. Install Laravel (IMPORTANT: first startup)

If this is your first run of the boilerplate (folder duplicated from laravel-blade), you must install Laravel:

```bash
mkdir -p laravel
docker compose up -d --build
docker compose exec -u root app bash -c "cd /var/www && composer create-project laravel/laravel laravel --prefer-dist && chown -R laravel:laravel /var/www/laravel"
docker compose exec app chmod -R 775 storage bootstrap/cache
```

**Explanation**: 
- Creates the `laravel/` folder where the project will be installed
- Starts the Docker containers
- Installs Laravel as root in the `laravel/` subfolder and then changes permissions to the laravel user
- Sets correct permissions for storage and cache

This separates Docker configuration files (in the root) from Laravel files (in `laravel/`), avoiding conflicts between Docker `.env` and Laravel `.env`.

### 5. Verify that the containers are active

```bash
docker-compose ps
```

You should see all services in "Up" state.

### 6. Configure the Laravel database

Modify the `laravel/.env` file to configure the MySQL database connection:

```bash
# Edit laravel/.env
```

Update the following variables:

```env
APP_NAME=LaravelBlade
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8080

DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=laravel

CACHE_DRIVER=redis
SESSION_DRIVER=redis

REDIS_HOST=redis
REDIS_PASSWORD=null
REDIS_PORT=6379
```

**Note**: The `APP_KEY` has been generated automatically during Laravel installation.

### 7. Set correct permissions for storage and cache directories

```bash
docker compose exec app chmod -R 777 storage bootstrap/cache
```

**Explanation**: Assigns the necessary permissions to the directories where Laravel writes temporary files, logs and cache.

### 8. Run database migrations

```bash
docker compose exec app php artisan migrate
```

**Explanation**: Creates the tables in the database according to the schemas defined in the migrations.

If you also want to populate the database with test data:

```bash
docker compose exec app php artisan db:seed
```

### 9. (Optional) Install NPM dependencies for frontend

If the project uses frontend assets (CSS/JS):

```bash
docker compose exec app npm install
docker compose exec app npm run dev
```

## 🌐 Access to the Application

Once all steps are completed, you can access:

- **Laravel Application (local)**: http://localhost:8080
- **Laravel Application (Traefik)**: https://laravel-myproject.domain.com (if configured with Traefik)
- **phpMyAdmin**: http://localhost:8082
  - Server: `db`
  - Username: `laravel`
  - Password: `laravel`
  - Database: `laravel`

### 🔒 Traefik Configuration

The project is configured to work with Traefik as a reverse proxy. To use it:

1. **Make sure Traefik is running** on your system with a network called `traefik_proxy`

2. **Configure the domain** in the `.env` file:
   ```env
   TRAEFIK_DOMAIN=your-domain.com
   ```

3. **SSL Certificates**: The project uses the resolver configured in `TRAEFIK_CERTRESOLVER` (default: `cloudflare`). Modify the `.env` file if necessary:
   ```env
   TRAEFIK_CERTRESOLVER=letsencrypt
   ```

4. **Local and remote access**: 
   - The application remains accessible at `localhost:PORT_NGINX` for local development
   - With Traefik active, it will also be available at `https://TRAEFIK_DOMAIN`

### 📝 Managing Environment Variables

The project uses two separate `.env` files:

- **`.env`**: Variables for Docker Compose (ports, container names, Traefik domain, DB credentials)
- **`laravel/.env`**: Variables for the Laravel application (app configuration, cache, sessions, etc.)

**Important**: When you modify database credentials in `.env`, remember to update them also in `laravel/.env`.

**Project structure**:
```
.
├── .env                     # Docker Compose config
├── docker-compose.yml       # Container configuration
├── Dockerfile
├── nginx/
├── php/
└── laravel/                 # Laravel application
    ├── .env                 # Laravel config
    ├── app/
    ├── public/
    └── ...
```

**Important**: When you modify database credentials in `.env`, remember to update them also in `laravel/.env`
```

## 🛠️ Useful Commands

### Container Management

```bash
# Start the containers
docker compose up -d

# Stop the containers
docker compose down

# Stop containers and remove volumes (WARNING: deletes database data)
docker compose down -v

# View logs of all services
docker compose logs -f

# View logs of a specific service
docker compose logs -f app
docker compose logs -f nginx
docker compose logs -f db

# Restart a specific service
docker compose restart app
docker compose restart nginx
```

### Laravel (Artisan) Commands

```bash
# Execute artisan commands
docker compose exec app php artisan <command>

# Common examples:
docker compose exec app php artisan migrate
docker compose exec app php artisan migrate:fresh --seed
docker compose exec app php artisan cache:clear
docker compose exec app php artisan config:clear
docker compose exec app php artisan route:list
docker compose exec app php artisan make:controller NomeController
docker compose exec app php artisan make:model NomeModel -m
docker compose exec app php artisan tinker
```

### Composer

```bash
# Install a new dependency
docker compose exec app composer require name/package

# Update dependencies
docker compose exec app composer update

# Autoload dump
docker compose exec app composer dump-autoload
```

### Container Shell Access

```bash
# Access the PHP container
docker compose exec app bash

# Access the MySQL container
docker compose exec db bash

# Access MySQL client directly
docker compose exec db mysql -u laravel -plaravel laravel
```

### Database

```bash
# Export the database
docker compose exec db mysqldump -u laravel -plaravel laravel > backup.sql

# Import a database
docker compose exec -T db mysql -u laravel -plaravel laravel < backup.sql

# Complete database reset
docker compose exec app php artisan migrate:fresh --seed
```

## 🔧 Troubleshooting

### Problem: Port already in use

If you receive an error like "port is already allocated":

1. Check which process is using the port:
   ```bash
   sudo lsof -i :8080
   ```

2. Modify the port in the `docker-compose.yml` file at the line:
   ```yaml
   ports:
     - "8080:80"  # Change 8080 with another free port, e.g: 8090:80
   ```

### Problem: Permission denied

If you receive permission errors:

```bash
docker compose exec app chmod -R 777 storage bootstrap/cache
sudo chown -R $USER:$USER .
```

### Problem: Container won't start

Check the logs to identify the problem:

```bash
docker compose logs app
docker compose logs nginx
docker compose logs db
```

### Problem: Database connection error

1. Verify that the database container is active:
   ```bash
   docker compose ps db
   ```

2. Check the credentials in the `.env` file:
   ```env
   DB_HOST=db
   DB_PORT=3306
   DB_DATABASE=laravel
   DB_USERNAME=laravel
   DB_PASSWORD=laravel
   ```

3. Clear the configuration cache:
   ```bash
   docker compose exec app php artisan config:clear
   ```

## 🗑️ Complete Removal

To completely remove all containers, networks and volumes:

```bash
docker compose down -v
docker system prune -a
```

**WARNING**: This command will delete all database data!

## 📝 Important Notes

1. **Database**: The database data is persistent thanks to the Docker volume `dbdata`. It will not be lost when you stop the containers.

2. **Ports**: The project uses the following ports:
   - 8080: Nginx (web server)
   - 3307: MySQL (database)
   - 6380: Redis (cache)
   - 8082: phpMyAdmin

3. **Code Changes**: Changes to PHP files will be immediately visible without restarting containers thanks to mounted volumes.

4. **Performance**: On Linux systems, performance should be excellent. On Windows/Mac there may be slowdowns due to file system sharing.

## 📚 Useful Resources

- [Laravel Documentation](https://laravel.com/docs)
- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## 🆘 Support

For problems or questions, check:
1. Container logs: `docker-compose logs -f`
2. Container status: `docker-compose ps`
3. Official Laravel documentation

---

**Happy developing! 🚀**
