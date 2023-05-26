> # UDG - CUCEI 
> #### 29 de Mayo de 2023
### <p align="center"> Anthony Esteven Sandoval Marquez</p>
### <p align="center"> Diego Ivan Becerra Gonzalez</p>
##
#### <p align="center"> Materia: Computacion Tolerante a Fallas </p>
#### <p align="center"> Profesor: Michel Emanuel López Franco </p>
#### <p align="center"> Ciclo: 2023-A </p>

> ## Proyecto Final
#### Se realizo una implementación en kubernetes para una aplicación realizada en Laravel que sirve como un sitio para hacer blogs. También cuenta con una base de datos de mysql que permite el registro de usuarios y la realización de blogs.
#### Para lograrlo se hicieron en un principio dos imágenes de Docker, una de php con todas las dependencias para que funcione Laravel y otra imagen de mysql para que contenga a la base de datos de la aplicación. Ya con las imágenes subidas a Docker hub solamente fueron utilizadas en los archivos yaml para hacer la implementación de kubernetes.


### <p align="center"> Docker</p>
#### Dokcerfile para la aplicacion de Laravel (PHP)
```dockerfile
FROM php:8.1.10-fpm

# Instala las dependencias necesarias
RUN apt-get update && apt-get install -y \
    libzip-dev \
    zip \
    unzip

# Instala las extensiones de PHP requeridas para Laravel
RUN docker-php-ext-install pdo_mysql zip

# Copia los archivos de la aplicación Laravel al directorio de trabajo
COPY . /var/www/html

WORKDIR /var/www/html

# Instala las dependencias de Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Instala las dependencias del proyecto
RUN composer install --no-interaction

# Genera una clave de aplicación de Laravel
RUN php artisan key:generate

# Establece los permisos adecuados en el directorio de almacenamiento de Laravel
RUN chown -R www-data:www-data /var/www/html/storage

# Ejecuta los comandos adicionales
RUN composer install
RUN php artisan key:generate
RUN rm -rf public/storage
RUN php artisan storage:link

EXPOSE 80

CMD ["php", "-S", "0.0.0.0:80", "-t", "public"]
```

#### Dokcerfile para la base de datos de la aplicacion
```dockerfile
FROM mysql:8.0

# Establece las variables de entorno para la configuración de la base de datos
ENV MYSQL_DATABASE=blog
ENV MYSQL_USER=blogger
ENV MYSQL_PASSWORD=password
ENV MYSQL_ROOT_PASSWORD=password

# Copia los archivos SQL de inicialización al directorio de Docker
COPY ./blog-db.sql /docker-entrypoint-initdb.d

EXPOSE 3306
```

### <p align="center"> Kubernetes</p>

#### Deployment para la aplicacion
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: php-app
  template:
    metadata:
      labels:
        app: php-app
    spec:
      containers:
      - name: php-app
        image: zaikron/php-laravel:latest
        ports:
          - containerPort: 80
        env:
          - name: MYSQL_ROOT_PASSWORD
            value: password
          - name: MYSQL_DATABASE
            value: blog
          - name: MYSQL_USER
            value: blogger
          - name: MYSQL_PASSWORD
            value: password
```

#### Servicio para la aplicacion
```yaml
apiVersion: v1
kind: Service
metadata:
  name: php-service
spec:
  selector:
    app: php-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

#### Deployment para la base de datos
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: zaikron/blog-db:latest
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: password
            - name: MYSQL_DATABASE
              value: blog
            - name: MYSQL_USER
              value: blogger
            - name: MYSQL_PASSWORD
              value: password
          ports:
            - containerPort: 3306
```

#### Servicio para la base de datos
```yaml
apiVersion: v1
kind: Service
metadata:
  name: db
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```
