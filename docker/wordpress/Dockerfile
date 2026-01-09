FROM php:8.1-apache

# Install required PHP extensions
RUN docker-php-ext-install mysqli

# Enable Apache rewrite module (needed by WordPress)
RUN a2enmod rewrite

# Download and extract WordPress
WORKDIR /var/www/html
RUN curl -o wordpress.tar.gz https://wordpress.org/latest.tar.gz \
    && tar -xzf wordpress.tar.gz --strip-components=1 \
    && rm wordpress.tar.gz

# Set correct permissions
RUN chown -R www-data:www-data /var/www/html

EXPOSE 80

