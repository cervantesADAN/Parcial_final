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
-    Crear el archivo de configuración para el servicio (ejemplo para PostgreSQL):

bash
Copiar código
nano mv-bd.cfg
Contenido:

plaintext
Copiar código
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
Actualizar el archivo principal nagios.cfg para incluir las nuevas configuraciones:

bash
Copiar código
nano /opt/nagios/etc/nagios.cfg
Agregar:

plaintext
Copiar código
cfg_file=/opt/nagios/etc/objects/mv-bd.cfg
Simulación de Alertas
Detener un contenedor para generar una alerta:


bash
Copiar código
docker stop postgres_container
Verificar la alerta generada en la interfaz web de Nagios.

Capturas de Pantalla
Agrega capturas en la carpeta screenshots/ y enlázalas en esta sección:
