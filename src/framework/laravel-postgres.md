# Laravel API + PostgreSQL

Para dockerizar una aplicacion de laravel API con postgreSQL se debe crear un Dockerfile para la aplicacion PHP con las dependencias necesarias en el sistema.

## Dockerfile Aplicacion PHP

Para la dockerizacion de una aplicacion PHP se declara una base de php version 7.4 con FPM y las dependencias necesarias para su ejecucion.

``` docker
# COMANDO FROM INDICA LA IMAGEN BASE DE LA NUEVA IMAGEN
FROM php:7.4-fpm

# COMANDO COPY COPIA DE LA CARPETA DEL SISTEMA A CARPETA DEL CONTENEDOR
# EN ESTE CASO SE COPIA EL COMPOSER.JSON DENTRO DE LA CARPETA BASE A 
# TRABAJAR EN EL CONTENEDOR
COPY composer.lock composer.json /var/www/

# SE ESTABLECE LA RUTA DE TRABAJO DE LOS COMANDOS CON WORKDIR
# ES LA RUTA DONDE SE EJECUTARAN LOS COMANDOS A INTRODUCIR
WORKDIR /var/www

# SE DEFINE EL VOLUMEN PARA LA PERSISTENCIAS DE LOS DATOS DE DICHA RUTA 
# DE FORMA PREDETERMINADA
VOLUME [ "/var/www" ]

# EL COMANDO RUN EJECUTA COMANDO DEL SISTEMA EN LA DISTRIBUCION BASE DEL CONTENEDOR
# SE INSTALAN LAS LIBRERIAS Y DEPENDENCIAS NECESARIAS PARA LA EJECUCION 
# DE UN AMBIENTE LARAVEL
RUN apt-get update && apt-get install -y build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    unzip \
    git \
    curl \
    vim \
    libzip-dev \
    libxml2-dev \
    libpq-dev

# SE LIMPIA LA INSTALACION DE LA LIBRERIA Y LAS CARPETA PARA EVITAR 
# ESPACIOS INNECESARIOS EN LA IMAGEN
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# SE INSTALAN LAS EXTENSIONES PHP NECESARIAS PARA LA VERSION A UTILIZAR
# UTILIZANDO EL COMANDO docker-php-ext-install
RUN docker-php-ext-install zip exif pcntl soap pdo_pgsql

# SE CONFIGURA LA EXTENSION GD PARA EL MANEJO DE IMAGENES EN PHP CON
# docker-php-ext-configure
RUN docker-php-ext-configure gd --with-gd --with-freetype-dir=/user/include \
    --with-jpeg-dir=/usr/include --with-png-dir=/usr/include

# SE INSTALA LA EXTENSION GD
RUN docker-php-ext-install gd

# SE INSTALA COMPOSER
RUN curl -sS https://getcomposer.org/installer | \
    php -- --install-dir=/usr/local/bin --filename=composer

# SE INTRODUCEN LOS PERMISOS DE USUARIO Y GRUPO
RUN chown -R www:www . /var/www

```

Este archivo se crea dentro de la carpeta raiz del proyecto Laravel en PHP con esto el sistema tendra un servidor PHP-FPM que podra ser ejecutado desde cualquier servidor web

## Archivos de configuracion para los entornos de la aplicacion

Para el correcto funcionamiento de la aplicacion en un entorno de prueba o produccion se debe configurar los archivos para el servidor web NGINX y el servidor PHP-FPM

### Archivo de configuracion app.conf para servidor web NGINX

Creamos un archivo de configuracion app.conf en la siguiente estructura de carpeta

```
- nginx
--- conf
---- app.conf
```

Este archivo llevara los siguientes valores de configuracion

```
server {
    listen 80;
    index index.php index.html;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/public;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass micontenedor-phpfpm:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_read_timeout 3600s;
    }
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
    }
}
```

Este archivo sera utilizado para la configuracion del servidor web en NGINX

### Archivo de configuracion php.ini para el servidor PHP-FPM

Crearemos un archivo php.ini en la siguiente estructura de carpetas


```
- php
--- php.ini
```

Este archivo llevara los siguientes valores de configuracion

```
upload_max_filesize=256M
post_max_size=256M
```

Este es el archivo que sera utilizado para la configuracion del servidor PHP-FPM

## Archivo docker-compose.yml para conexion con servidor web y base de datos

Una vez configurado el archivo Dockerfile y los archivos de configuracion para el entorno se procede a crear el archivo docker-compose.yml este archivo definira los servicios en diferentes contenedores y los ejecutara al mismo tiempo.

``` docker
# DECLARACION DE LA VERSION DE DOCKER
version: "3"

# DECLARACION DE SERVICIOSS
services:

    # DECLARACION DE CONTENEDOR
    micontenerdor-phpfpm:
        # RUTA DE DOCKERFILE PARA BUILD
        build: ./
        # NOMBRE DE CONTENEDOR
        container_name: micontenedor-phpfpm
        # INDICAR SI SE REINICIA EL CONTENEDOR EN CASO DE FALLA
        restart: unless-stopped
        # UTILIZACION DE CONSOLA VIRTUAL
        tty: true
        # DEFINICION DE VOLUMENES
        volumes:
            # SE DEFINEN LAS CARPETAS RELATIVAS DEL SISTEMA 
            : SE DEFINE LA CARPETA DEL CONTENEDOR A MONTAR
            - ./:/var/www
            - ./php/local.ini:/usr/local/etc/php/conf.d/local.ini
        # VARIABLES DE ENTORNO
        environment:
            SERVICE_NAME: micontenedor-phpfpm
            SERVICE_TAGS: dev
        # DECLARACION DE RED
        networks:
            # NOMBRE DE LA RED
            app-network:
                # IP DEL CONTENEDOR
                ipv4_address: 200.15.15.20
    
    # SERVIDOR WEB EN NGINX
    micontenedor-webserver:
        # IMAGEN A UTILIZAR PARA EL CONTENEDOR
        image: nginx:alpine
        container_name: micontenedor-webserver
        restart: unless-stopped
        tty: true
        # PUERTOS DE COMUNICACION PARA LOCAL : CONTENEDOR
        ports:
            # DECLARACION DE PUERTOS LOCAL:CONTENEDOR
            - "8010:80"
        volumes:
            - ./:/var/www/
            - ./nginx/conf.d/:/etc/nginx/conf.d
        networks:
            - app-network

    
    micontenedor-db:
        # SE UTILIZA IMAGEN DE POSTGRES
        image: postgres
        restart: always
        environment:
            # SE DECLARA LA BASE DE DATOS
            POSTGRES_DB: micontenedor
            # SE DECLARA LOS USUARIOS
            POSTGRES_USER: usuario
            # SE DECLARA LA CONTRASENA
            POSTGRES_PASSWORD: contrasena
        tty: true
        ports:
            - "5440:5432"
        volumes:
            - micontenedor-data:/var/lib/postgresql/data
        networks:
            - app-network

# DECLARACION DE REDES
networks:
    # DECLARACION DE RED
    app-network:
        # DRIVER
        driver: bridge
            # IPAM
            ipam:
                # CONFIGURACION
                config:
                    # SUBNET DE LA RED
                    - subnet: 200.15.15.1/24

# DECLARACION DE VOLUMENES
volumes:
    # DECLARACION DEL VOLUMEN
    micontenedor-data:
        # DRIVER DEL VOLUMEN
        driver: local

```

Una vez se encuentre el archivo docker-compose.yml creado se procedera a ejecutar los comandos para el levantamiento del sistema

## Ejecutar docker-compose.yml

Para ejecutar el archivo docker-compose.yml se ejecuta el comando

``` bash
docker-compose up
```


## Ejecutar por primera vez

Al ejecutar por primera vez la estructura de comandos de docker-compose up se debe realizar los siguientes comandos para configurar el entorno y la base de datos del sistema

### Creacion de .env

Se debe crear un archivo .env en la carpeta de la aplicacion con los siguientes datos

```
APP_NAME=Laravel
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack

DB_CONNECTION=pgsql
DB_HOST=micontenedor-db
DB_PORT=5432
DB_DATABASE=micontenedor
DB_USERNAME=usuario
DB_PASSWORD=contrasena

BROADCAST_DRIVER=redis
QUEUE_DRIVER=redis
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

MAIL_DRIVER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=micorreo@gmail.com
MAIL_PASSWORD=micontrasena
MAIL_ENCRYPTION=tls

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

```

Al tener listo nuestro archivo .env se procede a ejecutar la siguiente estructura de comandos

### Comandos para configuracion de laravel en docker

Se ejecutan los siguientes comandos en orden para la configuracion de la aplicacion en laravel

Instalar composer
```
docker exec -it micontenedor-phpfpm composer install
```

Instalar Llave de laravel
```
docker exec -it micontenedor-phpfpm php artisan key:generate
```

Migrar Base de datos
```
docker exec -it micontenedor-phpfpm php artisan migrate
```

Una vez ejecutados los comandos nuestra aplicacion estara lista para funcionar en docker, para una ejecucion mas veloz en un ambiente compartido se crea un archivo build.sh para los entornos linux y build.bat para los entornos windows con estos comandos

``` sh
docker exec -it micontenedor-phpfpm composer install
docker exec -it micontenedor-phpfpm php artisan key:generate
docker exec -it micontenedor-phpfpm php artisan migrate
```

Se otorgan permisos de ejecucion y se ejecuta el archivo

``` bash
sudo chmod 777 build.sh
./build.sh
```

Y se ejecuta los servicios con el comando docker-compose

``` bash
docker-compose up
```