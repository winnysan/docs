# VPS Laravel

### vytvorenie

```
ssh root@<ip-servera>

adduser <user>
usermod -aG sudo <user>

apt update && apt upgrade -y
reboot -f
```

pripojenie

```
ssh <user>@<ip-servera>
```

### pripojenie na VPS cez SSH kluc

vytvorenie kluca

```
ssh-keygen -t ed25519 -C "<mail@example.com>"
```

nahratie verejneho kluca na VPS

```
ssh-copy-id <user>@<ip-servera>
```

alebo manualne

```
cat ~/.ssh/id_ed25519.pub | ssh <user>@<ip-servera> "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

pripojenie

```
ssh <user>@<ip-servera>
ssh -i ~/.ssh/id_ed25519 <user>@<ip-servera>
```

### instalovanie

```
sudo apt install mc -y
sudo apt install htop -y
sudo apt install ufw -y
sudo apt install net-tools -y
```

zobrazenie pocuvajucich portov

```
sudo netstat -tunlp
```

nastavenie firewallu

```
sudo ufw app list
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw status
```

### PHP

instalacia

```
sudo apt install php-fpm php-cli -y

sudo apt install -y php8.3-common php8.3-mysql php8.3-zip php8.3-gd php8.3-mbstring php8.3-curl php8.3-xml php8.3-bcmath php8.3-tokenizer php8.3-sqlite3 php8.3-pgsql
```

### Composer

instalacia

```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'.PHP_EOL; } else { echo 'Installer corrupt'.PHP_EOL; unlink('composer-setup.php'); exit(1); }"
php composer-setup.php
php -r "unlink('composer-setup.php');"

sudo mv composer.phar /usr/local/bin/composer
```

### Node

instalacia

```
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
apt-get install nodejs -y
```

### Git

instalacia

```
sudo apt install git
git config --global user.name "<username>"
git config --global user.email "<mail@example.com>"
```

vytvorenie SSH kluca a pridanie do SSH agenta

```
ssh-keygen -t ed25519 -C "<mail@example.com>"
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519
```

pridanie verejneho kluca na Github

```
cat ~/.ssh/id_ed25519.pub

Settings
SSH and GPG keys
New SSH key
```

### Nginx

instalacia

```
sudo apt install nginx -y
```

povolit na firewale

```
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
```

vlastnik `www` a symlinky

```
sudo chown -R $USER:www-data /var/www
sudo chmod 755 /var/www
ln -s /var/www ~/www

sudo ln -s /etc/nginx/sites-enabled ~/sites-enabled
```

### Certbot

instalacia

```
sudo apt install certbot python3-certbot-nginx -y
```

vygenerovanie certifikatu, sam prida do nginx

```
sudo certbot --nginx -d <example.com>
```

umiestnenie certifikatu a kluca

```
/etc/letsencrypt/live/<example.com>/fullchain.pem
/etc/letsencrypt/live/<example.com>/privkey.pem
```

### nastavenie Nginx pre Laravel

upravit konfiguraciu `/etc/nginx/sites-enabled/default.conf`

```
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name <example.com>;
    root /var/www/<example.com>/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    ssl_certificate /etc/letsencrypt/live/<example.com>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<example.com>/privkey.pem;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}

server {
    listen 80;
    listen [::]:80;
    server_name <example.com>;

    return 301 https://$host$request_uri;
}
```

kontrola a restart

```
sudo nginx -t
sudo systemctl restart nginx
```

### Laravel

vytvorenie `.env` a instalacia

```
cp .env.example .env

composer install
npm install

php artisan key:generate
php artisan storage:link
php artisan migrate
```

nastavenie opravneni v `root` zlozke

```
sudo chown -R $USER:www-data .
find . -type f ! -name '.gitignore' -exec chmod 664 {} \;
find . -type d -exec chmod 775 {} \;
sudo chgrp -R www-data ./storage ./bootstrap/cache
sudo chmod -R ug+rwx ./storage ./bootstrap/cache
```

### Meilisearch

instalacia

```
curl -L https://github.com/meilisearch/meilisearch/releases/download/v1.14.0/meilisearch-linux-amd64 -o meilisearch
chmod +x meilisearch
sudo mv meilisearch /usr/local/bin/
```

vytvorenie pouzivatela pre meilisearch

```
sudo useradd -r -s /bin/false meilisearch
```

vytvorenie sluzby

```
sudo nano /etc/systemd/system/meilisearch.service

[Unit]
Description=Meilisearch
After=network.target

[Service]
ExecStart=/usr/local/bin/meilisearch --env production --http-addr 127.0.0.1:7700 --master-key=meilisearch-secret-key --db-path=/var/lib/meilisearch
User=meilisearch
WorkingDirectory=/var/lib/meilisearch
Restart=always
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target

```

vytvorenie datoveho adresara

```
sudo mkdir -p /var/lib/meilisearch
sudo chown meilisearch:meilisearch /var/lib/meilisearch
```

spustenie a povolenie sluzby

```
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable meilisearch
sudo systemctl start meilisearch
sudo systemctl status meilisearch
```

### PostgreSQL

instalacia

```
sudo apt install postgresql postgresql-contrib -y
```

vytvorenie databazy a pouzivatela

```
sudo -u postgres psql

CREATE USER <database-username> WITH PASSWORD '<database-password>';
CREATE DATABASE <database-name> OWNER <database-username>;
GRANT ALL PRIVILEGES ON DATABASE <database-name> TO <database-username>;
\q
```

nastavenie `.env`

```
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=<database-name>
DB_USERNAME=<database-username>
DB_PASSWORD=<database-password>
```

### pripojenie k databazi cez SSH tunel

v `TablePlus` nastavit hodnoty z `.env` a pripojit `Over SSH`

```
Host: 127.0.0.1
Port: 5432
User: <database-username>
Password: <database-password>
Database: <database-name>

Over SSH
Server: <ip-servera>
Port: 22
User: <user>
Password: prazdne
Use SSH key: ~/.ssh/id_25519
```
