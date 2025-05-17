# Guia de Configuração do Azure com Shibboleth 5


Este guia apresenta as instruções para integrar o Shibboleth IdP com o Microsoft Entra ID (Azure), utilizando o fluxo de autenticação SAML. A configuração ocorre em **duas etapas principais**:


1. 🔐 **Trust Tasks** — Estabelece a confiança entre o IdP e o Entra ID, por meio do registro de metadados, certificados e configuração da aplicação no Azure.
   - `Trust Task 1`: Adição do bloco SPSSODescriptor no metadado do IdP
   - `Trust Task 2`: Criação da aplicação no Entra ID
   - `Trust Task 3`: Registro local do IdP do Entra ID
   - `Trust Task 4`: Configuração de atributos no Azure

2. 🔁 **Proxy Tasks** — Ativa e configura o proxy SAML no IdP para funcionar com o Entra ID como IdP remoto.

   - `Proxy Task 1`: Ativação do fluxo SAML no authn.properties
   - `Proxy Task 2`: Filtro de atributos para os claims recebidos
   - `Proxy Task 3`: Mapeamento e interpretação dos atributos do Azure
   - `Proxy Task 4`: Normalização do username com base no atributo recebido
   - `Proxy Task 5`: Definições de atributos e conectores no resolver

> 📚 Este guia é baseado em: [SAML Proxying EntraID / Azure with the Shibboleth IdP](https://shibboleth.atlassian.net/wiki/spaces/KB/pages/2783936889/SAML+Proxying+EntraID+Azure+with+the+Shibboleth+IdP)

**Importante:** Faça backup dos arquivos do IdPv5 antes de iniciar, pois diversos arquivos serão modificados.

 
## 🔐 Trust Tasks

###  🔐  Trust Task 1
Atualizar o metadado do IdP adicionando o bloco **SPSSODescriptor** em:

```bash
/opt/shibboleth-idp/metadata/idp-metadata.xml
```
  Lembre-se de adicionar ao bloco `“Certificate”` o certificado utilizado no metadado.
```xml
  <!-- New SP block START →

	<SPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
 
    	<KeyDescriptor>
        	<ds:KeyInfo>
            	<ds:X509Data>
                	<ds:X509Certificate>
                	AQUI VAI SEU CERTIFICADO DE ASSINATURA E CRIPTOGRAFIA
                	</ds:X509Certificate>
            	</ds:X509Data>
        	</ds:KeyInfo>
    	</KeyDescriptor>
 
	<AssertionConsumerService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://FQDN/idp/profile/Authn/SAML2/POST/SSO" index="0"/>
</SPSSODescriptor>


<!-- New SP block END -->
```
### 🔐 Trust Task 2

Criar uma conta no EntraID e posteriormente, criar uma aplicação em:

**`choose Enterprise Applications` > `Create New Application` > `Non Gallery Application` > `Actually create one` > `select SAML Sign On` > `select SAML Sign  On settings`**

- Para criação utilize as seguintes informações:
	- EntityID: https://FQDN/idp/shibboleth
	- Assertion Consumer Service (ACS) URL: https://FQDN/idp/profile/Authn/SAML2/POST/SSO

### 🔐 Trust Task 3
Registrar o IdP do EntraID localmente em: 
```bash
/opt/shibboleth-idp/conf/metadata-providers.xml
```
  Da seguinte forma:
```xml
<MetadataProvider id="AzureAD-idp-metadata"
    	xsi:type="FilesystemMetadataProvider"
    	metadataFile="%{idp.home}/metadata/AzureAD-idp.xml" />
```
  O metadado presente no diretório **metadataFile** pode ser semelhante ao visualizado abaixo:
  ```xml
  <?xml version="1.0" encoding="utf-8"?>


<EntityDescriptor xmlns="urn:oasis:names:tc:SAML:2.0:metadata" xmlns:shibmd="urn:mace:shibboleth:metadata:1.0"  ID="_99d2d659-7c5b-4441-92eb-eadeaf026bc4" entityID="https://sts.windows.net/d0b2d7a2-9968-4e6f-af18-d0e442897c9e/">


  <Signature xmlns="http://www.w3.org/2000/09/xmldsig#">
	<SignedInfo>
  	<CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
  	<SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"/>
  	<Reference URI="#_99d2d659-7c5b-4441-92eb-eadeaf026bc4">
    	<Transforms>
      	<Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"/>
      	<Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"/>
    	</Transforms>
    	<DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
    	<DigestValue>sLD8bGhUc7ikLd6cVH8KKdm7RvjvN1lFpYk64526xuo=</DigestValue>
  	</Reference>
	</SignedInfo>
	<SignatureValue>c4PYJ1I2/XC2gc6OFVUrQBXiSJkc5p1fMXuQRim0/ywbanh3lqflKiyofnbszGyTlOO77gDkFINgwrOtXuyCmoPh7Q3IroOnIK7xYcKB6l6BdXrY5h9CrAs9hQ7uAIPuntulqe4oUP4gJcvMiqWzUrv9kraY/cnxhHj7NmG7ELkjA2JKN1W0NNsIuuWdDG6lp3kq8WvnoKN+kdIrkWsy5Azgo8oBHKu2qM6mmC8lDhOfSGADRIQyaMQ62teo9Pzj88WYxLzvPm3ko5oMl9MoLf8CtJmK/XQI+UH+713WkEaNjPOMH/wY3WPHna53QbUy7elXMkSJ9XBajA+SVH99Kg==</SignatureValue>
	<ds:KeyInfo xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
  	<ds:X509Data>
    	<ds:X509Certificate>MIIC8DCCAdigAwIBAgIQXrmahQ0uT4NJFTFCs3R9hzANBgkqhkiG9w0BAQsFADA0MTIwMAYDVQQDEylNaWNyb3NvZnQgQXp1cmUgRmVkZXJhdGVkIFNTTyBDZXJ0aWZpY2F0ZTAeFw0yNDEwMjIxNDQxNDRaFw0yNzEwMjIxNDQxNDRaMDQxMjAwBgNVBAMTKU1pY3Jvc29mdCBBenVyZSBGZWRlcmF0ZWQgU1NPIENlcnRpZmljYXRlMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA4VVtqUweWuwD8bJP9EbhseBFsTTgcgaWxk488JWOBT21j13UQzvihIMunGV3zVAF6rnxqMJALsWp20qXFgjnBI+FT1UhPRKnopdbeEKzYXs8Dpr0Y7z2tbv+Vy/7L58/oLdoPQzmJZBjMQRxmbVPQMJ/xuj15zhiPaNYvqhm/KwcjwEZdZi5f+AXjTOliVUXzmI9ZKVKL1wW/VkGCFyQMSkzKZwzFB2UCOmgsZ9gfpnXeHrKk3II6+/laHGEs8d7eHGBb7ZEbkRespy/G/+rPW+vSSxo2Y6i9xvz3Rs1gptNuGHSAZpcnnfSl1rSZs0abYgWZ/sa8bdqDhlgLFM6xQIDAQABMA0GCSqGSIb3DQEBCwUAA4IBAQBfWLwIhJS2Ti1fEDNpqK0EYBeVzsYr4bFIL7aMIRss23zl8CxzA1lNRI6rzRp4fLgqHW8gYz+K8K+QUfhueRFRHqIwebVv+tGOOtSyu8hGSg4nL2nZtCx4c3QREAjoMWdxK+EvOnN2rSwUtTOv9lLAljeptn1i6C7yPJPP/3PEdzmwbunKpOkx2R/zM1wrfypWbWmwEryIM5Iab1FlOXhTdL89FQNoFoPTB/eNL23Crf2xUVOKdqeZvNhXsklmJYRzEFcNtfepUe3b0vNcQevcQ8DxSoFQ355vYX8hyR1XQQnlorAMISItzdyeI6xXy3M2jPID33aH9PCoTGJoCq0y</ds:X509Certificate>
  	</ds:X509Data>
	</ds:KeyInfo>
  </Signature>
  <RoleDescriptor xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:fed="http://docs.oasis-open.org/wsfed/federation/200706" xsi:type="fed:SecurityTokenServiceType" protocolSupportEnumeration="http://docs.oasis-open.org/wsfed/federation/200706">
	<KeyDescriptor use="signing">
  	<KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
    	<X509Data>
      	<X509Certificate>MIIC8DCCAdigAwIBAgIQXrmahQ0uT4NJFTFCs3R9hzANBgkqhkiG9w0BAQsFADA0MTIwMAYDVQQDEylNaWNyb3NvZnQgQXp1cmUgRmVkZXJhdGVkIFNTTyBDZXJ0aWZpY2F0ZTAeFw0yNDEwMjIxNDQxNDRaFw0yNzEwMjIxNDQxNDRaMDQxMjAwBgNVBAMTKU1pY3Jvc29mdCBBenVyZSBGZWRlcmF0ZWQgU1NPIENlcnRpZmljYXRlMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA4VVtqUweWuwD8bJP9EbhseBFsTTgcgaWxk488JWOBT21j13UQzvihIMunGV3zVAF6rnxqMJALsWp20qXFgjnBI+FT1UhPRKnopdbeEKzYXs8Dpr0Y7z2tbv+Vy/7L58/oLdoPQzmJZBjMQRxmbVPQMJ/xuj15zhiPaNYvqhm/KwcjwEZdZi5f+AXjTOliVUXzmI9ZKVKL1wW/VkGCFyQMSkzKZwzFB2UCOmgsZ9gfpnXeHrKk3II6+/laHGEs8d7eHGBb7ZEbkRespy/G/+rPW+vSSxo2Y6i9xvz3Rs1gptNuGHSAZpcnnfSl1rSZs0abYgWZ/sa8bdqDhlgLFM6xQIDAQABMA0GCSqGSIb3DQEBCwUAA4IBAQBfWLwIhJS2Ti1fEDNpqK0EYBeVzsYr4bFIL7aMIRss23zl8CxzA1lNRI6rzRp4fLgqHW8gYz+K8K+QUfhueRFRHqIwebVv+tGOOtSyu8hGSg4nL2nZtCx4c3QREAjoMWdxK+EvOnN2rSwUtTOv9lLAljeptn1i6C7yPJPP/3PEdzmwbunKpOkx2R/zM1wrfypWbWmwEryIM5Iab1FlOXhTdL89FQNoFoPTB/eNL23Crf2xUVOKdqeZvNhXsklmJYRzEFcNtfepUe3b0vNcQevcQ8DxSoFQ355vYX8hyR1XQQnlorAMISItzdyeI6xXy3M2jPID33aH9PCoTGJoCq0y</X509Certificate>
    	</X509Data>
  	</KeyInfo>
	</KeyDescriptor>
	<fed:ClaimTypesOffered>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
    	<auth:DisplayName>Name</auth:DisplayName>
    	<auth:Description>The mutable display name of the user.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier">
    	<auth:DisplayName>Subject</auth:DisplayName>
    	<auth:Description>An immutable, globally unique, non-reusable identifier of the user that is unique to the application for which a token is issued.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname">
    	<auth:DisplayName>Given Name</auth:DisplayName>
    	<auth:Description>First name of the user.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname">
    	<auth:DisplayName>Surname</auth:DisplayName>
    	<auth:Description>Last name of the user.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/identity/claims/displayname">
    	<auth:DisplayName>Display Name</auth:DisplayName>
    	<auth:Description>Display name of the user.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/identity/claims/nickname">
    	<auth:DisplayName>Nick Name</auth:DisplayName>
    	<auth:Description>Nick name of the user.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationinstant">
    	<auth:DisplayName>Authentication Instant</auth:DisplayName>
    	<auth:Description>The time (UTC) when the user is authenticated to Windows Azure Active Directory.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationmethod">
    	<auth:DisplayName>Authentication Method</auth:DisplayName>
    	<auth:Description>The method that Windows Azure Active Directory uses to authenticate users.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/identity/claims/objectidentifier">
    	<auth:DisplayName>ObjectIdentifier</auth:DisplayName>
    	<auth:Description>Primary identifier for the user in the directory. Immutable, globally unique, non-reusable.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/identity/claims/tenantid">
    	<auth:DisplayName>TenantId</auth:DisplayName>
    	<auth:Description>Identifier for the user's tenant.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/identity/claims/identityprovider">
    	<auth:DisplayName>IdentityProvider</auth:DisplayName>
    	<auth:Description>Identity provider for the user.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress">
    	<auth:DisplayName>Email</auth:DisplayName>
    	<auth:Description>Email address of the user.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/ws/2008/06/identity/claims/groups">
    	<auth:DisplayName>Groups</auth:DisplayName>
    	<auth:Description>Groups of the user.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/identity/claims/accesstoken">
    	<auth:DisplayName>External Access Token</auth:DisplayName>
    	<auth:Description>Access token issued by external identity provider.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/ws/2008/06/identity/claims/expiration">
    	<auth:DisplayName>External Access Token Expiration</auth:DisplayName>
    	<auth:Description>UTC expiration time of access token issued by external identity provider.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/identity/claims/openid2_id">
    	<auth:DisplayName>External OpenID 2.0 Identifier</auth:DisplayName>
    	<auth:Description>OpenID 2.0 identifier issued by external identity provider.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/claims/groups.link">
    	<auth:DisplayName>GroupsOverageClaim</auth:DisplayName>
    	<auth:Description>Issued when number of user's group claims exceeds return limit.</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/ws/2008/06/identity/claims/role">
    	<auth:DisplayName>Role Claim</auth:DisplayName>
    	<auth:Description>Roles that the user or Service Principal is attached to</auth:Description>
  	</auth:ClaimType>
  	<auth:ClaimType xmlns:auth="http://docs.oasis-open.org/wsfed/authorization/200706" Uri="http://schemas.microsoft.com/ws/2008/06/identity/claims/wids">
    	<auth:DisplayName>RoleTemplate Id Claim</auth:DisplayName>
    	<auth:Description>Role template id of the Built-in Directory Roles that the user is a member of</auth:Description>
  	</auth:ClaimType>
	</fed:ClaimTypesOffered>
	<fed:SecurityTokenServiceEndpoint>
  	<wsa:EndpointReference xmlns:wsa="http://www.w3.org/2005/08/addressing">
    	<wsa:Address>https://login.microsoftonline.com/d0b2d7a2-9968-4e6f-af18-d0e442897c9e/wsfed</wsa:Address>
  	</wsa:EndpointReference>
	</fed:SecurityTokenServiceEndpoint>
	<fed:PassiveRequestorEndpoint>
  	<wsa:EndpointReference xmlns:wsa="http://www.w3.org/2005/08/addressing">
    	<wsa:Address>https://login.microsoftonline.com/d0b2d7a2-9968-4e6f-af18-d0e442897c9e/wsfed</wsa:Address>
  	</wsa:EndpointReference>
	</fed:PassiveRequestorEndpoint>
  </RoleDescriptor>
  <RoleDescriptor xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:fed="http://docs.oasis-open.org/wsfed/federation/200706" xsi:type="fed:ApplicationServiceType" protocolSupportEnumeration="http://docs.oasis-open.org/wsfed/federation/200706">
	<KeyDescriptor use="signing">
  	<KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
    	<X509Data>
      	<X509Certificate>MIIC8DCCAdigAwIBAgIQXrmahQ0uT4NJFTFCs3R9hzANBgkqhkiG9w0BAQsFADA0MTIwMAYDVQQDEylNaWNyb3NvZnQgQXp1cmUgRmVkZXJhdGVkIFNTTyBDZXJ0aWZpY2F0ZTAeFw0yNDEwMjIxNDQxNDRaFw0yNzEwMjIxNDQxNDRaMDQxMjAwBgNVBAMTKU1pY3Jvc29mdCBBenVyZSBGZWRlcmF0ZWQgU1NPIENlcnRpZmljYXRlMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA4VVtqUweWuwD8bJP9EbhseBFsTTgcgaWxk488JWOBT21j13UQzvihIMunGV3zVAF6rnxqMJALsWp20qXFgjnBI+FT1UhPRKnopdbeEKzYXs8Dpr0Y7z2tbv+Vy/7L58/oLdoPQzmJZBjMQRxmbVPQMJ/xuj15zhiPaNYvqhm/KwcjwEZdZi5f+AXjTOliVUXzmI9ZKVKL1wW/VkGCFyQMSkzKZwzFB2UCOmgsZ9gfpnXeHrKk3II6+/laHGEs8d7eHGBb7ZEbkRespy/G/+rPW+vSSxo2Y6i9xvz3Rs1gptNuGHSAZpcnnfSl1rSZs0abYgWZ/sa8bdqDhlgLFM6xQIDAQABMA0GCSqGSIb3DQEBCwUAA4IBAQBfWLwIhJS2Ti1fEDNpqK0EYBeVzsYr4bFIL7aMIRss23zl8CxzA1lNRI6rzRp4fLgqHW8gYz+K8K+QUfhueRFRHqIwebVv+tGOOtSyu8hGSg4nL2nZtCx4c3QREAjoMWdxK+EvOnN2rSwUtTOv9lLAljeptn1i6C7yPJPP/3PEdzmwbunKpOkx2R/zM1wrfypWbWmwEryIM5Iab1FlOXhTdL89FQNoFoPTB/eNL23Crf2xUVOKdqeZvNhXsklmJYRzEFcNtfepUe3b0vNcQevcQ8DxSoFQ355vYX8hyR1XQQnlorAMISItzdyeI6xXy3M2jPID33aH9PCoTGJoCq0y</X509Certificate>
    	</X509Data>
  	</KeyInfo>
	</KeyDescriptor>
	<fed:TargetScopes>
  	<wsa:EndpointReference xmlns:wsa="http://www.w3.org/2005/08/addressing">
    	<wsa:Address>https://sts.windows.net/d0b2d7a2-9968-4e6f-af18-d0e442897c9e/</wsa:Address>
  	</wsa:EndpointReference>
	</fed:TargetScopes>
	<fed:ApplicationServiceEndpoint>
  	<wsa:EndpointReference xmlns:wsa="http://www.w3.org/2005/08/addressing">
    	<wsa:Address>https://login.microsoftonline.com/d0b2d7a2-9968-4e6f-af18-d0e442897c9e/wsfed</wsa:Address>
  	</wsa:EndpointReference>
	</fed:ApplicationServiceEndpoint>
	<fed:PassiveRequestorEndpoint>
  	<wsa:EndpointReference xmlns:wsa="http://www.w3.org/2005/08/addressing">
    	<wsa:Address>https://login.microsoftonline.com/d0b2d7a2-9968-4e6f-af18-d0e442897c9e/wsfed</wsa:Address>
  	</wsa:EndpointReference>
	</fed:PassiveRequestorEndpoint>
  </RoleDescriptor>
  <Extensions>
  	<shibmd:Scope regexp="false">uemgoutlookcom.onmicrosoft.com</shibmd:Scope>
  </Extensions>
  <IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
<!--	<Extensions>-->
<!--    	<shibmd:Scope regexp="false">uemgoutlookcom.onmicrosoft.com</shibmd:Scope>-->
<!--	</Extensions>-->
	<KeyDescriptor use="signing">
  	<KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
    	<X509Data>
      	<X509Certificate>MIIC8DCCAdigAwIBAgIQXrmahQ0uT4NJFTFCs3R9hzANBgkqhkiG9w0BAQsFADA0MTIwMAYDVQQDEylNaWNyb3NvZnQgQXp1cmUgRmVkZXJhdGVkIFNTTyBDZXJ0aWZpY2F0ZTAeFw0yNDEwMjIxNDQxNDRaFw0yNzEwMjIxNDQxNDRaMDQxMjAwBgNVBAMTKU1pY3Jvc29mdCBBenVyZSBGZWRlcmF0ZWQgU1NPIENlcnRpZmljYXRlMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA4VVtqUweWuwD8bJP9EbhseBFsTTgcgaWxk488JWOBT21j13UQzvihIMunGV3zVAF6rnxqMJALsWp20qXFgjnBI+FT1UhPRKnopdbeEKzYXs8Dpr0Y7z2tbv+Vy/7L58/oLdoPQzmJZBjMQRxmbVPQMJ/xuj15zhiPaNYvqhm/KwcjwEZdZi5f+AXjTOliVUXzmI9ZKVKL1wW/VkGCFyQMSkzKZwzFB2UCOmgsZ9gfpnXeHrKk3II6+/laHGEs8d7eHGBb7ZEbkRespy/G/+rPW+vSSxo2Y6i9xvz3Rs1gptNuGHSAZpcnnfSl1rSZs0abYgWZ/sa8bdqDhlgLFM6xQIDAQABMA0GCSqGSIb3DQEBCwUAA4IBAQBfWLwIhJS2Ti1fEDNpqK0EYBeVzsYr4bFIL7aMIRss23zl8CxzA1lNRI6rzRp4fLgqHW8gYz+K8K+QUfhueRFRHqIwebVv+tGOOtSyu8hGSg4nL2nZtCx4c3QREAjoMWdxK+EvOnN2rSwUtTOv9lLAljeptn1i6C7yPJPP/3PEdzmwbunKpOkx2R/zM1wrfypWbWmwEryIM5Iab1FlOXhTdL89FQNoFoPTB/eNL23Crf2xUVOKdqeZvNhXsklmJYRzEFcNtfepUe3b0vNcQevcQ8DxSoFQ355vYX8hyR1XQQnlorAMISItzdyeI6xXy3M2jPID33aH9PCoTGJoCq0y</X509Certificate>
    	</X509Data>
  	</KeyInfo>
	</KeyDescriptor>
	<SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/d0b2d7a2-9968-4e6f-af18-d0e442897c9e/saml2"/>
	<SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/d0b2d7a2-9968-4e6f-af18-d0e442897c9e/saml2"/>
	<SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://login.microsoftonline.com/d0b2d7a2-9968-4e6f-af18-d0e442897c9e/saml2"/>
  </IDPSSODescriptor>
</EntityDescriptor>
  ```
  ### 🔐 Trust Task 4
  Configurar no **EntraID** a entrega de atributos para o IdP, na seção **User Attributes & Claims.**


## 🔁 Proxy Tasks

### 🔁 Proxy Task 1
Mudar o fluxo para SAML, em:

```bash
/opt/shibboleth-idp/conf/authn/authn.properties
```
Configurando da seguinte forma:
```bash
idp.authn.flows=SAML

idp.authn.SAML.proxyEntityID = https://sts.windows.net/d0b2d7a2-9968-4e6f-af18-d0e442897c9e/
```

### 🔁 Proxy Task 2
Atualizar o filtro de atributos em:
```bash
/opt/shibboleth-idp/conf/attribute-filter.xml
```
Para ficar da seguinte forma:
```xml
[...]

<AttributeFilterPolicy id="FilterPolicyObject-Proxy-FromAzure-byIssuer-Type">
	<PolicyRequirementRule xsi:type="Issuer" value="https://sts.windows.net/d0b2d7a2-9968-4e6f-af18-d0e442897c9e/" />
     	 
	<AttributeRule attributeID="azureDisplayname" permitAny="true" />
	<AttributeRule attributeID="azureGivenname" permitAny="true" />
	<AttributeRule attributeID="azureSurname" permitAny="true" />
	<AttributeRule attributeID="azureAuthnmethodsreferences" permitAny="true" />
	<AttributeRule attributeID="azureIdentityprovider" permitAny="true" />
	<AttributeRule attributeID="azureTenantid" permitAny="true" />
	<AttributeRule attributeID="azureEmailaddress" permitAny="true" />
	<AttributeRule attributeID="azureObjectidentifier" permitAny="true" />
	<AttributeRule attributeID="azureGroups" permitAny="true" />
	<AttributeRule attributeID="azureName">
    	<PermitValueRule xsi:type="ScopeMatchesShibMDScope" />
	</AttributeRule>
</AttributeFilterPolicy>
</AttributeFilterPolicyGroup>

```
### 🔁 Proxy Task 3
Atualizar o IdP para ele reconhecer os Claims do AzureID. Crie um arquivo **AzureClaims.xml** em:
```bash
/opt/shibboleth-idp/attributes/azureClaims.xml
```
O arquivo poderá ser semelhante ao visualizado abaixo:
```xml
  <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   	xmlns:context="http://www.springframework.org/schema/context"
   	xmlns:util="http://www.springframework.org/schema/util"
   	xmlns:p="http://www.springframework.org/schema/p"
   	xmlns:c="http://www.springframework.org/schema/c"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                       	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                       	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd"
                       	 
   	default-init-method="initialize"
   	default-destroy-method="destroy">
 
	<bean parent="shibboleth.TranscodingRuleLoader">
	<constructor-arg>
	<list>
    	<!-- claims relevant to person record -->
    	<bean parent="shibboleth.TranscodingProperties">
        	<property name="properties">
            	<props merge="true">
                	<prop key="id">azureName</prop>
                	<prop key="transcoder">SAML2ScopedStringTranscoder</prop>
                	<prop key="saml2.name">http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name</prop>
                	<prop key="saml2.nameFormat">urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified</prop>
                	<prop key="displayName.en">Name</prop>
                	<prop key="description.en">Azure UPN of an account expected to be scoped thus transcoded that way</prop>
            	</props>
        	</property>
    	</bean>
    	<bean parent="shibboleth.TranscodingProperties">
        	<property name="properties">
            	<props merge="true">
                	<prop key="id">azureEmailaddress</prop>
                	<prop key="transcoder">SAML2StringTranscoder</prop>
                	<prop key="saml2.name">http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress</prop>
                	<prop key="saml2.nameFormat">urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified</prop>
                	<prop key="displayName.en">Mail</prop>
                	<prop key="description.en">Azure emailaddress field of an account</prop>
            	</props>
        	</property>
    	</bean>
    	<bean parent="shibboleth.TranscodingProperties">
        	<property name="properties">
            	<props merge="true">
                	<prop key="id">azureDisplayname</prop>
                	<prop key="transcoder">SAML2StringTranscoder</prop>
                	<prop key="saml2.name">http://schemas.microsoft.com/identity/claims/displayname</prop>
                	<prop key="saml2.nameFormat">urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified</prop>
                	<prop key="displayName.en">Mail</prop>
                	<prop key="description.en">Azure displayname field of an account</prop>
            	</props>
        	</property>
    	</bean>
    	<bean parent="shibboleth.TranscodingProperties">
        	<property name="properties">
            	<props merge="true">
                	<prop key="id">azureGivenname</prop>
                	<prop key="transcoder">SAML2StringTranscoder</prop>
                	<prop key="saml2.name">http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname</prop>
                	<prop key="saml2.nameFormat">urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified</prop>
                	<prop key="displayName.en">Given name</prop>
                	<prop key="description.en">Azure given name of an account</prop>
            	</props>
        	</property>
    	</bean>
    	<bean parent="shibboleth.TranscodingProperties">
        	<property name="properties">
            	<props merge="true">
                	<prop key="id">azureSurname</prop>
                	<prop key="transcoder">SAML2StringTranscoder</prop>
                	<prop key="saml2.name">http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname</prop>
                	<prop key="saml2.nameFormat">urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified</prop>
                	<prop key="displayName.en">Surname</prop>
                	<prop key="description.en">Azure surname of an account</prop>
            	</props>
        	</property>
    	</bean>
 
    	<!-- default claims from Azure for any entity -->
    	<bean parent="shibboleth.TranscodingProperties">
        	<property name="properties">
            	<props merge="true">
                	<prop key="id">azureTenantid</prop>
                	<prop key="transcoder">SAML2StringTranscoder</prop>
                	<prop key="saml2.name">http://schemas.microsoft.com/identity/claims/tenantid</prop>
                	<prop key="saml2.nameFormat">urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified</prop>
                	<prop key="displayName.en">Tenant ID</prop>
                	<prop key="description.en">Azure tenantid</prop>
            	</props>
        	</property>
    	</bean>
    	<bean parent="shibboleth.TranscodingProperties">
        	<property name="properties">
            	<props merge="true">
                	<prop key="id">azureObjectidentifier</prop>
                	<prop key="transcoder">SAML2StringTranscoder</prop>
                	<prop key="saml2.name">http://schemas.microsoft.com/identity/claims/objectidentifier</prop>
                	<prop key="saml2.nameFormat">urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified</prop>
                	<prop key="displayName.en">Object Identifier</prop>
                	<prop key="description.en">Azure object identifier of an account</prop>
            	</props>
        	</property>
    	</bean>
     	<bean parent="shibboleth.TranscodingProperties">
        	<property name="properties">
            	<props merge="true">
                	<prop key="id">azureIdentityprovider</prop>
                	<prop key="transcoder">SAML2StringTranscoder</prop>
                	<prop key="saml2.name">http://schemas.microsoft.com/identity/claims/identityprovider</prop>
                	<prop key="saml2.nameFormat">urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified</prop>
                	<prop key="displayName.en">Identity Provider entityid</prop>
                	<prop key="description.en">Azure entityID of the tenant in Azure</prop>
            	</props>
        	</property>
    	</bean>
    	<bean parent="shibboleth.TranscodingProperties">
        	<property name="properties">
            	<props merge="true">
                	<prop key="id">azureAuthnmethodsreferences</prop>
                	<prop key="transcoder">SAML2StringTranscoder</prop>
                	<prop key="saml2.name">http://schemas.microsoft.com/claims/authnmethodsreferences</prop>
                	<prop key="saml2.nameFormat">urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified</prop>
                	<prop key="displayName.en">Authnmethodsreferences</prop>
                	<prop key="description.en">Azure authentication method (password, modern auth? etc)</prop>
            	</props>
        	</property>
    	</bean>
    	<bean parent="shibboleth.TranscodingProperties">
        	<property name="properties">
            	<props merge="true">
                	<prop key="id">azureGroups</prop>
                	<prop key="transcoder">SAML2StringTranscoder</prop>
                	<prop key="saml2.name">http://schemas.microsoft.com/ws/2008/06/identity/claims/groups</prop>
                	<prop key="saml2.nameFormat">urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified</prop>
                	<prop key="displayName.en">Groups</prop>
                	<prop key="description.en">Azure Group Membership</prop>
            	</props>
        	</property>
    	</bean>
 
 
 
	</list>
	</constructor-arg>
	</bean>
	 
</beans>
```
Atualizar o arquivo presente em:
```bash
/opt/shibboleth-idp/attributes/default-rules.xml
```
Para incluir os novos claims:	
 ```xml
<import resource="azureClaims.xml" />
 ```
E adicionar um novo **DataConnector** em:
  ```bash
  /opt/shibboleth-idp/conf/attribute-resolver.xml
  ```
  Da seguinte maneira:
```xml
  <DataConnector id="passthroughAttributes" xsi:type="Subject"
	exportAttributes="azureName azureEmailaddress azureDisplayname azureGivenname azureSurname azureTenantid azureObjectidentifier azureIdentityprovider azureAuthnmethodsreferences" />
```
  ### 🔁 Proxy Task 4
  Realizar a extração do _username_ normalizado no fluxo SAML ao atualizar o seguinte arquivo:
```bash
/opt/shibboleth-idp/conf/c14n/subject-c14n.properties
```
Adicionar as seguintes informações:
```bash
  idp.c14n.attribute.attributesToResolve=  azureName,azureEmailaddress,azureDisplayname,azureGivenname,azureSurname,azureTenantid,azureObjectidentifier,azureIdentityprovider,azureAuthnmethodsreferences


#  and then select a principal name from
idp.c14n.attribute.attributeSourceIds = azureName


# Allows direct use of attributes via SAML proxy authn, bypasses resolver
idp.c14n.attribute.resolveFromSubject = true
idp.c14n.attribute.resolutionCondition=  shibboleth.Conditions.FALSE


#
# Specific to this KB article, this is the regex used in the next step for subject-c14n.xml
idp.c14.saml.proxy.regex.for.principal=^(.+)@uemgoutlookcom\.onmicrosoft\.com$
```
  Após isso, em: 
```bash
/opt/shibboleth-idp/conf/c14n/subject-c14n.xml
```
  Descomentar o seguinte bloco:
```xml
      	<bean id="c14n/attribute" parent="shibboleth.PostLoginSubjectCanonicalizationFlow" />
```
  E adicionar a seguinte informação (antes da finalização do bloco **beans**):
  ```xml
  [...]    
<util:list id="shibboleth.c14n.attribute.Transforms">  
   	<bean parent="shibboleth.Pair" p:first="%{idp.c14.saml.proxy.regex.for.principal}" p:second="$1" />
	</util:list>
</beans>
  ```
### 🔁 Proxy Task 5
  Configurar a passagem de atributos em:
```bash
/opt/shibboleth-idp/conf/attribute-resolver.xml
```
  Para ficar da seguinte forma:
```xml
  <!-- ========================================== -->
	<!--  	Attribute Definitions             	-->
	<!-- ========================================== -->



	<AttributeDefinition xsi:type="Simple" id="userPrincipalName">
    	<InputDataConnector ref="passthroughAttributes" attributeNames="azureName" />
	</AttributeDefinition>

<AttributeDefinition xsi:type="Simple" id="eduPersonPrincipalName">
    	<InputDataConnector ref="passthroughAttributes" attributeNames="azureName" />
	</AttributeDefinition>
 
 
<AttributeDefinition xsi:type="Simple" id="sn">
    	<InputDataConnector ref="passthroughAttributes" attributeNames="azureSurname" />
	</AttributeDefinition>
	 
<AttributeDefinition xsi:type="Simple" id="givenName">
    	<InputDataConnector ref="passthroughAttributes" attributeNames="azureGivenname" />
	</AttributeDefinition>
 
<AttributeDefinition xsi:type="Simple" id="mail">
    	<InputDataConnector ref="passthroughAttributes" attributeNames="azureEmailaddress" />
	</AttributeDefinition>
 
<AttributeDefinition xsi:type="Simple" id="displayName">
    	<InputDataConnector ref="passthroughAttributes" attributeNames="azureDisplayname" />
	</AttributeDefinition>
 
	<AttributeDefinition xsi:type="Simple" id="cn">
    	<InputDataConnector ref="passthroughAttributes" attributeNames="azureDisplayname" />
	</AttributeDefinition>

    
	<!-- ========================================== -->
	<!--  	Data Connectors                   	-->
	<!-- ========================================== -->

	<DataConnector id="passthroughAttributes" xsi:type="Subject"
	exportAttributes="azureName azureEmailaddress azureDisplayname azureGivenname azureSurname azureTenantid azureObjectidentifier azureIdentityprovider azureAuthnmethodsreferences" />
```
  
Pronto! Agora basta testar seu IdP utilizando um SP que tenha relação de confiança com a entidade.