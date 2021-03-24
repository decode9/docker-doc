---
sidebar: auto
---
# Introduccion

Docker es un sistema que automatiza el despliegue de aplicaciones creando contenedores con las dependencias necesarias para ejecutarlas, en palabras mas simple, simula el entorno que se configura en un servidor o equipo fisico o virtual para ejecutar una aplicacion especifica.

[[toc]]

## Instalacion de docker

Para instalar docker es necesario descargar el instalador del sistema, este existe en windows, linux y mac.

Las instrucciones vienen en el link de cada plataforma

### Linux UBUNTU

```
https://docs.docker.com/engine/install/ubuntu/
```

### Windows

```
https://hub.docker.com/editions/community/docker-ce-desktop-windows
```

Para el correcto funcionamiento de docker en windows ejecutan los siguientes comandos en una consola powershell con permisos de administrador

``` powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

Una vez activadas ambas funcionalidades se procede a instalar Ubuntu 20.04 y se reinicia el equipo.

### Mac

``` http
https://hub.docker.com/editions/community/docker-ce-desktop-mac
```

## Comandos Basicos de docker

Para el manejo de docker se utilizaran los comandos docker y docker-compose a traves de la consola de comandos.

para ver las diferentes opciones que se puede utilizar la instruccion help en cada comando o instruccion

### docker
Las instrucciones que se utilizaran del comando docker.

1. container
    - Esta instruccion permite la manipulacion de los contenedores en docker
2. image
    - Esta instruccion permite la manipulacion de las imagenes en docker
3. network
    - Esta instruccion permite la manipulacion de las redes creadas en docker
4. volume
    - Permite la manipulacion de los volumenes de informacion en docker
5. exec
    - Permite Ejecutar un comando dentro de un contenedor de docker
6. rm
    - Permite Eliminar un contenedor
7. create
    - Permite crear un contenedor
8. start
    - Permite iniciar un contenedor
9. stop
    - Permite detener un contenedor
10. ps
    - Lista los contenedores creados

### docker-compose
Las instrucciones que se utilizaran del comando docker-compose.

1. up
    - Esta instruccion permite el levantamiento de los servicios especificados en un documento docker-compose.yml
2. down
    - Esta instruccion permite la destruccion de los contenedores de los servicios especificados en un documento docker-compose.yml
3. stop
    - Esta instruccion permite detener los contenedores de los servicios especificados en el documento docker-compose.yml
4. start
    - Esta instruccion permite ejecutar los contenedores de los servicios especificados en el documento docker-compose.yml