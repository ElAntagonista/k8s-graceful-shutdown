FROM php:8.2-fpm

RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

# Install PHP and composer dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    libzip-dev \
    libonig-dev \
    zip \
    jpegoptim optipng pngquant gifsicle \
    curl

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install extensions
RUN docker-php-ext-install pdo_mysql mbstring zip exif pcntl
RUN docker-php-ext-configure gd
RUN docker-php-ext-install gd

RUN sed -i 's/;process_control_timeout = 0/process_control_timeout = 20s/' /usr/local/etc/php-fpm.conf


# Copy existing application directory to a temporary directory in the Docker image
# Copy existing application directory
COPY . /var/www

# Change current user to www
RUN chown -R www-data:www-data /var/www

# Change current user to www
USER www-data

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000