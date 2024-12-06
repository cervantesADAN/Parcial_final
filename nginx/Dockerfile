# Imagen base con Nginx
FROM nginx:latest

# Copia los archivos de la platilla
#COPY /plantilla /usr/share/nginx/html

# Copia el archivo de configuración de Nginx personalizado
COPY nginx.conf /etc/nginx/conf.d/

# Inicia Nginx en primer plano
CMD ["nginx", "-g", "daemon off;"]