FROM osixia/openldap:1.1.11

LABEL mantainer="gianluca.fares@giafar.it"

RUN apt-get update && apt-get install -y --no-install-recommends vim curl && \
    rm -rf /var/lib/apt/lists/*

ADD bootstrap/ /container/service/slapd/assets/config/bootstrap/
ADD certs/ /container/service/slapd/assets/certs/
ADD environment/ /container/environment/01-custom/