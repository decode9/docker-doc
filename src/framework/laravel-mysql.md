# Laravel API + MYSQL / MariaDB

Para dockerizar una aplicacion de laravel API con postgreSQL se debe crear un Dockerfile para la aplicacion PHP con las dependencias necesarias en el sistema.

## Dockerfile Aplicacion PHP

Para la dockerizacion de una aplicacion laravel en php por favor siga las intrucciones hasta el titulo "Archivos de configuracion para los entornos de aplicacion" debido a que la configuracion es la misma hasta ese punto en la pagina de [Laravel API + PostgreSQL](laravel-postgres)

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
        # SE UTILIZA IMAGEN DE MYSQL o MARIADB
        image: mysql/mariadb
        restart: always
        environment:
            # SE DECLARA LA BASE DE DATOS
            MYSQL_DATABASE: micontenedor
            # SE DECLARA LA CONTRASENA DEL USUARIO ROOT
            MYSQL_ROOT_PASSWORD: password
        tty: true
        ports:
            - "3310:3306"
        volumes:
            # SE DECLARA LA PERSISTENCIAS DE LOS DATOS MYSQL
            - micontenedor-data:/var/lib/mysql
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

DB_CONNECTION=mysql
DB_HOST=micontenedor-db
DB_PORT=3306
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

## Configuracion de la base de datos

Antes de ejecutar la configuracion del entorno laravel se debe preconfigurar el usuario de base de datos para la red correspondiente.

Se ejecuta la linea de comandos del contenedor de base de datos
``` bash
docker exec -it micontenedor-db sh
```

En la terminal del contenedor se ejecuta los comandos para acceder a la base de datos
``` sh
mysql -u root -p
# La CONTRASENA NO APARECE EN CONSOLA SE INTRODUCE DE FORMA OCULTA
# SE UTILIZA LA CONTRASENA PUESTA EN EL ARCHIVO docker-compose.yml
password: password
```

Se crea el usuario de para la aplicacion en el PHP-FPM server y se le otorgan los permisos a la base de datos

``` mariadb
CREATE USER 'usuario'@'200.15.15.20' IDENTIFIED BY 'contrasena';
GRANT ALL PRIVILEGES ON micontenedor.* TO 'usuario'@'200.15.15.20';
FLUSH PRIVILEGES:
```

Una vez creado el usuario se dispone a realizar la configuacion del entorno en el servidor PHP-FPM, Para una ejecucion mas veloz al momento de cambiar de entorno se crea un archivo create_user.sql con las instrucciones SQL y un archivo build_database.sh para Linus y build_database.bat para Windows con el siguiente comando

```sh
docker exec -i mysql -u root -ppassword < create_user.sql
```

Una vez ejecutado el comando estara listo nuestro usuario en la base de datos.

### Comandos para configuracion de laravel en docker

Se ejecutan los comandos o el archivo explicado en la entrada [Laravel API + POSTGRESQL](laravel-postgres#comandos-para-configuracion-de-laravel-en-docker) del mismo titulo

Una vez ejecutados los comandos nuestra aplicacion estara lista para funcionar en docker


## Ejecutar docker-compose.yml

Para ejecutar el archivo docker-compose.yml se ejecuta el comando

``` bash
docker-compose up
```
