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
cd ldap && docker build -t giafar/openldap . && cd ..
cd idp && docker build -t giafar/shibboleth-idp . && cd ..
cd sp && docker build -t giafar/shibboleth-sp . && cd ..
```
services are related each other so is important to start them in the right order with the correct name
```
docker run -d --rm --name spid-ldap giafar/openldap
docker run -d --rm --name spid-idp --link=spid-ldap:ldap.example.org giafar/shibboleth-idp
docker run -d --rm --name spid-sp --link=spid-idp:idp.example.org giafar/shibboleth-sp
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

## Playing with the images

Once images are started open your browser and navigate to [https://sp.example.org](https://sp.example.org):
1. Check shibboleth native access control with default entityID;
1. Check apache control by cookie test. Redirect to SPID provider selector;
1. Logout;
1. Session information.

## Check shibboleth native access control with default entityID
The /secure location is protected by shibboleth native configuration and if no session is present then you will be redirected to the identity provider for authentication. Use test as username and *password.1* as password to login and return to sp /secure page

## Check apache control by cookie test. Redirect to SPID provider selector
The /secure-by/apache location is protected by apache mod rewrite and checks that the _shib.* cookie is present in the request. If there is no cookie than you will be redirected to the /login.html page which contains standard SPID authentication buttons. To test the IDP use the **Aruba ID** button

## Logout
Once logged in click on the link to logout using the *Local* logout shibboleth native method

## Session Information
Once logged in click on the link to see session information and assertions sent by the idp

