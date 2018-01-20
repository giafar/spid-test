FROM php:7.0-apache

ARG APACHE_ROOT=/etc/apache2
ARG WEB_ROOT=/var/www/html

LABEL mantainer="gianluca.fares@giafar.it"

RUN apt-get update && apt-get install -y --no-install-recommends vim shibboleth-sp2-utils shibboleth-sp2-common curl unzip libapache2-mod-shib2

RUN	rm -rf /var/lib/apt/lists/* && rm -fR /tmp/* 

RUN sed -i '/exec/i \
/etc/init.d/shibd start ' /usr/local/bin/docker-php-entrypoint

COPY ./sp-httpd/ ${APACHE_ROOT}/

COPY ./sp-site/ ${WEB_ROOT}/

COPY ./sp-conf/ /etc/shibboleth/

RUN cd /tmp && \
    curl -k -o master.zip https://codeload.github.com/italia/bootstrap-italia/zip/master && \
    unzip -o master.zip 'bootstrap-italia-master/dist/*' -d ${WEB_ROOT}/ && \
    rm -f /tmp/master-zip
RUN chown -R root:root /etc/shibboleth/*
    

