# Docker image for Shibboleth Service Provider

This image is based on [httpd:2.4](https://hub.docker.com/_/httpd/), the version of Shibboleth SP available with debian 8.10 and is customized according the [AGID - SPID - Regole tecniche Service Provider](http://spid-regole-tecniche.readthedocs.io/en/latest/regole-tecniche-sp.html).

## Docker
### Building the image
To build the image:
```
docker build -t giafar/spid-sp .
```
if you change the tag *giafar/shibboleth-sp* please remember to modify the docker-compose.yml file as well.
### From docker hub
To download the image from docker hub
```
docker pull giafar/spid-sp
```
### Running the image
To run the image in interactive mode and expose the LDAP protocolo:
```
docker run -it --name spid-sp -p 443:443 giafar/spid-sp
```
or in detach mode
```
docker run -d --name spid-sp -p 443:443 giafar/spid-sp
```
To look at the logs in detached mode
```
docker container log --follow spid-sp
```
To get the ip adddress of a running container
```
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' spid-sp
```
### Running the image with IDP and LDAP backend
The image uses the *giafar/shibboleth-idp* for SSO and expects the service is located at idp.example.org
```
docker run -d --rm --name spid-ldap giafar/spid-ldap
docker run -d --rm --name spid-idp --link=spid-ldap:ldap.example.org giafar/spid-idp
docker run -d --rm --name spid-sp --link=spid-idp:idp.example.org giafar/spid-sp
```
Shibboleth logs **are not** redirected to the standard docker log so if you wanna follow this kind of log and apache logs
```
docker container exec -i spid-sp sh -c 'tail -f logs/* /var/log/shibboleth/*'
```
To stop and cleanup everything
```
docker container stop spid-sp
docker container stop spid-idp
docker container stop spid-ldap
```

## Shibboleth SP customization

*TODO*

### Deploy SP metadata
If you modify SP metadata configuration remember to update the sp-example-org-metadata.xml which is inside the idp/shibboleth/metadata folder.
On a running giafar/spid-sp container run the following command and rebuild the spid-idp image
```
curl -k -o <idp home folder>/shibboleth/metadata/sp-example-org-metadata.xml https://sp.example.org/saml/Metadata
```
Remember to edit the file host including the correct IP Address for sp.example.org
## Apache customization

*TODO*
