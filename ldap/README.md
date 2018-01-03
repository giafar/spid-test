# Openldap docker image for Shibboleth IDP
This image is based on [osixia/openldap](https://github.com/osixia/docker-openldap) docker image and is configured as Shibboleth IDP backend for password authentication.
The image has some custom ldiff and a schema file to pre-populate the directory with a new **objectclass** which has some custom **attributetype**.

The new objectclass, see [spid.schema](bootstrap/schema/spid.schema), is **spidObject** and is a *SUP inetOrgPerson* that **MUST** have the *attributetype* **spidCode** and **MAY** have some of **name**, **mobilePhone**, **digitalAddress**, **expirationDate**, **address**, **email**, **familyName**, **placeOfBirth**, **countyOfBirth**, **dateOfBirth**, **gender**, **companyName**, **registeredOffice**, **fiscalNumber**, **ivaCode**, **idCard**: fields are defined by [AGID - Agenzia per l'Italia Digitale](http://www.agid.gov.it/sites/default/files/regole_tecniche/tabella_attributi_idp_v1_0.pdf) but you can change according your needs.

The **spidCode** is an unique identifier assigned by the Identity Provider and cannot be changed by the user. A custom ocl is defined to deny any change to this field: see [00-ocl.ldiff](bootstrap/ldif/custom/00-ocl.ldif). The file enable everyone to read the directory except for the *userPassword* and *shadowLastChange* fields that are granted to the admin user only in write mode.

The directory is pre-populated with the OU Users and some users:
+ admin: cn=admin,dc=example,dc=org the directory admin;
+ shibboleth: uid=shibboleth,ou=Users,dc=example,dc=org which is the user Shibboleth IDP uses to connect to the directory;
+ test: uid=test,ou=Users,dc=example,dc=org used for authentication.

The password is **password.1** for all users.

## Docker
### Building the image
To build the image:
```
docker build -t giafar/openldap .
```
if you change the tag *giafar/openldap* please remember to modify the docker-compose.yml file as well.

### Running the image
To run the image in interactive mode and expose the LDAP protocolo:
```
docker run -it --name spid-ldap -p 389:389 giafar/openldap
```
or in detach mode
```
docker run -d --name spid-ldap -p 389:389 giafar/openldap
```
To look at the logs in detached mode
```
docker container log --follow spid-ldap
```
To get the ip adddress of a running container
```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' spid-ldap
```

## TLS
TSL is not configured so is available only LDAP and not LDAPS.
