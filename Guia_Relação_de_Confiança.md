# 🧩 Guia de Configuração da Relação de Confiança

A etapa de relação de confiança deverá ser executada no **Keycloak** e nos ambientes que são **Proxies Servers**.

Acesse as configurações do proxy que deseja testar:
- [Shibboleth IdP](#-shibboleth-idp-com-entraid)
- [SimpleSAMLphp](#-simplesamlphp)
- [SATOSA](#-satosa)
## 📌 Shibboleth IdP com EntraId

### 🔐 Keycloak

* Acessar o `Administration Console`;
* Autenticar com o usuário `admin`;
* Criar ou acessar o realm: `SBRC2025`;
* Ir em **Identity Providers** > **Add provider** > `SAML v2.0`;

Configurar:

* **Entity ID**:

  ```
  https://keycloak.gidlab.rnp.br/realms/SBRC2025/account/
  ```

* **SAML entity descriptor**:

  ```
  https://dev.cafeexpresso.rnp.br/idp/shibboleth
  ```

* Clicar em **Add**;

* Alterar para:

  * `NameID Format`: `Persistent`
  * `Principal Type`: `Subject NameID`
  * Ativar:

    ```
    Allow create
    HTTP-POST binding response
    HTTP-POST binding for AuthnRequest
    HTTP-POST binding logout
    Want AuthnRequests signed
    Want Assertions signed
    Want Assertions encrypted
    ```

* Clicar em **Save**.

### ↔️ Mapeamento (Mappers)

| Name     | Sync mode | Mapper type        | Friendly Name | User Attribute |
| -------- | --------- | ------------------ | ------------- | -------------- |
| Name     | Import    | Attribute Importer | givenName     | firstName      |
| Mail     | Import    | Attribute Importer | mail          | email          |
| LastName | Import    | Attribute Importer | sn            | lastName       |

### ⚙️ Configurar Shibboleth IdP

1. Editar `conf/metadata-providers.xml`:

   ```xml
   <MetadataProvider id="keycloak-metadata"
       xsi:type="FilesystemMetadataProvider"
       metadataFile="%{idp.home}/metadata/keycloak.xml" />
   ```
2. Salvar o metadata do Keycloak em `metadata/keycloak.xml`;
3. Reiniciar:

   ```
   sudo systemctl restart jetty.service
   ```

---

## 📌 SimpleSAMLphp

### 🔐 Keycloak

* Realm: `SBRC2025`;
* Identity Provider: `SAML v2.0`;
* Desmarcar: `Use entity descriptor`;
* Importar metadata:

  ```
  https://proxyssp-dev.gidlab.rnp.br/simplesaml/module.php/saml/idp/metadata
  ```
* Entity ID:

  ```
  https://keycloak.gidlab.rnp.br/realms/SBRC2025/account/
  ```
* Alterar para:

  * `NameID Format`: `Persistent`
  * `Principal Type`: `Subject NameID`
  * Ativar:

    ```
    Allow create
    HTTP-POST binding response
    HTTP-POST binding for AuthnRequest
    HTTP-POST binding logout
    Want AuthnRequests signed
    Want Assertions signed
    Want Assertions encrypted
    ```
* Clicar em **Save**.

### ↔️ Mapeamento (Mappers)

| Name     | Sync mode | Mapper type        | Attribute Name                     | Name Format            | User Attribute |
| -------- | --------- | ------------------ | ---------------------------------- | ---------------------- | -------------- |
| Name     | Import    | Attribute Importer | urn\:oid:2.5.4.42                  | ATTRIBUTE\_FORMAT\_URI | firstName      |
| Mail     | Import    | Attribute Importer | urn\:oid:0.9.2342.19200300.100.1.3 | ATTRIBUTE\_FORMAT\_URI | email          |
| LastName | Import    | Attribute Importer | urn\:oid:2.5.4.4                   | ATTRIBUTE\_FORMAT\_URI | lastName       |

### ⚙️ Configurar SSPHP

1. Acessar: `/simplesaml/module.php/admin/federation`;
2. Usar: `XML to SimpleSAMLphp metadata converter`;
3. Colar metadado de:

   ```
   https://FQDN_KEYCLOAK/realms/NAME/broker/NAME/endpoint/descriptor
   ```
4. Gerar conversão e salvar em:

   ```
   /var/simplesamlphp/metadata/saml20-sp-remote.php
   ```

---

## 📌 SATOSA

### 🔐 Keycloak

* Realm: `SBRC2025`;
* Identity Provider: `SAML v2.0`;
* Entity ID:

  ```
  https://keycloak.gidlab.rnp.br/realms/SBRC2025/account/
  ```
* Metadata (SAML entity descriptor):

  ```
  https://proxy.gidlab.rnp.br/Saml2IDP/proxy.xml
  ```
* Alterar:

  * `NameID Format`: `Persistent`
  * `Principal Type`: `Subject NameID`
  * Ativar:

    ```
    Allow create
    HTTP-POST binding response
    HTTP-POST binding for AuthnRequest
    HTTP-POST binding logout
    Want AuthnRequests signed
    Want Assertions signed
    Want Assertions encrypted
    ```
* Clicar em **Save**.

### ↔️ Mapeamento (Mappers)

| Name     | Sync mode | Mapper type        | Friendly Name | User Attribute |
| -------- | --------- | ------------------ | ------------- | -------------- |
| Name     | Import    | Attribute Importer | givenName     | firstName      |
| Mail     | Import    | Attribute Importer | mail          | email          |
| LastName | Import    | Attribute Importer | sn            | lastName       |

### ⚙️ Configurar SATOSA

1. Editar: `volume/plugins/frontends/saml2_frontend.yaml`:

   ```yaml
   metadata:
     local:
       - keycloak.xml
   ```
2. Salvar metadado do Keycloak em `volume/keycloak.xml`;
3. Reiniciar:

   ```
   docker compose restart
   ```
