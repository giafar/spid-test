FROM httpd:2.4

LABEL mantainer="gianluca.fares@giafar.it"

RUN apt-get update && apt-get install -y --no-install-recommends vim shibboleth-sp2-utils shibboleth-sp2-common curl

RUN cd /tmp && apt-get download libapache2-mod-shib2 && \
    mkdir shib && find . -name "libapache*.deb" -type f -exec dpkg -x {} /tmp/shib \; && \
    find . -name "*shib2.so" -exec cp {} /usr/local/apache2/modules \; && \
    chmod +x /usr/local/apache2/modules/mod_shib2.so

RUN	rm -rf /var/lib/apt/lists/* && rm -fR /tmp/* && mkdir /var/log/shibboleth

RUN sed -i '/httpd/i \
/etc/init.d/shibd start ' /usr/local/bin/httpd-foreground

RUN echo "Include conf/extra/httpd-shib2.conf" >> /usr/local/apache2/conf/httpd.conf

COPY ./sp-httpd/ /usr/local/apache2/

COPY ./sp-site/ /usr/local/apache2/htdocs/

COPY ./sp-conf/ /etc/shibboleth/

RUN chown -R root:root /etc/shibboleth/*
    

