#Recibimos Argumentos
ARG php_version

FROM php:${php_version}
LABEL  maintainer="ricurdinola@gmail.com"

#Actualizamos paquete e instalamos algunas librerías por defecto.
#Modificar según requerimiento del proyecto
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
		zip unzip libzip-dev \
		libmcrypt-dev \
        libaio1 \
        libicu-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install zip

RUN docker-php-ext-configure intl
RUN docker-php-ext-install intl
RUN docker-php-ext-enable intl

#Copiamos de la imagen composer a la maquina local.
COPY --from=composer /usr/bin/composer /usr/bin/composer

#Instalamos Node
ENV NODE_VERSION=14.16.0
RUN apt install -y curl
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
ENV NVM_DIR=/root/.nvm
RUN . "$NVM_DIR/nvm.sh" && nvm install ${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm use v${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm alias default v${NODE_VERSION}
ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin/:${PATH}"

RUN mkdir "/var/config" && mkdir "/usr/lib/oracle" && mkdir "/usr/lib/oracle/21.4" && mkdir "/usr/lib/oracle/21.4/client64"

#Copiamos los archivos de configuración del Apache y PHP
COPY ./config/apache2/apache2.conf /etc/apache2/
COPY ./config/apache2/sites-available/000-default.conf /etc/apache2/sites-available/
COPY ./config/php/ /usr/local/etc/php

#Copiamos el InstantClient
COPY ./instantclient-sqlplus-linux.x64-21.4.zip /usr/lib/oracle/21.4/client64
COPY ./instantclient-sdk-linux.x64-21.4.zip /usr/lib/oracle/21.4/client64
COPY ./instantclient-basic-linux.x64-21.4.zip /usr/lib/oracle/21.4/client64

RUN unzip /usr/lib/oracle/21.4/client64/instantclient-sqlplus-linux.x64-21.4.zip -d /usr/lib/oracle/21.4/client64/lib \
    && unzip /usr/lib/oracle/21.4/client64/instantclient-sdk-linux.x64-21.4.zip -d /usr/lib/oracle/21.4/client64/lib \
    && unzip /usr/lib/oracle/21.4/client64/instantclient-basic-linux.x64-21.4.zip -d /usr/lib/oracle/21.4/client64/lib \
    && echo /usr/lib/oracle/21.4/client64/lib/instantclient_21_4 > /etc/ld.so.conf.d/oracle.conf \
    && ldconfig && pecl channel-update pecl.php.net

RUN echo 'instantclient,/usr/lib/oracle/21.4/client64/lib/instantclient_21_4' | pecl install oci8

RUN a2enmod rewrite

#Seteamos en el path, el directorio de composer
RUN echo 'export PATH="$PATH:$HOME/.composer/vendor/bin"' >> ~/.bashrc

#Seteamos la zona horaria
#RUN cp /usr/share/zoneinfo/America/Argentina/Buenos_Aires /etc/localtime

#Exponemos el puerto 80 y 443
EXPOSE 80 443