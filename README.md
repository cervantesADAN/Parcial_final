# **Parcial Final: Monitoreo de Servidores con Docker y Nagios**

Este proyecto implementa una solución de monitoreo para servidores críticos usando **Nagios Core** dentro de un entorno **Dockerizado**. Está diseñado para supervisar:
- Un servidor de bases de datos (PostgreSQL).
- Un servidor web (NGINX).

## **Características**
- Despliegue automatizado con Docker Compose.
- **Nagios Core** configurado para monitorear:
  - Estado de conexión de PostgreSQL.
  - Disponibilidad de puertos HTTP/HTTPS en NGINX.
  - Estado general de los servidores.
- Persistencia de datos con volúmenes Docker.
- Documentación detallada para replicar el entorno.

## **Requisitos del Entorno**
- Sistema operativo Linux (Ubuntu recomendado) con soporte para Docker.
- Conexión a Internet para descargar imágenes Docker oficiales.
- Permisos administrativos para ejecutar comandos `sudo`.
## link del video de prueba:
https://drive.google.com/file/d/1sC2daEkOoWQ8r3F5DXc8wGYEqgYkvKUd/view?usp=sharing
---


## **Instrucciones de Instalación**

### **1. Instalación de Docker y Docker Compose**
- Actualizar paquetes del sistema e instalar Docker y Docker Compose:
  ```bash
  sudo apt update && sudo apt upgrade -y
  sudo apt install docker.io -y
  sudo apt install docker-compose -y
  sudo systemctl enable docker
  sudo systemctl start docker
 
### **2. Despliegue de Nagios**
### - 1. Crear una carpeta para el proyecto de Nagios:
  
   mkdir ~/nagios-docker

   cd ~/nagios-docker

### - ii. Crear el archivo docker-compose.yml con el siguiente contenido:

   version: '3'

   services:

     nagios:

       image: jasonrivers/nagios:latest

       container_name: nagios

       ports:

         - "8080:80"

       volumes:

         - ./nagios/etc:/opt/nagios/etc
  
         - ./nagios/var:/opt/nagios/var
  
         - ./nagios/libexec:/opt/nagios/libexec
  
       restart: unless-stopped
  
### - iii. Crear las carpetas necesarias para la configuración de Nagios:
 - mkdir -p nagios/etc nagios/var nagios/libexec

### Iniciar Nagios con Docker Compose:

 - docker-compose up -d

 - Acceder a la interfaz web de Nagios en: http://<IP-de-tu-maquina>:8080

### 3. Configuración del Servidor de Bases de Datos (PostgreSQL)
### -Crear una carpeta para el proyecto:
 - mkdir ~/docker-db
 
 - cd ~/docker-db
   
### Crear el archivo docker-compose.yml:
version: '3.3'

services:

  postgres:
  
    image: postgres:latest
    
    container_name: postgres_container
    
    environment:
    
      POSTGRES_USER: usuario
      
      POSTGRES_PASSWORD: 12345
      
      POSTGRES_DB: empresa
      
    ports:
    
      - "5432:5432"
      
    volumes:
    
      - postgres_data:/var/lib/postgresql/data
      

  pgadmin:
  
    image: dpage/pgadmin4
    
    container_name: pgadmin
    
    environment:
    
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      
      PGADMIN_DEFAULT_PASSWORD: admin_password
      
    ports:
    
      - "5000:80"
      
    volumes:
    
      - pgadmin_data:/var/lib/pgadmin
      

volumes:

  postgres_data:
  
  pgadmin_data:
  
### Iniciar los contenedores:

-docker-compose up -d

### Acceder a pgAdmin en tu navegador:
 - http://<IP-de-tu-maquina>:5000

### 4. Configuración del Servidor NGINX
 - Crear una carpeta para NGINX:
  
- mkdir ~/docker-nginx
 
- cd ~/docker-nginx
  
### Crear el archivo Dockerfile para NGINX:

dockerfile

   -FROM nginx:latest
   
   -COPY ./default.conf /etc/nginx/conf.d/default.conf
   
   -Construir la imagen y ejecutar el contenedor:
   

   -docker build -t img_dk_nginx .
   
   -docker run -d --name cont_nginx -p 80:80 img_dk_nginx

   -NAcceder al servidor en tu navegador:

   - http://<IP-del-servidor>


5. Configuración de Monitoreo en Nagios
Acceder al contenedor de Nagios:

-   docker exec -it nagios bash
  
-   Dirigirse al directorio de configuración:

-   cd /opt/nagios/etc/objects
- 
  ### Crear el archivo de configuración para el servicio (ejemplo para PostgreSQL):
- nano mv-bd.cfg
  
- Contenido:

- plaintext
  
define host {

    use                     linux-server
    
    host_name               mv-bd
    
    alias                   Database Server
    
    address                 <IP-DE-MV-BD>
  
}

define service {

    use                     generic-service
    
    host_name               mv-bd
    
    service_description     PostgreSQL Connection
    
    check_command           check_nrpe!check_postgresql
    
}

### Actualizar el archivo principal de nagios.cfg para incluir las nuevas configuraciones:

- nano /opt/nagios/etc/nagios.cfg
  
- Agregar:
  
- cfg_file=/opt/nagios/etc/objects/mv-bd.cfg

### Validar y Reiniciar Nagios
- nagios -v /opt/nagios/etc/nagios.cfg
  
- Si no hay errores, reinicia Nagios:
### Pasos para Configurar el Cliente de Nagios (NRPE) en mv-bd
1. Instalar NRPE y los Plugins de Nagios en mv-bd

### Conéctate a mv-bd y actualiza los paquetes:

- sudo apt update

### Instala el cliente NRPE y los plugins de Nagios:

- Instala el cliente NRPE y los plugins de Nagios:

2\.Configurar NRPE en mv-bd

### Edita el archivo de configuración de NRPE:

- sudo nano /etc/nagios/nrpe.cfg

### Realiza los siguientes cambios:

### Permitir conexiones desde el servidor Nagios:

### Busca la línea que comienza con allowed\_hosts y agrega la IP del servidor Nagios 	(mv-nagios). Ejemplo:

`	`-allowed\_hosts=127.0.0.1,192.168.1.40

### Agregar comandos personalizados para monitoreo:

`	`-command[check\_mysql]=/usr/lib/nagios/plugins/check\_tcp -H 127.0.0.1 -p 3306

`	`-command[check\_disk]=/usr/lib/nagios/plugins/check\_disk -w 20% -c 10% -p /

`	`-command[check\_load]=/usr/lib/nagios/plugins/check\_load -w 5,4,3 -c 10,6,4

3\. Reiniciar el Servicio NRPE

### Reinicia el servicio para aplicar los cambios:

- sudo systemctl restart nagios-nrpe-server

### Verifica que esté corriendo:

- sudo systemctl status nagios-nrpe-server

4\. Probar la Conexión Desde el Servidor Nagios

### Desde el contenedor de Nagios, prueba conectarte a NRPE en mv-bd:

### Instala el cliente NRPE en el contenedor (si no está instalado):

`	`-apt update

`	`-apt install nagios-plugins -y

### Ejecuta una prueba para verificar que NRPE responde:

`	`-/usr/lib/nagios/plugins/check\_nrpe -H <IP-DE-MV-BD>

5\. Verificar y definir el comando en commands.cfg

### Accede al contenedor de Nagios:

- docker exec -it nagios bash

### Edita el archivo de comandos de Nagios:

- nano /opt/nagios/etc/objects/commands.cfg

### contenido que debe estar o agregar si no esta:

\# Comando para verificar carga de la CPU

define command {

command\_name    check\_load

command\_line    $USER1$/check\_load -w 5,4,3 -c 10,6,4

}

\# Comando para verificar el espacio en disco

define command {

command\_name    check\_disk

command\_line    $USER1$/check\_disk -w 20% -c 10% -p /

}

\# Comando para verificar la conexión a PostgreSQL

define command {

command\_name    check\_postgresql

command\_line    $USER1$/check\_tcp -H $HOSTADDRESS$ -p 5432

}

define command {

command\_name    check\_https

command\_line    $USER1$/check\_http -H $HOSTADDRESS$ -S -p 443

}

### Validar la Configuración de Nagios

### Validar la configuración:

`	`-nagios -v /opt/nagios/etc/nagios.cfg

### Si no hay errores, reinicia Nagios para que los cambios tomen efecto:

`	`-pkill nagios

`	`-nagios /opt/nagios/etc/nagios.cfg

## Verificar en la Interfaz Web de Nagios

`	`Accede a la interfaz web de Nagios (http://<IP-de-Nagios>:8080) y verifica si los 	servicios ahora aparecen para mv-bd:

### configurar los servicios de monitoreo para nginx

### Ve al directorio de configuración de Nagios:

- cd /opt/nagios/etc/objects/

### Crea un archivo de configuración para el host remoto, por ejemplo: nginx\_host.cfg.

- sudo nano nginx\_host.cfg

### Agrega la definición del host:

- define host {

use                     linux-server

host\_name               nginx-server

alias                   Servidor NGINX

address                 <IP\_DEL\_HOST\_REMOTO>

max\_check\_attempts      3

check\_period            24x7

notification\_interval   30

notification\_period     24x7

}

define service {

use                     generic-service

host\_name               nginx-server

service\_description     HTTP Service

check\_command           check\_http

}

define service {

use                     generic-service

host\_name               nginx-server

service\_description     HTTPS Service

check\_command           check\_https

}

define service {

use                     generic-service

host\_name               nginx-server

service\_description     NGINX Status

check\_command           check\_nrpe!check\_nginx

}

### Verificar y habilitar la configuración

### Incluye el archivo nginx\_host.cfg en la configuración principal de Nagios, si no está incluido:

- sudo nano /opt/nagios/etc/nagios.cfg

### Agrega la línea:

- cfg\_file=/opt/nagios/etc/objects/nginx\_host.cfg

### Verifica que la configuración sea válida:

- sudo nagios -v /opt/nagios/etc/nagios.cfg

### Si no hay errores, reinicia el servicio Nagios para aplicar los cambios:

- sudo systemctl restart nagios

