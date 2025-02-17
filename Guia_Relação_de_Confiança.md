# Guia de Configuração da Relação de Confiança

## SimpleSAMLphp

Para configurar o *proxy* utilizando **SimpleSAMLphp**, siga os passos abaixo:

1. Acesse a interface administrativa do SimpleSAMLphp:  

**https://FQDN_SIMPLESAMLPHP/simplesaml/module.php/admin/federation**

2. Vá até **XML to SimpleSAMLphp metadata converter**.  
3. Copie o metadado do Keycloak, disponível geralmente em:  

**https://FQDN_KEYCLOAK/realms/NAME/broker/NAME/endpoint/descriptor**

4. Cole o metadado copiado no conversor e gere o formato correto.  
5. Copie o metadado formatado e cole no arquivo de configuração:  

```
/var/simplesamlphp/metadata/saml20-sp-remote.php
```



---

## Shibboleth

Para configurar o *proxy* utilizando **Shibboleth**, siga os passos abaixo:

1. Faça o download do metadado do Keycloak com `wget`:  
```sh
wget -O /opt/shibboleth-idp/metadata/keycloak.xml https://FQDN_KEYCLOAK/realms/NAME/broker/NAME/endpoint/descriptor
```

2. Edite o arquivo de configuração de metadados:

```
/opt/shibboleth-idp/conf/metadata-providers.xml
```

3. Adicione a seguinte entrada no arquivo:

> ```
> <MetadataProvider id="keycloak-metadata"
>         xsi:type="FilesystemMetadataProvider"
>         metadataFile="%{idp.home}/metadata/keycloak.xml" />
> ```

