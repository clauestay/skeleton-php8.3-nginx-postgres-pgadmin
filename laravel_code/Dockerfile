# FROM php:8.1-fpm
# RUN docker-php-ext-install mysqli pdo pdo_mysql
# RUN docker-php-ext-enable mysqli

FROM php:8.3-fpm

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

RUN set -eux; \
    apt-get update; \
    apt-get upgrade -y; \
    apt-get install -y --no-install-recommends \
            # libaoi1 necesario para oci8
            libaio1 \
            git \
            zip \
            unzip \
            curl \
            # gnupg necesarios para instalar npm
            gnupg \
            libmemcached-dev \
            libz-dev \
            libpq-dev \
            libjpeg-dev \
            libpng-dev \
            libfreetype6-dev \
            libssl-dev \
            libwebp-dev \
            libxpm-dev \
            libmcrypt-dev \
            libonig-dev; \
    rm -rf /var/lib/apt/lists/*

RUN set -eux; \
    docker-php-ext-install pdo_pgsql; \
    # Install the PHP gd library
    docker-php-ext-configure gd \
            --prefix=/usr \
            --with-jpeg \
            --with-webp \
            --with-xpm \
            --with-freetype; \
    docker-php-ext-install gd; \
    php -r 'var_dump(gd_info());'

# INSTALACION ORACLE INSTANTCLIENT Y OCI8    

# crear carpeta
RUN mkdir -p /opt/oracle

# copiar archivos oracle
COPY ./oracle /opt/oracle

# entrar a carpeta y crear enlace
RUN cd /opt/oracle/instantclient_11_2 && \
ln -s libclntsh.so.11.1 libclntsh.so

# instalar oci8
RUN echo "instantclient,/opt/oracle/instantclient_11_2" | pecl install oci8

# copiar php.ini de produccion para usar
RUN cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini 

# agregar linea al archivo php.ini
RUN echo "extension=oci8" >> /usr/local/etc/php/php.ini

# agregar a ld.so.conf.d y ejecutar ldconfig para actualizar registro
RUN echo /opt/oracle/instantclient_11_2 > /etc/ld.so.conf.d/oracle-instantclient.conf
RUN ldconfig

# instalar Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Instala Node.js y npm
# RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
# ARG NODE_MAJOR=20
# RUN echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list


ARG NODE_VERSION=20
RUN curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_VERSION.x nodistro main" > /etc/apt/sources.list.d/nodesource.list \
    && apt-get update \
    && apt-get install -y nodejs \
    && npm install -g npm