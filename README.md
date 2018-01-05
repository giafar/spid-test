# Shibboleth IDP and SP for SPID integration

This project contains three docker images:
+ [ldap](ldap/README.md): an openladap instance used as idp authentication backend
+ [idp](idp/README.md): is the shibboleth identity provider
+ [sp](sp/README.md): is the shibboleth servcice provider

The project aims to create a testing environment for the Italian public identity manager, [SPID](http://spid-regole-tecniche.readthedocs.io/en/latest/introduzione.html), which is based on SAML 2.0. The spec defines how Service Provider and Identity Provider works and how they must be implemented.

**I'm so sorry but the project still lack of documentation**

## What is and what is not still implemented

*TODO*
## Docker
### Building the images
To build the images:
```
cd ldap && docker build -t giafar/spid-ldap . && cd ..
cd idp && docker build -t giafar/spi-idp . && cd ..
cd sp && docker build -t giafar/spid-sp . && cd ..
```
### From docker hub
To download the images from docker hub
```
docker pull giafar/spid-ldap
docker pull giafar/spid-idp
docker pull giafar/spid-sp
```
### Running the images
services are related each othe, it is important to start them in the right order with the correct name
```
docker run -d --rm --name spid-ldap giafar/spid-ldap
docker run -d --rm --name spid-idp --link=spid-ldap:ldap.example.org giafar/spid-idp
docker run -d --rm --name spid-sp --link=spid-idp:idp.example.org giafar/spid-sp
```
As you can notice no port is exposed and images are linked by fqdn:
1. ldap.example.org;
1. idp.example.org;
1. sp.example.org.

Modify the host file to include required fqnd or run the following script


```
echo "$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' spid-sp) sp.example.org" | sudo tee --append /etc/hosts > /dev/null
echo "$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' spid-idp) idp.example.org" | sudo tee --append /etc/hosts > /dev/null
echo "$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' spid-ldap) ldap.example.org" | sudo tee --append /etc/hosts > /dev/null

```
To stop everything
```
docker container stop spid-sp
docker container stop spid-idp
docker container stop spid-ldap
```

## Playing with the images

Once images are started open your browser and navigate to [https://sp.example.org](https://sp.example.org):
1. Check shibboleth native access control with default entityID;
1. Check apache control by cookie test. Redirect to SPID provider selector;
1. Check shibboleth native access control with default entityID and forced SpidL2 authnContextClassRef by request mapper;
1. Check shibboleth native access control with default entityID, forced SpidL1 authnContextClassRef by request mapper and SPID Attributes Set 2;
1. Check shibboleth native access control with default entityID, forced SpidL1 authnContextClassRef by request mapper and SPID Attributes Set 3;
1. Check shibboleth native access control with default entityID, forced SpidL1 authnContextClassRef by request mapper and SPID Attributes Set 4;
1. Check apache control by cookie test. Redirect to SPID provider selector;
1. Logout;
1. Session information;
1. Metadata.

### Check shibboleth native access control with default entityID
The */secure* location is protected by shibboleth native configuration and if no session is present then you will be redirected to the identity provider for authentication. Use test as username and *password.1* as password to login and return to sp /secure page

### Check shibboleth native access control with default entityID, forced SpidL1 authnContextClassRef by request mapper and SPID Attributes Set 2
The */secure-spidl1-attr-2* location is protected by shibboleth native mapper using an *applicationId* that maps an *ApplicationOverride* that forces the AttributeConsumerServiceIndex to index *1* ( Set Attributi 2 - ID Cittadini completo): see [shibboleth2.xml](sp-conf/shibboleth2.xml) and [sp image docs](sp/README.md).
*Check shibboleth native access control with default entityID, forced SpidL1 authnContextClassRef by request mapper and SPID Attributes Set 3* and *Check shibboleth native access control with default entityID, forced SpidL1 authnContextClassRef by request mapper and SPID Attributes Set 4* except that are used *Set Attributi 3 - ID Cittadini e Persone giuridiche* and *Set Attributi 4 - ID Persone giuridiche* indexes.

### Check apache control by cookie test. Redirect to SPID provider selector
The */secure-by/apache* location is protected by apache mod rewrite and checks that the _shib.* cookie is present in the request. If there is no cookie than you will be redirected to the /login.html page which contains standard SPID authentication buttons. To test the IDP use the **Aruba ID** button
### Check apache control by cookie test. Redirect to SPID provider selector
The */secure-by-apache* locations is protected by apache using an url rewrite rule
```
RewriteEngine On
RewriteCond %{REQUEST_URI} ^/secure-by-apache/.*
RewriteCond %{HTTP_COOKIE} !^_shib.*$ [NC]
RewriteRule .* /login.html?target=%{REQUEST_URI} [NC,R,L,QSA]
```
The */login.html* page has the standard SPID button, thanks to [.it Developers Italia](https://github.com/italia/spid-sp-access-button), and a combo to select the *authnContextClassRef*. Please select the *Aruba* identity provider which is mapped to idp.example.org

### Logout
Once logged in click on the link to logout using the *Local* logout shibboleth native method

### Session Information
Once logged in click on the link to see session information and assertions sent by the idp

### Metadata
View SP metadata