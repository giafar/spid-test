FROM tomcat:8.5

LABEL mantainer="gianluca.fares@giafar.it"

ARG SCOPE=example.org
ARG SVCNAME=idp.example.org
ARG INSTALLDIR=/opt/shibboleth-idp
ARG TMPDIR=/tmp/idp

RUN apt-get update && apt-get install -y --no-install-recommends vim curl && \
    rm -rf /var/lib/apt/lists/*
    
#COPY shibboleth-identity-provider-3.3.2.tar.gz ${TMPDIR}/
ADD https://shibboleth.net/downloads/identity-provider/3.3.2/shibboleth-identity-provider-3.3.2.tar.gz ${TMPDIR}/
COPY ./tomcat/ /usr/local/tomcat/

RUN cd ${TMPDIR} && tar xzf shibboleth-identity-provider-3.3.2.tar.gz

COPY ./shibboleth/ ${TMPDIR}/shibboleth-identity-provider-3.3.2/

# Thanks to https://www.switch.ch/aai/guides/idp/installation/#shibbolethidp
RUN cd ${TMPDIR}/shibboleth-identity-provider-3.3.2/ && \
    echo "idp.sealer.password = $(openssl rand -base64 12)" > credentials.properties && \
    echo "\n\
idp.additionalProperties= /conf/ldap.properties, /conf/saml-nameid.properties, /conf/services.properties, /conf/credentials.properties \n\
idp.sealer.storePassword= %{idp.sealer.password}\n\
idp.sealer.keyPassword= %{idp.sealer.password}\n\
idp.signing.key= %{idp.home}/credentials/idp-signing.key\n\
idp.signing.cert= %{idp.home}/credentials/idp-signing.crt\n\
idp.encryption.key= %{idp.home}/credentials/idp-encryption.key\n\
idp.encryption.cert= %{idp.home}/credentials/idp-encryption.crt\n\
idp.entityID= https://${SVCNAME}/idp/shibboleth\n\
idp.scope=${SCOPE}\n\
idp.authn.flows=MFA\n\
idp.ui.fallbackLanguages= en,de,fr\n\
idp.entityID.metadataFile=${INSTALLDIR}/metadata/idp-metadata.xml\n\
idp.loglevel.ldap=INFO\n\
idp.loglevel.root=INFO\n\
" >> temp.properties && \
    bin/install.sh \
        -Didp.src.dir=. \
        -Didp.target.dir=${INSTALLDIR} \
        -Didp.merge.properties=temp.properties \
        -Didp.sealer.password=$(cut -d ' ' -f3 <credentials.properties) \
        -Didp.keystore.password=$(cut -d ' ' -f3 <credentials.properties)\
        -Didp.conf.filemode=644 \
        -Didp.host.name=${SVCNAME} \
        -Didp.uri.subject.alt.name=${SVCNAME} \
        -Didp.scope=${SCOPE}   && \
    mv credentials.properties ${INSTALLDIR}/conf && \
    cp ${TMPDIR}/shibboleth-identity-provider-3.3.2/metadata/* ${INSTALLDIR}/metadata/ && \
    cp ${TMPDIR}/shibboleth-identity-provider-3.3.2/credentials/* ${INSTALLDIR}/credentials/ && \
    chmod 600 $INSTALLDIR/credentials/*.key

COPY ./shibboleth/metadata ${TMPDIR}/shibboleth-identity-provider-3.3.2/metadata

RUN cd ${INSTALLDIR}/edit-webapp/WEB-INF/lib && \
    curl -O https://repo1.maven.org/maven2/jstl/jstl/1.2/jstl-1.2.jar && \
    cd ${TMPDIR}/shibboleth-identity-provider-3.3.2/ && \
    bin/build.sh -Didp.target.dir=${INSTALLDIR} && \
    rm -fR ${TMPDIR}/shibboleth-identity-provider-3.3.2 

