# VPS Laravel (docker)

## vytvorenie

```
ssh root@<ip-servera>

adduser <user>
usermod -aG sudo <user>

apt update && apt upgrade -y
reboot -f
```

```
ssh <user>@<ip-servera>
```

## instalovanie

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

## DNS zaznamy

Typ `A`

```
Host: example.com
Typ: A
Hodnota: <ip-servera>
```

Typ `CNAME`

```
Host: *.example.com
Typ: CNAME
Hodnota: example.com
```

## docker

instalacia najnovsej verzie

```
sudo apt install curl apt-transport-https ca-certificates software-properties-common -y

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce -y

sudo systemctl enable docker --now
sudo systemctl status docker

docker version
docker compose version
```

spustanie dockeru bez `sudo` 

```
sudo usermod -aG docker $USER
newgrp
groups $USER
su - $USER
```

## testovaci projekt

symlink ku projektu

```
mkdir -p ~/project/server/public
echo "<?php echo 'test';" > ~/project/server/public/index.php
mkdir -p ~/docker
ln -s ~/project/server ~/docker/src
```

vytvorenie `docker-compose.yml`

```
cat << 'EOF' > ~/docker/docker-compose.yml
services:
  php:
    image: php:8.3-fpm
    container_name: php_fpm
    restart: unless-stopped
    volumes:
      - ./src:/var/www/html

  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./src:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - php
EOF
```

nastavenie `nginx`

```
mkdir -p ~/docker/nginx

cat << 'EOF' > ~/docker/nginx/default.conf
server {
    listen 80;
    server_name example.com;
    root /var/www/html/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ ^/index\.php(/|$) {
        fastcgi_pass php:9000;
		fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
EOF
```

vytvorenie, spustenie, zastavenie, vycistenie

```
cd docker
 
docker compose up -d --build
docker compose up
docker composer down
docker system prune -a
```

zistenie nazvu kontajnera, vstup do kontainera

```
docker ps

docker exec -it php_fpm sh
docker exec -it nginx sh
```

## SSL certifikat

vytvorenie a umiesnenie certifikatu

```
sudo apt install certbot

sudo certbot certonly --standalone \
  --agree-tos \
  --no-eff-email \
  --email info@example.com \
  -d example.com

mkdir -p ~/docker/certs

sudo cp /etc/letsencrypt/live/example.com/fullchain.pem ~/docker/certs/fullchain.pem &&
sudo cp /etc/letsencrypt/live/example.com/privkey.pem ~/docker/certs/privkey.pem
```

nastavenie `nginx` SSL s presmerovanim

```
mkdir -p ~/docker/nginx

cat << 'EOF' > ~/docker/nginx/default.conf
server {
    listen 443 ssl;
    server_name example.com;
    root /var/www/html/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    ssl_certificate /etc/nginx/certs/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/privkey.pem;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ ^/index\.php(/|$) {
        fastcgi_pass php:9000;
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
    server_name example.com;

    return 301 https://$host$request_uri;
}
EOF
```

nastavenie `docker-compose.yml` SSL

```
cat << 'EOF' > ~/docker/docker-compose.yml
services:
  php:
    image: php:8.3-fpm
    container_name: php_fpm
    restart: unless-stopped
    volumes:
      - ./src:/var/www/html

  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./src:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - php
EOF
```

testy certifikatu

```
openssl x509 -in ~/docker/certs/fullchain.pem -text -noout
curl -vk https://example.com
docker exec -it nginx ls -l /etc/nginx/certs
```

## MYSQL

vytvorenie `.env`

```
cat << 'EOF' > ~/docker/.env.example2
DB_ROOT_PASSWORD=rootpassword
DB_DATABASE=database
DB_USERNAME=username
DB_PASSWORD=password
EOF

cp ~/docker/.env.example ~/docker/.env
```

vytvorenie `Dockerfile`

```
cat << 'EOF' > ~/docker/Dockerfile
FROM php:8.3-fpm

RUN apt-get update && apt-get install -y \
    default-mysql-client \
    && docker-php-ext-install pdo_mysql
EOF
```

vytvorenie `docker-compose.yml`

```
cat << 'EOF' > ~/docker/docker-compose.yml
services:
  php:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: php_fpm
    restart: unless-stopped
    volumes:
      - ./src:/var/www/html
    depends_on:
      - mysql

  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./src:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - php

  mysql:
    image: mysql:8.0
    container_name: mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_USER: ${DB_USERNAME}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    volumes:
      - ./database:/var/lib/mysql
EOF
```

vytvorenie testovacieho `index.php`

```
cat << 'EOF' > ~/project/server/public/index.php
<?php

$host = 'mysql';
$db   = 'database';
$user = 'username';
$pass = 'password';

$dsn = "mysql:host=$host;dbname=$db;charset=utf8mb4";

try {
    $pdo = new PDO($dsn, $user, $pass);
    echo 'Spojenie s databázou je OK.';
    $pdo = null;
} catch (PDOException $e) {
    echo 'Chyba spojenia: ' . $e->getMessage();
}
EOF
```
