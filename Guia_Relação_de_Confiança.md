# Guia de Configuração da Relação de Confiança

A etapa de relação de confiança deverá ser executada no Keycloak e nos ambientes que são Proxies Servers.

## Shibboleth IdP com EntraId IDP:

Abrir o painel `Administration Console` do Keycloak;

Autenticar com usuário admin;

Criar novo realm: `SBRC2025`;

Se já criado, acessar o Realm "SBRC2025".


Ir nas configurações de "Identity providers" do Keycloak;

Adicionar um novo como "SAML v2.0";

Selecionar os nomes: Alias, Display Name.

Definir o "Service provider entity ID" com:
`https://keycloak.gidlab.rnp.br/realms/SBRC2025/account/`

Em `SAML entity descriptor` colocar o endereços onde se encontra o metadado do IdP que deseja adicionar:
`https://dev.cafeexpresso.rnp.br/idp/shibboleth`

Aguardar carregar todo o metadado. E clicar em "Add".
 
Voltar em "Identity providers" e selecionar o item criado.

Trocar o "NameID policy format" para `Persistent`.

Manter "Principal type" em `Subject NameID`.

Habilitar os seguintes itens:

```
Allow create
HTTP-POST binding response
HTTP-POST binding for AuthnRequest
HTTP-POST binding logout
Want AuthnRequests signed
   -Nos menus de escolha de algoritmos, poderá deixar os padrões já escolhidos.
Want Assertions signed
Want Assertions encrypted
```

E clicar em "Save".

Voltar em "Identity providers" e selecionar o item criado. E entrar no menu superior "Mappers" para realizar o mapeamento de atributos.
Clicar em "Add Mapper" para definir os seguintes itens:
| **Name** | **Sync mode override** |   **Mapper type**  | **Friendly Name** | **User Attribute Name** |
|:--------:|:----------------------:|:------------------:|:-----------------:|:-----------------------:|
|   Name   |         Import         | Attribute Importer |     givenName     |        firstName        |
|   Mail   |         Import         | Attribute Importer |        mail       |          email          |
| LastName |         Import         | Attribute Importer |         sn        |         lastName        |

Ao finalizar, clicar em "Save".

Em "Endpoints" "SAML 2.0 Service Provider Metadata" estará o metadado que deverá ser usado para realizar a "Relação de Confiança" no IdP Shibboleth.

No Shibboleth IdP V5 já configurado com o EntraId, deverá editar o arquivo "conf/metadata-providers.xml", adicionando a seguinte diretiva antes da linha `</MetadataProvider>`:
```
<MetadataProvider id="keycloak-metadata"
        xsi:type="FilesystemMetadataProvider"
        metadataFile="%{idp.home}/metadata/keycloak.xml" />
```

Depois deverá criar o arquivo `metadata/keycloak.xml` com as informações baixadas em "Endpoints" "SAML 2.0 Service Provider Metadata".

Após, deverá reiniciar o serviço Jetty do Shibboleth IdP:
`sudo systemctl restart jetty.service`




---
## SimpleSAMLphp:

Abrir o painel `Administration Console` do Keycloak;

Autenticar com usuário admin;

Criar novo realm: `SBRC2025`;

Se já criado, acessar o Realm "SBRC2025".


Ir nas configurações de "Identity providers" do Keycloak;

Adicionar um novo como "SAML v2.0";

Selecionar os nomes: Alias, Display Name.

Definir o "Service provider entity ID" com:
`https://keycloak.gidlab.rnp.br/realms/SBRC2025/account/`

Desabilitar a opção "Use entity descriptor".

Baixar o metadado do SSPhp e dicionar usando `Import config from file`.

Endereço do metadado: `https://proxyssp-dev.gidlab.rnp.br/simplesaml/module.php/saml/idp/metadata`

No novo "Service provider entity ID" criado, adicione o endereço:
`https://keycloak.gidlab.rnp.br/realms/SBRC2025/account/`


Trocar o "NameID policy format" para `Persistent`.

Manter "Principal type" em `Subject NameID`.

Habilitar os seguintes itens:

```
Allow create
HTTP-POST binding response
HTTP-POST binding for AuthnRequest
HTTP-POST binding logout
Want AuthnRequests signed
   -Nos menus de escolha de algoritmos, poderá deixar os padrões já escolhidos.
Want Assertions signed
Want Assertions encrypted
```

E clicar em "Save".

Voltar em "Identity providers" e selecionar o item criado. E entrar no menu superior "Mappers" para realizar o mapeamento de atributos.
Clicar em "Add Mapper" para definir os seguintes itens:

| **Name** | **Sync mode override** |   **Mapper type**  |         **Attribute Name**        |    **Name Format**   | **User Attribute Name** |
|:--------:|:----------------------:|:------------------:|:---------------------------------:|:--------------------:|:-----------------------:|
|   Name   |         Import         | Attribute Importer |          urn:oid:2.5.4.42         | ATTRIBUTE_FORMAT_URI | firstName               |
|   Mail   |         Import         | Attribute Importer | urn:oid:0.9.2342.19200300.100.1.3 | ATTRIBUTE_FORMAT_URI | email                   |
| LastName |         Import         | Attribute Importer |          urn:oid:2.5.4.4          | ATTRIBUTE_FORMAT_URI | lastName                |

Ao finalizar, clicar em "Save".


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

## SATOSA:

Abrir o painel `Administration Console` do Keycloak;

Autenticar com usuário admin;

Criar novo realm: `SBRC2025`;

Se já criado, acessar o Realm "SBRC2025".


Ir nas configurações de "Identity providers" do Keycloak;

Ir nas configurações de "Identity providers";

Adicionar um novo como "SAML v2.0";

Selecionar os nomes: Alias, Display Name.

Definir o "Service provider entity ID" com:
`https://keycloak.gidlab.rnp.br/realms/SBRC2025/account/`

Em "SAML entity descriptor" colocar o endereços onde se encontra o metadado do IdP que deseja adicionar:
`https://proxy.gidlab.rnp.br/Saml2IDP/proxy.xml`

Aguardar carregar todo o metadado. E clicar em "Add".
 
Voltar em "Identity providers" e selecionar o item criado.

Trocar o "NameID policy format" para `Persistent`.

Manter "Principal type" em `Subject NameID`.

Habilitar os seguintes itens:

```
Allow create
HTTP-POST binding response
HTTP-POST binding for AuthnRequest
HTTP-POST binding logout
Want AuthnRequests signed
   -Nos menus de escolha de algoritmos, poderá deixar os padrões já escolhidos.
Want Assertions signed
Want Assertions encrypted
```

E clicar em "Save".

Voltar em "Identity providers" e selecionar o item criado. E entrar no menu superior "Mappers" para realizar o mapeamento de atributos.
Clicar em "Add Mapper" para definir os seguintes itens:

| **Name** | **Sync mode override** |   **Mapper type**  | **Friendly Name** | **User Attribute Name** |
|:--------:|:----------------------:|:------------------:|:-----------------:|:-----------------------:|
|   Name   |         Import         | Attribute Importer |     givenName     |        firstName        |
|   Mail   |         Import         | Attribute Importer |        mail       |          email          |
| LastName |         Import         | Attribute Importer |         sn        |         lastName        |

Ao finalizar, clicar em "Save".

Em "Endpoints" "SAML 2.0 Service Provider Metadata" estará o metadado que deverá ser usado para realizar a "Relação de Confiança" no Satosa.

Com o Satosa já configurado com alguma federação, deverá editar o arquivo "volume/plugins/frontends/saml2_frontend.yaml", editando a diretiva de Metadado, por exemplo:
```
    metadata:
      local:
        - keycloak.xml
```

Depois deverá criar o arquivo `volume/keycloak.xml` com as informações baixadas em "Endpoints" "SAML 2.0 Service Provider Metadata".

Após, deverá reiniciar o serviço Satosa:
`docker compose restart`
