FROM php:8.2-apache

# Install dependencies and MongoDB extension
RUN apt-get update && apt-get install -y \
    libssl-dev \
    unzip \
    git \
 && pecl install mongodb \
 && docker-php-ext-enable mongodb \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Enable Apache mod_rewrite for routing
RUN a2enmod rewrite

# Setup document root to the public directory
ENV APACHE_DOCUMENT_ROOT /var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Copy application files
WORKDIR /var/www/html
COPY . .

# Allow Composer to run as root
ENV COMPOSER_ALLOW_SUPERUSER=1

# Run composer install ignoring strict platform requirement checks
RUN composer install --no-dev --no-interaction --optimize-autoloader --ignore-platform-reqs

# Copy startup script that applies the $PORT env variable for Render
COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

# Expose default port
EXPOSE 80

# Use startup script so Apache listens on Render's dynamic $PORT
CMD ["/usr/local/bin/docker-entrypoint.sh"]
