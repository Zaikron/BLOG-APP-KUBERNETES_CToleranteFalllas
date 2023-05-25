FROM mysql:8.0

# Establece las variables de entorno para la configuración de la base de datos
ENV MYSQL_DATABASE=blog
ENV MYSQL_USER=blogger
ENV MYSQL_PASSWORD=password
ENV MYSQL_ROOT_PASSWORD=password

# Copia los archivos SQL de inicialización al directorio de Docker
COPY ./blog-db.sql /docker-entrypoint-initdb.d

EXPOSE 3306
