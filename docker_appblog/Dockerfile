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

