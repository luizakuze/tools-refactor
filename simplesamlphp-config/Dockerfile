FROM php:8.1-apache

RUN docker-php-ext-install mysqli && docker-php-ext-enable mysqli
RUN apt-get update && apt-get install -y \
    openssl \
    ssl-cert \
    && a2enmod ssl \
    && apt-get clean

RUN apt-get update && apt-get install -y \
    sqlite3 \
    libsqlite3-dev

RUN apt-get update && apt-get install -y --no-install-recommends \
  curl \
  git \
  libmemcached-dev \
  libpng-dev \
  unzip \
  zlib1g-dev \
  libonig-dev \
  && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-install -j5 gd mbstring pdo pdo_mysql 
RUN docker-php-ext-install pdo pdo_sqlite
COPY . /var/www/html/
COPY ./default-ssl.conf /etc/apache2/sites-available/000-default.conf

RUN chown -R www-data:www-data /var/www/html && chmod -R 755 /var/www/html
RUN a2ensite default-ssl.conf
EXPOSE 80 443
CMD ["apache2-foreground"]

