# https://hub.docker.com/_/drupal/
FROM drupal:8-fpm

ENV DEBIAN_FRONTEND noninteractive

WORKDIR /var/www/html

# Install supporting packages.
RUN apt-get update \
  && apt-get install -y git default-mysql-client unzip \
  && apt-get clean \
  && apt-get autoclean \
  && apt-get -y autoremove \
  && rm -rf /var/lib/apt/lists/*

# Install Composer, Drush and Drupal Console.
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
  && HOME=/ /usr/local/bin/composer global require drush/drush:~8 \
  && ln -s /.composer/vendor/drush/drush/drush /usr/local/bin/drush \
  && curl https://drupalconsole.com/installer -L -o /usr/local/bin/drupal \
  && chmod +x /usr/local/bin/drupal \
  && php --version; composer --version; drupal --version; drush --version
