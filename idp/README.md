# Docker image for Shibboleth Identity Provider

This image is based on [tomcat:8.5](https://hub.docker.com/_/tomcat/), [Shibboleth IDP 3.3.2](https://shibboleth.net/downloads/identity-provider/3.3.2/shibboleth-identity-provider-3.3.2.tar.gz) and is customized according the [AGID - SPID - Regole tecniche Identity Provider](http://spid-regole-tecniche.readthedocs.io/en/latest/regole-tecniche-idp.html).

## Docker
### Building the image
To build the image:
```
docker build -t giafar/shibboleth-idp .
```
if you change the tag *giafar/spid-idp* please remember to modify the docker-compose.yml file as well.
### From docker hub
To download the image from docker hub
```
docker pull giafar/spid-idp
```
### Running the image
To run the image in interactive mode and expose the LDAP protocolo:
```
docker run -it --name spid-idp -p 443:443 giafar/spid-idp
```
or in detach mode
```
docker run -d --name spid-idp -p 443:443 giafar/spid-idp
```
To look at the logs in detached mode
```
docker container log --follow spid-idp
```
To get the ip adddress of a running container
```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' spid-idp
```
### Running the image with the LDAP backend
The image uses the *giafar/openldap* backend for user authentication and expects the directory service is located at ldap.example.org
```
docker run -d --rm --name spid-ldap giafar/spid-ldap
docker run -d --rm --name spid-idp --link=spid-ldap:ldap.example.org giafar/spid-idp
```
Shibboleth logs **are not** redirected to the standard docker log so if you wanna follow this kind of log
```
docker container exec -i spid-idp sh -c 'tail -f /opt/shibboleth-idp/logs/*'
```
To stop and cleanup everything
```
docker container stop spid-idp
docker container stop spid-ldap
```

## Shibboleth IDP customization
The image claims to implement a basic **AGID Identity Provider** according this specs.
Some distributions files, [Shibboleth Configuration File Summary](https://wiki.shibboleth.net/confluence/display/IDP30/ConfigurationFileSummary), have been modified and discussed in this document:
1. **Access Control Configuration**: shibboleth/dist/conf/access-control.xml.dist;
1. **Attribute Filter Configuration**: shibboleth/dist/conf/attribute-filter.xml.dist;
1. **Attribute Resolver**: shibboleth/dist/conf/attribute-resolver.xml.dist;
1. **LDAP Properties**: shibboleth/dist/conf/ldap.properties.dist;
1. **Logging**: shibboleth/dist/conf/logback.xml.dist;
1. **Metadata Filter Plugin**: shibboleth/dist/conf/metadata-providers.xml.dist;
1. **Authentication Configuration**: shibboleth/dist/conf/authn/general-authn.xml.dist;
1. **IPAddress Authn**: shibboleth/dist/conf/authn/ipaddress-authn-config.xml.dist;
1. **MultiFActor Authn**: shibboleth/dist/conf/authn/mfa-authn-config.xml.dist;
1. **Password Authn**: shibboleth/dist/conf/authn/password-authn-config.xml;

### Access Control Configuration
Just to allow access from 0.0.0.0/0 network.

See [Shibboleth Access Control Configuration](https://wiki.shibboleth.net/confluence/display/IDP30/AccessControlConfiguration)

### Shibboleth Attribute Filter Configuration
*TODO*

See [Shibboleth Attribute Filter Configuration](https://wiki.shibboleth.net/confluence/display/IDP30/AttributeFilterConfiguration)

### Shibboleth Attribute Resolver
*TODO*

See [Shibboleth Attribute Resolver](https://wiki.shibboleth.net/confluence/display/IDP30/AttributeResolverConfiguration)

### LDAP Properties
*TODO*

### Shibboleth Logging
*TODO*

See [Shibboleth Logging](https://wiki.shibboleth.net/confluence/display/IDP30/LoggingConfiguration)
### Shibboleth Metadata Filter Plugin
*TODO*

See [Shibboleth Metadata Filter Plugin](https://wiki.shibboleth.net/confluence/display/IDP30/MetadataFilterPlugin)
### Shibboleth Authentication Configuration
*TODO*

See [Shibboleth Authentication Configuration](https://wiki.shibboleth.net/confluence/display/IDP30/AuthenticationConfiguration)
### Shibboleth IPAddress Authn
*TODO*

See [Shibboleth IPAddress Authn](https://wiki.shibboleth.net/confluence/display/IDP30/IPAddressAuthnConfiguration)

### Shibboleth MultiFActor Authn
*TODO*

See [Shibboleth MultiFActor Authn](https://wiki.shibboleth.net/confluence/display/IDP30/MultiFactorAuthnConfiguration)

### Shibboleth Password Authn
*TODO*

See [Shibboleth Password Authn](https://wiki.shibboleth.net/confluence/display/IDP30/PasswordAuthnConfiguration)

## Tomcat customization

The tomcat installation exposes the Shibboleth IDP solution via http and https. The tomcat/conf directory is copied inside the  /usr/local/tomcat docker image folder and deploy a new server.xml and a Catalina/localhost/idx.xml file which is responsable to deply del idp.war package

### Server.xml
The new entry is
```
<Connector port="443" protocol="org.apache.coyote.http11.Http11AprProtocol" maxThreads="150" SSLEnabled="true" >
    <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
    <SSLHostConfig>
        <Certificate certificateKeyFile="/opt/shibboleth-idp/credentials/idp.example.org.key" certificateFile="/opt/shibboleth-idp/credentials/idp.example.org.crt" type="RSA" />
    </SSLHostConfig>
    </Connector>
```    
a preconfigured X509 certificate and the related key are deployed during the docker image build.

### idp.xml
Just deploy a new war
```
<Context docBase="/opt/shibboleth-idp/war/idp.war" unpackWAR="true" swallowOutput="true">
  <Manager pathname="" />
</Context>
``` 
