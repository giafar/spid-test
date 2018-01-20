FROM httpd:2.4

ARG APACHE_ROOT=/usr/local/apache2
ARG WEB_ROOT=/usr/local/apache2/htdocs

LABEL mantainer="gianluca.fares@giafar.it"

RUN apt-get update && apt-get install -y --no-install-recommends vim shibboleth-sp2-utils shibboleth-sp2-common curl unzip

RUN cd /tmp && apt-get download libapache2-mod-shib2 && \
    mkdir shib && find . -name "libapache*.deb" -type f -exec dpkg -x {} /tmp/shib \; && \
    find . -name "*shib2.so" -exec cp {} ${APACHE_ROOT}/modules \; && \
    chmod +x ${APACHE_ROOT}/modules/mod_shib2.so

RUN	rm -rf /var/lib/apt/lists/* && rm -fR /tmp/* && mkdir /var/log/shibboleth

RUN sed -i '/httpd/i \
/etc/init.d/shibd start ' /usr/local/bin/httpd-foreground

RUN echo "Include conf/extra/httpd-shib2.conf" >> ${APACHE_ROOT}/conf/httpd.conf

COPY ./sp-httpd/ ${APACHE_ROOT}/

COPY ./sp-site/ ${WEB_ROOT}/

COPY ./sp-conf/ /etc/shibboleth/

RUN cd /tmp && \
    curl -k -o master.zip https://codeload.github.com/italia/bootstrap-italia/zip/master && \
    unzip -o master.zip 'bootstrap-italia-master/dist/*' -d ${WEB_ROOT}/ && \
    rm -f /tmp/master-zip
RUN chown -R root:root /etc/shibboleth/*
    

