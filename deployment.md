# Deployment Guide - STI API

Panduan lengkap untuk deployment STI API ke server production menggunakan Nginx, PHP-FPM, dan MySQL dengan CI/CD automation.

**Version:** 1.0  
**Date:** December 2025  
**Stack:** Laravel 11.x | PHP 8.3 | MySQL 8.0 | Nginx

---

## 1. Prerequisites

### 1.1 Server Requirements

**Minimum Server Specifications:**
- **CPU**: 2 cores
- **RAM**: 4GB (8GB recommended)
- **Storage**: 20GB SSD
- **OS**: Ubuntu 20.04 LTS atau 22.04 LTS

**Software Requirements:**
- PHP >= 8.2 (PHP 8.3 recommended)
- MySQL >= 8.0
- Nginx >= 1.18
- Composer >= 2.0
- Git >= 2.25

### 1.2 Access Requirements

- SSH access ke server dengan sudo privileges
- GitHub repository access
- Database credentials
- Domain/Subdomain untuk aplikasi

---

## 2. Server Setup

### 2.1 Update System

```bash
sudo apt update
sudo apt upgrade -y
```

### 2.2 Install PHP 8.3 & Extensions

```bash
# Add PHP repository
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

# Install PHP 8.3 and required extensions
sudo apt install -y php8.3 \
    php8.3-fpm \
    php8.3-cli \
    php8.3-common \
    php8.3-mysql \
    php8.3-xml \
    php8.3-mbstring \
    php8.3-curl \
    php8.3-zip \
    php8.3-gd \
    php8.3-bcmath \
    php8.3-intl \
    php8.3-redis \
    php8.3-opcache

# Verify PHP installation
php8.3 -v
```

### 2.3 Configure PHP-FPM

Edit PHP-FPM configuration:

```bash
sudo nano /etc/php/8.3/fpm/php.ini
```

**Important Settings:**
```ini
memory_limit = 256M
upload_max_filesize = 50M
post_max_size = 50M
max_execution_time = 300
max_input_time = 300
```

Edit PHP-FPM pool configuration:

```bash
sudo nano /etc/php/8.3/fpm/pool.d/www.conf
```

**Important Settings:**
```ini
user = www-data
group = www-data
listen = /run/php/php8.3-fpm.sock
listen.owner = www-data
listen.group = www-data
pm = dynamic
pm.max_children = 50
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
```

Restart PHP-FPM:

```bash
sudo systemctl restart php8.3-fpm
sudo systemctl enable php8.3-fpm
```

### 2.4 Install MySQL

```bash
# Install MySQL Server
sudo apt install -y mysql-server

# Secure MySQL installation
sudo mysql_secure_installation

# Start and enable MySQL
sudo systemctl start mysql
sudo systemctl enable mysql
```

**Create Database and User:**

```bash
sudo mysql -u root -p
```

```sql
-- Create database for staging
CREATE DATABASE sti_api_staging CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create database for production
CREATE DATABASE sti_api_prod CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create user (replace 'password' with strong password)
CREATE USER 'sti_api_user'@'localhost' IDENTIFIED BY 'strong_password_here';

-- Grant privileges
GRANT ALL PRIVILEGES ON sti_api_staging.* TO 'sti_api_user'@'localhost';
GRANT ALL PRIVILEGES ON sti_api_prod.* TO 'sti_api_user'@'localhost';

FLUSH PRIVILEGES;
EXIT;
```

### 2.5 Install Nginx

```bash
sudo apt install -y nginx

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

### 2.6 Install Composer

```bash
# Download Composer
cd /tmp
curl -sS https://getcomposer.org/installer | php

# Move to global location
sudo mv composer.phar /usr/local/bin/composer

# Verify installation
composer --version
```

### 2.7 Install Git

```bash
sudo apt install -y git

# Configure Git (optional)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

---

## 3. Initial Deployment

### 3.1 Create Application Directories

```bash
# Create directories for staging and production
sudo mkdir -p /var/www/sti-api
sudo mkdir -p /var/www/sti-api-prod

# Set ownership
sudo chown -R $USER:www-data /var/www/sti-api
sudo chown -R $USER:www-data /var/www/sti-api-prod

# Set permissions
sudo chmod -R 755 /var/www/sti-api
sudo chmod -R 755 /var/www/sti-api-prod
```

### 3.2 Clone Repository (Staging)

```bash
cd /var/www/sti-api
git clone <repository-url> .

# Checkout staging branch
git checkout staging
```

### 3.3 Clone Repository (Production)

```bash
cd /var/www/sti-api-prod
git clone <repository-url> .

# Checkout production branch
git checkout production
```

### 3.4 Install Dependencies

**For Staging:**
```bash
cd /var/www/sti-api
composer install --no-dev --optimize-autoloader
```

**For Production:**
```bash
cd /var/www/sti-api-prod
composer install --no-dev --optimize-autoloader
```

### 3.5 Set Permissions

```bash
# Staging
cd /var/www/sti-api
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache

# Production
cd /var/www/sti-api-prod
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```

---

## 4. Environment Configuration

### 4.1 Create .env File (Staging)

```bash
cd /var/www/sti-api
cp .env.example .env
nano .env
```

**Key Environment Variables:**

```env
APP_NAME="STI API"
APP_ENV=staging
APP_KEY=
APP_DEBUG=false
APP_URL=https://api-staging.example.com

LOG_CHANNEL=stack
LOG_LEVEL=error

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=sti_api_staging
DB_USERNAME=sti_api_user
DB_PASSWORD=your_database_password

BROADCAST_DRIVER=log
CACHE_DRIVER=file
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

# Sanctum
SANCTUM_STATEFUL_DOMAINS=api-staging.example.com

# UDI API Integration
UDI_API=https://udi-api.example.com
KOOR_USERNAME=your_udi_username
KOOR_PASSWORD=your_udi_password

# Firebase FCM
FIREBASE_CREDENTIALS=/var/www/sti-api/storage/firebase-credentials.json

# Google OAuth
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_REDIRECT_URI=https://api-staging.example.com/api/web/v1/ta/auth/google/callback
```

### 4.2 Create .env File (Production)

```bash
cd /var/www/sti-api-prod
cp .env.example .env
nano .env
```

**Key Environment Variables (Production):**

```env
APP_NAME="STI API"
APP_ENV=production
APP_KEY=
APP_DEBUG=false
APP_URL=https://api.example.com

# ... (same structure as staging, but with production values)
DB_DATABASE=sti_api_prod
SANCTUM_STATEFUL_DOMAINS=api.example.com
GOOGLE_REDIRECT_URI=https://api.example.com/api/web/v1/ta/auth/google/callback
```

### 4.3 Generate Application Key

```bash
# Staging
cd /var/www/sti-api
php8.3 artisan key:generate

# Production
cd /var/www/sti-api-prod
php8.3 artisan key:generate
```

### 4.4 Run Migrations

```bash
# Staging
cd /var/www/sti-api
php8.3 artisan migrate --force

# Production
cd /var/www/sti-api-prod
php8.3 artisan migrate --force
```

### 4.5 Create Storage Link

```bash
# Staging
cd /var/www/sti-api
php8.3 artisan storage:link

# Production
cd /var/www/sti-api-prod
php8.3 artisan storage:link
```

### 4.6 Optimize Application

```bash
# Staging
cd /var/www/sti-api
php8.3 artisan config:cache
php8.3 artisan route:cache
php8.3 artisan view:cache

# Production
cd /var/www/sti-api-prod
php8.3 artisan config:cache
php8.3 artisan route:cache
php8.3 artisan view:cache
```

---

## 5. CI/CD Setup

### 5.1 GitHub Secrets Configuration

Di GitHub repository, buka **Settings → Secrets and variables → Actions**, lalu tambahkan secrets berikut:

**Required Secrets:**

1. **SERVER_IP**
   - IP address server production
   - Example: `192.168.1.100` atau `api.example.com`

2. **SSH_USER**
   - Username untuk SSH access
   - Example: `deploy` atau `ubuntu`

3. **SSH_KEY**
   - Private SSH key untuk authentication
   - Generate dengan: `ssh-keygen -t rsa -b 4096 -C "github-actions"`
   - Copy private key (id_rsa) ke GitHub Secrets
   - Copy public key (id_rsa.pub) ke server: `~/.ssh/authorized_keys`

### 5.2 Generate SSH Key for GitHub Actions

**On Local Machine:**

```bash
# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "github-actions-deploy" -f ~/.ssh/github_actions_deploy

# Copy public key to server
ssh-copy-id -i ~/.ssh/github_actions_deploy.pub user@your-server-ip

# Or manually add to server
cat ~/.ssh/github_actions_deploy.pub
# Copy output and add to ~/.ssh/authorized_keys on server
```

**On Server:**

```bash
# Add public key to authorized_keys
nano ~/.ssh/authorized_keys
# Paste the public key content
chmod 600 ~/.ssh/authorized_keys
```

**Add to GitHub Secrets:**

1. Copy private key content:
   ```bash
   cat ~/.ssh/github_actions_deploy
   ```

2. Go to GitHub → Repository → Settings → Secrets → New repository secret
3. Name: `SSH_KEY`
4. Value: Paste the entire private key content (including `-----BEGIN RSA PRIVATE KEY-----` and `-----END RSA PRIVATE KEY-----`)

### 5.3 Configure GitHub Actions Workflow

File `.github/workflows/main.yml` sudah dikonfigurasi. Pastikan:

1. Branch yang di-deploy: `staging` dan `production`
2. Deployment directories sesuai:
   - Staging: `/var/www/sti-api`
   - Production: `/var/www/sti-api-prod`

**Optional: Add Queue Worker Restart to CI/CD**

Jika ingin queue worker otomatis restart setelah deployment, tambahkan ke `.github/workflows/main.yml` sebelum `Restarting web server`:

```yaml
echo "Restarting queue worker..."
if [ "${{ github.ref }}" = "refs/heads/staging" ]; then
  /usr/local/bin/restart-queue-staging.sh || echo "Queue worker restart failed or not configured"
elif [ "${{ github.ref }}" = "refs/heads/production" ]; then
  /usr/local/bin/restart-queue-prod.sh || echo "Queue worker restart failed or not configured"
fi
```

**Note:** Pastikan script queue worker sudah dibuat dan executable sebelum update CI/CD.

### 5.4 Test CI/CD Connection

**Test SSH Connection:**

```bash
# From local machine, test SSH connection
ssh -i ~/.ssh/github_actions_deploy user@your-server-ip
```

**Test GitHub Actions:**

1. Push ke branch `staging`:
   ```bash
   git checkout staging
   git push origin staging
   ```

2. Check GitHub Actions tab untuk melihat deployment status

---

## 6. Deployment Process

### 6.1 Automatic Deployment (CI/CD)

**Deploy to Staging:**

```bash
# Merge changes to staging branch
git checkout staging
git merge feature/your-feature
git push origin staging
```

GitHub Actions akan otomatis:
1. Checkout code
2. SSH ke server
3. Pull latest code dari branch staging
4. Install composer dependencies
5. Clear cache
6. Run migrations
7. Restart queue worker (jika script tersedia)
8. Restart Nginx

**Deploy to Production:**

```bash
# Merge staging to production
git checkout production
git merge staging
git push origin production
```

GitHub Actions akan otomatis deploy ke production dengan proses yang sama.

### 6.2 Manual Deployment (If Needed)

**Staging:**

```bash
cd /var/www/sti-api
git fetch origin
git reset --hard origin/staging

# Install dependencies
composer install --no-interaction --prefer-dist --optimize-autoloader

# Clear cache
php8.3 artisan optimize:clear

# Run migrations
php8.3 artisan migrate --force

# Cache config and routes
php8.3 artisan config:cache
php8.3 artisan route:cache
php8.3 artisan view:cache

# Restart queue worker
sudo /usr/local/bin/restart-queue-staging.sh

# Restart services
sudo systemctl restart php8.3-fpm
sudo systemctl restart nginx
```

**Production:**

```bash
cd /var/www/sti-api-prod
git fetch origin
git reset --hard origin/production

# Install dependencies
composer install --no-interaction --prefer-dist --optimize-autoloader

# Clear cache
php8.3 artisan optimize:clear

# Run migrations
php8.3 artisan migrate --force

# Cache config and routes
php8.3 artisan config:cache
php8.3 artisan route:cache
php8.3 artisan view:cache

# Restart queue worker
sudo /usr/local/bin/restart-queue-prod.sh

# Restart services
sudo systemctl restart php8.3-fpm
sudo systemctl restart nginx
```

---

## 7. Post-Deployment

### 7.1 Configure Nginx

**Staging Nginx Configuration:**

```bash
root@srv560687:/etc/nginx/sites-available# nano api-dev-sti.dinus.id
```
```
server {
    server_name api-dev-sti.dinus.id;
    root /var/www/sti-api/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm-sti-api.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_read_timeout 1800s;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api-dev-sti.dinus.id/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api-dev-sti.dinus.id/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
server {
    if ($host = api-dev-sti.dinus.id) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name api-dev-sti.dinus.id;
    return 404; # managed by Certbot

}
```


**Production Nginx Configuration:**
```bash
root@srv560687:/etc/nginx/sites-available# nano api-sti.dinus.id
```

```nginx                                                         
server {
    server_name api-sti.dinus.id;
    root /var/www/sti-api-prod/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm-sti-api.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_read_timeout 1800s;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api-sti.dinus.id/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api-sti.dinus.id/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
server {
    if ($host = api-sti.dinus.id) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name api-sti.dinus.id;
    return 404; # managed by Certbot
}
```

**Enable Sites:**

```bash
# Enable staging
sudo ln -s /etc/nginx/sites-available/sti-api-staging /etc/nginx/sites-enabled/

# Enable production
sudo ln -s /etc/nginx/sites-available/sti-api-prod /etc/nginx/sites-enabled/

# Test Nginx configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

### 7.2 Setup SSL Certificate (Let's Encrypt)

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Get certificate for staging
sudo certbot --nginx -d api-staging.example.com

# Get certificate for production
sudo certbot --nginx -d api.example.com

# Auto-renewal (already configured by certbot)
sudo certbot renew --dry-run
```

### 7.3 Setup Queue Worker

**Option 1: Manual Queue Worker Script (Recommended)**

Buat script untuk manage queue worker:

**For Production:**

```bash
sudo nano /usr/local/bin/restart-queue-prod.sh
```

```bash
#!/bin/bash

# Production Queue Worker Restart Script
APP_DIR="/var/www/sti-api-prod"
PHP_BIN="/usr/bin/php8.3"
LOG_FILE="$APP_DIR/storage/logs/queue.log"

cd $APP_DIR

# Find and kill existing queue worker
QUEUE_PID=$(ps aux | grep "[q]ueue:work" | grep "$APP_DIR" | awk '{print $2}')

if [ ! -z "$QUEUE_PID" ]; then
    echo "Stopping existing queue worker (PID: $QUEUE_PID)..."
    kill $QUEUE_PID
    sleep 2
    
    # Force kill if still running
    if ps -p $QUEUE_PID > /dev/null 2>&1; then
        kill -9 $QUEUE_PID
    fi
fi

# Clear Laravel cache
echo "Clearing Laravel cache..."
$PHP_BIN artisan config:clear
$PHP_BIN artisan cache:clear
$PHP_BIN artisan route:clear
$PHP_BIN artisan view:clear

# Start new queue worker
echo "Starting new queue worker..."
nohup $PHP_BIN $APP_DIR/artisan queue:work --sleep=3 --tries=3 > $LOG_FILE 2>&1 &

# Wait a moment and check if started
sleep 2
NEW_PID=$(ps aux | grep "[q]ueue:work" | grep "$APP_DIR" | awk '{print $2}')

if [ ! -z "$NEW_PID" ]; then
    echo "Queue worker started successfully (PID: $NEW_PID)"
else
    echo "ERROR: Queue worker failed to start!"
    exit 1
fi

# Show running queue workers
echo "Running queue workers:"
ps aux | grep "queue:work" | grep -v grep
```

**For Staging:**

```bash
sudo nano /usr/local/bin/restart-queue-staging.sh
```

```bash
#!/bin/bash

# Staging Queue Worker Restart Script
APP_DIR="/var/www/sti-api"
PHP_BIN="/usr/bin/php8.3"
LOG_FILE="$APP_DIR/storage/logs/queue.log"

cd $APP_DIR

# Find and kill existing queue worker
QUEUE_PID=$(ps aux | grep "[q]ueue:work" | grep "$APP_DIR" | awk '{print $2}')

if [ ! -z "$QUEUE_PID" ]; then
    echo "Stopping existing queue worker (PID: $QUEUE_PID)..."
    kill $QUEUE_PID
    sleep 2
    
    # Force kill if still running
    if ps -p $QUEUE_PID > /dev/null 2>&1; then
        kill -9 $QUEUE_PID
    fi
fi

# Clear Laravel cache
echo "Clearing Laravel cache..."
$PHP_BIN artisan config:clear
$PHP_BIN artisan cache:clear
$PHP_BIN artisan route:clear
$PHP_BIN artisan view:clear

# Start new queue worker
echo "Starting new queue worker..."
nohup $PHP_BIN $APP_DIR/artisan queue:work --sleep=3 --tries=3 > $LOG_FILE 2>&1 &

# Wait a moment and check if started
sleep 2
NEW_PID=$(ps aux | grep "[q]ueue:work" | grep "$APP_DIR" | awk '{print $2}')

if [ ! -z "$NEW_PID" ]; then
    echo "Queue worker started successfully (PID: $NEW_PID)"
else
    echo "ERROR: Queue worker failed to start!"
    exit 1
fi

# Show running queue workers
echo "Running queue workers:"
ps aux | grep "queue:work" | grep -v grep
```

**Make scripts executable:**

```bash
sudo chmod +x /usr/local/bin/restart-queue-prod.sh
sudo chmod +x /usr/local/bin/restart-queue-staging.sh
```

**Usage:**

```bash
# Restart production queue worker
sudo /usr/local/bin/restart-queue-prod.sh

# Restart staging queue worker
sudo /usr/local/bin/restart-queue-staging.sh
```

**Option 2: Supervisor (Alternative)**

Jika ingin menggunakan Supervisor untuk auto-restart:

```bash
# Install Supervisor
sudo apt install -y supervisor

# Create config file for production
sudo nano /etc/supervisor/conf.d/sti-api-prod-queue.conf
```

```ini
[program:sti-api-prod-queue-worker]
process_name=%(program_name)s_%(process_num)02d
command=/usr/bin/php8.3 /var/www/sti-api-prod/artisan queue:work --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=1
redirect_stderr=true
stdout_logfile=/var/www/sti-api-prod/storage/logs/queue-worker.log
stopwaitsecs=3600
```

```bash
# Create config file for staging
sudo nano /etc/supervisor/conf.d/sti-api-staging-queue.conf
```

```ini
[program:sti-api-staging-queue-worker]
process_name=%(program_name)s_%(process_num)02d
command=/usr/bin/php8.3 /var/www/sti-api/artisan queue:work --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=1
redirect_stderr=true
stdout_logfile=/var/www/sti-api/storage/logs/queue-worker.log
stopwaitsecs=3600
```

```bash
# Reload Supervisor
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start sti-api-prod-queue-worker:*
sudo supervisorctl start sti-api-staging-queue-worker:*
```

### 7.4 Setup Cron Scheduler

```bash
# Edit crontab
sudo crontab -e -u www-data
```

Add Laravel scheduler untuk staging dan production:

```cron
# Staging scheduler
* * * * * cd /var/www/sti-api && /usr/bin/php8.3 artisan schedule:run >> /dev/null 2>&1

# Production scheduler
* * * * * cd /var/www/sti-api-prod && /usr/bin/php8.3 artisan schedule:run >> /dev/null 2>&1
```

**Note:** Jika menggunakan queue worker script manual, pastikan untuk restart queue worker setelah deployment. Update CI/CD workflow atau tambahkan ke deployment script.

### 7.5 Setup Log Rotation

```bash
sudo nano /etc/logrotate.d/sti-api
```

```
/var/www/sti-api/storage/logs/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data www-data
    sharedscripts
    postrotate
        systemctl reload php8.3-fpm > /dev/null 2>&1 || true
    endscript
}
```

---

## 8. Troubleshooting

### 8.1 Common Issues

**Issue: 500 Internal Server Error**

```bash
# Check Nginx error logs
sudo tail -f /var/log/nginx/sti-api-error.log

# Check Laravel logs
tail -f /var/www/sti-api/storage/logs/laravel.log

# Check PHP-FPM logs
sudo tail -f /var/log/php8.3-fpm.log

# Check permissions
sudo chown -R www-data:www-data /var/www/sti-api/storage
sudo chmod -R 775 /var/www/sti-api/storage
```

**Issue: Permission Denied**

```bash
# Fix ownership
sudo chown -R www-data:www-data /var/www/sti-api
sudo chown -R www-data:www-data /var/www/sti-api-prod

# Fix permissions
sudo chmod -R 755 /var/www/sti-api
sudo chmod -R 755 /var/www/sti-api-prod
sudo chmod -R 775 /var/www/sti-api/storage
sudo chmod -R 775 /var/www/sti-api-prod/storage
```

**Issue: Database Connection Error**

```bash
# Test MySQL connection
mysql -u sti_api_user -p sti_api_staging

# Check MySQL service
sudo systemctl status mysql

# Check .env database credentials
cat /var/www/sti-api/.env | grep DB_
```

**Issue: Composer Dependencies Error**

```bash
# Clear composer cache
composer clear-cache

# Reinstall dependencies
composer install --no-interaction --prefer-dist --optimize-autoloader
```

**Issue: Migration Fails**

```bash
# Check migration status
php8.3 artisan migrate:status

# Rollback last migration
php8.3 artisan migrate:rollback

# Run migrations with force
php8.3 artisan migrate --force
```

### 8.2 Performance Optimization

**Enable OPcache:**

```bash
sudo nano /etc/php/8.3/fpm/php.ini
```

```ini
opcache.enable=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.revalidate_freq=2
opcache.fast_shutdown=1
```

**Restart PHP-FPM:**

```bash
sudo systemctl restart php8.3-fpm
```

### 8.3 Monitoring

**Check Service Status:**

```bash
# Check all services
sudo systemctl status nginx
sudo systemctl status php8.3-fpm
sudo systemctl status mysql

# Check disk space
df -h

# Check memory usage
free -h

# Check PHP-FPM processes
ps aux | grep php-fpm
```

**Monitor Logs:**

```bash
# Real-time log monitoring
sudo tail -f /var/log/nginx/sti-api-error.log
tail -f /var/www/sti-api/storage/logs/laravel.log
```

---

## 9. Backup Strategy

### 9.1 Database Backup

**Create Backup Script:**

```bash
sudo nano /usr/local/bin/backup-sti-api-db.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/var/backups/sti-api"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="sti_api_prod"
DB_USER="sti_api_user"
DB_PASS="your_password"

mkdir -p $BACKUP_DIR

# Backup database
mysqldump -u $DB_USER -p$DB_PASS $DB_NAME | gzip > $BACKUP_DIR/db_$DATE.sql.gz

# Keep only last 7 days
find $BACKUP_DIR -name "db_*.sql.gz" -mtime +7 -delete

echo "Backup completed: db_$DATE.sql.gz"
```

**Make executable:**

```bash
sudo chmod +x /usr/local/bin/backup-sti-api-db.sh
```

**Add to crontab:**

```bash
sudo crontab -e
```

```cron
# Daily backup at 2 AM
0 2 * * * /usr/local/bin/backup-sti-api-db.sh
```

### 9.2 Application Files Backup

```bash
# Backup storage files
sudo tar -czf /var/backups/sti-api/storage_$(date +%Y%m%d).tar.gz /var/www/sti-api-prod/storage/app
```

---

## 10. Security Checklist

- [ ] SSL certificate installed and auto-renewal configured
- [ ] Firewall configured (UFW)
- [ ] SSH key authentication only (disable password auth)
- [ ] Regular security updates enabled
- [ ] Database user has minimal required privileges
- [ ] `.env` file has correct permissions (600)
- [ ] `APP_DEBUG=false` in production
- [ ] Strong database passwords
- [ ] Regular backups configured
- [ ] Log rotation configured
- [ ] Nginx security headers configured
- [ ] PHP-FPM security settings configured

---

## 11. Quick Reference

### 11.1 Deployment Commands

```bash
# Quick deploy to staging (with queue worker restart)
cd /var/www/sti-api && git pull origin staging && composer install --no-dev --optimize-autoloader && php8.3 artisan migrate --force && php8.3 artisan optimize:clear && sudo /usr/local/bin/restart-queue-staging.sh && sudo systemctl restart php8.3-fpm nginx

# Quick deploy to production (with queue worker restart)
cd /var/www/sti-api-prod && git pull origin production && composer install --no-dev --optimize-autoloader && php8.3 artisan migrate --force && php8.3 artisan optimize:clear && sudo /usr/local/bin/restart-queue-prod.sh && sudo systemctl restart php8.3-fpm nginx
```

### 11.2 Useful Commands

```bash
# Clear all caches
php8.3 artisan optimize:clear

# Clear individual caches
php8.3 artisan config:clear
php8.3 artisan cache:clear
php8.3 artisan route:clear
php8.3 artisan view:clear

# Cache everything
php8.3 artisan config:cache && php8.3 artisan route:cache && php8.3 artisan view:cache

# Check queue status
php8.3 artisan queue:work --once

# Run scheduler manually
php8.3 artisan schedule:run

# Check queue:listen processes
ps aux | grep "queue:work"
```

### 11.3 Queue Worker Management

```bash
# Check running queue workers
ps aux | grep "queue:work" | grep -v grep

# Restart staging queue worker
sudo /usr/local/bin/restart-queue-staging.sh

# Restart production queue worker
sudo /usr/local/bin/restart-queue-prod.sh

# Manual kill queue worker (if needed)
# Find PID first
ps aux | grep "queue:work" | grep "/var/www/sti-api"
# Kill by PID
kill <PID>

# View queue logs
tail -f /var/www/sti-api/storage/logs/queue.log
tail -f /var/www/sti-api-prod/storage/logs/queue.log
```

---

**Last Updated:** December 2025  
**Version:** 1.0

