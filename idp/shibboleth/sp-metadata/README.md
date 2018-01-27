# SP-METADATA

If the entityID is https://sp.example.org then the file name must be ac5e671e900b7b7db957b97e8f8bc85751847472.xml. See [Shibboleth IDP referenc](https://wiki.shibboleth.net/confluence/display/IDP30/LocalDynamicMetadataProvider)
```
echo -n "https://sp.example.org" | openssl sha1
```