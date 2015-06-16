FROM php:5.6-apache

RUN a2enmod rewrite

# install the PHP extensions we need
RUN apt-get update && apt-get install -y unzip less nmap vim libpng12-dev libjpeg-dev && rm -rf /var/lib/apt/lists/* \
  && docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
  && docker-php-ext-install gd
RUN docker-php-ext-install mysqli

VOLUME /var/www/html

ENV WORDPRESS_VERSION 4.2.2
ENV WORDPRESS_UPSTREAM_VERSION 4.2.2
ENV WORDPRESS_SHA1 d3a70d0f116e6afea5b850f793a81a97d2115039
ENV WORDPRESS_W3_TOTAL_CACHE_VERSION 0.9.4.1

# upstream tarballs include ./wordpress/ so this gives us /usr/src/wordpress
RUN curl -o wordpress.tar.gz -SL https://wordpress.org/wordpress-${WORDPRESS_UPSTREAM_VERSION}.tar.gz \
  && echo "$WORDPRESS_SHA1 *wordpress.tar.gz" | sha1sum -c - \
  && tar -xzf wordpress.tar.gz -C /usr/src/ \
  && rm wordpress.tar.gz

RUN curl -o w3_total_cache.zip -SL https://downloads.wordpress.org/plugin/w3-total-cache.${WORDPRESS_W3_TOTAL_CACHE_VERSION}.zip \
  && unzip w3_total_cache.zip -d /usr/src/wordpress/wp-content/plugins/ \
  && rm w3_total_cache.zip

RUN curl -o insert_php.zip -SL https://downloads.wordpress.org/plugin/insert-php.zip \
  && unzip insert_php.zip -d /usr/src/wordpress/wp-content/plugins/ \
  && rm insert_php.zip

# Add base w3 cache config
RUN mkdir /usr/src/wordpress/wp-content/w3tc-config
COPY config/w3-total-cache-master.php /usr/src/wordpress/wp-content/w3tc-config/master.php

RUN curl -o session_mgr.zip -SL https://downloads.wordpress.org/plugin/wp-session-manager.latest-stable.zip \
  && unzip session_mgr.zip -d /usr/src/wordpress/wp-content/plugins/ \
  && rm session_mgr.zip

# Turn off canonical redirects for the sake of random container ports
# http://marvintam.com/2009/05/wordpress-on-localhost-port-number/
RUN curl -o disable_redirects.zip -SL https://downloads.wordpress.org/plugin/disable-canonical-url-redirects.zip \
  && unzip disable_redirects.zip -d /usr/src/wordpress/wp-content/plugins/ \
  && mv /usr/src/wordpress/wp-content/plugins/disable-canonical-url-redirects/disable_canonical_url_redirects/*.php /usr/src/wordpress/wp-content/plugins/disable-canonical-url-redirects/. \
  && rm -Rf /usr/src/wordpress/wp-content/plugins/disable-canonical-url-redirects/disable_canonical_url_redirects \
  && rm disable_redirects.zip

RUN chown -R www-data:www-data /usr/src/wordpress

COPY docker-entrypoint.sh /entrypoint.sh

# grr, ENTRYPOINT resets CMD now
ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]
