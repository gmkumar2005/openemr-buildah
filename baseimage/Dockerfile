FROM  php:7.4-fpm
LABEL Maintainer="Kiran Kumar Mallru" \
      Description="OpenEMR is the most popular open source electronic health records and medical practice management solution."


RUN apt-get update
RUN apt-get install -y mariadb-client \
                       imagemagick \
                       supervisor \
                       nginx
                    
ADD https://raw.githubusercontent.com/mlocati/docker-php-extension-installer/master/install-php-extensions /usr/local/bin/

RUN chmod uga+x /usr/local/bin/install-php-extensions && sync && \
    install-php-extensions pdo_mysql \
                           ldap \
                           xsl \
                           gd \
                           zip \
                           soap \
                           gettext \
                           mysqli \
                           sockets \
                           tokenizer \
                           xmlreader \
                           calendar 
                            
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
# Configure nginx
COPY nginx.conf /etc/nginx/nginx.conf
# Remove default server definition
# RUN rm /etc/nginx/conf.d/default.conf

# Configure PHP-FPM
COPY fpm-pool.conf /etc/php7/php-fpm.d/www.conf
# COPY config/fpm-pool.conf /usr/local/etc/php-fpm.d/www.conf
COPY php.ini /etc/php7/conf.d/custom.ini
# use default value variables_order = "GPCS" 
# RUN sed -i 's/variables_order/;&/' /etc/php7/php.ini
# Configure supervisord
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
# Setup document root
RUN mkdir -p /var/www/html
RUN mkdir -p /var/www/html/public

COPY index.php/ /var/www/html/public/


# Make sure files/folders needed by the processes are accessable when they run under the nobody user
RUN chown -R nobody:nogroup /var/www/html && \
  chown -R nobody:nogroup /run && \
  chown -R nobody:nogroup /var/lib/nginx && \
  chown -R nobody:nogroup /var/log/nginx

# Make the document root a volume
# VOLUME /var/www/html

# Switch to use a non-root user from here on
USER nobody

# Add application
WORKDIR /var/www/html
# COPY src/ /var/www/html/

# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping
