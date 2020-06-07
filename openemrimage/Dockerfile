FROM  idaas/openemrbase
LABEL Maintainer="Kiran Kumar Mallru" \
      Description="OpenEMR is the most popular open source electronic health records and medical practice management solution."


# Configure nginx
COPY nginx.conf /etc/nginx/nginx.conf
# Remove default server definition
# RUN rm /etc/nginx/conf.d/default.conf
USER root

# Configure PHP-FPM

RUN  rm  /etc/php7/php-fpm.d/www.conf
RUN  rm  /usr/local/etc/php-fpm.d/www.conf
RUN  rm  /usr/local/etc/php-fpm.d/docker.conf
RUN  rm  /usr/local/etc/php-fpm.d/www.conf.default
RUN  rm  /usr/local/etc/php-fpm.d/zz-docker.conf
COPY fpm-pool.conf /etc/php7/php-fpm.d/www.conf

COPY fpm-pool.conf /usr/local/etc/php-fpm.d/www.conf
# COPY config/fpm-pool.conf /usr/local/etc/php-fpm.d/www.conf
COPY php.ini /etc/php7/conf.d/custom.ini
# use default value variables_order = "GPCS" 
# RUN sed -i 's/variables_order/;&/' /etc/php7/php.ini
# Configure supervisord
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Setup document root
RUN mkdir -p /var/www/html
    
# RUN curl -LJ https://github.com/openemr/openemr/archive/v5_0_2_3.tar.gz  -o  openemr.tar.gz && \
    # tar -zxf openemr.tar.gz --strip-components 1

COPY openemr-5.0.2.tar.gz  .
RUN tar -zxf openemr-5.0.2.tar.gz --strip-components 1

# Cleanup 
RUN rm openemr-5.0.2.tar.gz
# RUN rm /var/www/html/install/install_defaults.php 
RUN rm /var/www/html/sites/default/sqlconf.php
COPY sqlconf.php /var/www/html/sites/default/sqlconf.php

RUN composer install --no-cache

# COPY install_defaults.php /var/www/html/install/
# COPY config_override.php /var/www/html
# Make sure files/folders needed by the processes are accessable when they run under the nobody user

RUN touch /var/log/supervisor/supervisor.log /var/log/fpm-php.www.log /var/log/fpm.log /var/log/fpm-error.log /var/log/nginx/nginx.log /var/log/nginx/nginx-error.log

RUN chown -R nobody:nogroup /var/www/html 
RUN chown -R nobody:nogroup /var/log
# Make the document root a volume
# VOLUME /var/www/html

COPY dummy-cert /etc/nginx/dummy-cert
COPY dummy-key /etc/nginx/dummy-key

# Switch to use a non-root user from here on
USER nobody

# Add application
WORKDIR /var/www/html

# Expose the port nginx is reachable on
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

# Configure a healthcheck to validate that everything is up&running
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping
