# Proxies em Ambientes de Gerenciamento de Identidades: Estudo de Casos e Avaliação Prática

Este repositório contém as documentações e orientações do ambiente de degustação e dos proxies mencionados no artigo **Proxies em Ambientes de Gerenciamento de Identidades: Estudo de Casos e Avaliação Prática**.  

## 🎯 Ambiente de Degustação

Para acessar o ambiente de degustação:

1️⃣ Acesse: [🔗 keycloak.gidlab.rnp.br](https://keycloak.gidlab.rnp.br)

2️⃣ Clique em **"Administration Console"**

3️⃣ Insira as credenciais:


  4️⃣ No canto superior esquerdo, troque o **REALM** de `master` para `SBRC2025`
  5️⃣ Vá até **Clients**
  6️⃣ Clique na **Home URL** da conta
  7️⃣ Clique em **Sign in** (canto superior direito)
  8️⃣ Escolha o proxy desejado em **"Or sign in with"**

**Contas para utilização dos IdP3 e IdP4**

- **Usuário:** `aluno`
- **Senha:** `aluno@idpNUMERO` -->
1️⃣ Acesse: [🔗 keycloak.gidlab.rnp.br/realms/SBRC2025/account/#/](https://keycloak.gidlab.rnp.br/realms/SBRC2025/account/#/)

2️⃣ No canto superior direito da página, clique em 'Sign in'

3️⃣ Escolha o proxy desejado em **"Or sign in with"**

3️⃣ Caso escolha o SATOSA ou o SSPHP, será redirecionado para um serviço de descoberta onde deverá escolher um IdP para autenticação. Os IdPs disponíveis para teste são: IdP3 e IdP4

**Contas para utilização do IdP3**

- **Usuário:** `aluno`
- **Senha:** `aluno@idp3`

**Contas para utilização do IdP4**

- **Usuário:** `aluno`
- **Senha:** `aluno@idp4`


**Conta para utilização do Entra ID**
_Será necessário trocar a senha inicial e registrar um token no Microsoft Autenticator._

* **Usuário 1:** `sbrc2025-01@uemgoutlookcom.onmicrosoft.com` **Senha inicial:** `Suha702122`

* **Usuário 2:** `sbrc2025-02@uemgoutlookcom.onmicrosoft.com` **Senha inicial:** `Jowa134074`

* **Usuário 3:** `sbrc2025-03@uemgoutlookcom.onmicrosoft.com` **Senha inicial:** `Dupu340104`

* **Usuário 4:** `sbrc2025-04@uemgoutlookcom.onmicrosoft.com` **Senha inicial:** `Foya273444`

* **Usuário 5:** `sbrc2025-05@uemgoutlookcom.onmicrosoft.com` **Senha inicial:** `Rapa083085`


Exemplo de fluxo:

![FLuxo](./fluxo.png)

---

## Configurando sua própria instância de *Proxies*

### 🔹Shibboleth e Microsoft Entra ID

Para instalação siga os passos abaixo nesta ordem:   

1️⃣ [**Instalação do Shibboleth 5**](Instalação_Shibboleth_5.md)  

2️⃣[**Configuração do Azure com Shibboleth 5**](Configuração_do_Azure_com_Shibboleth_5.md)

---

### 🔹 SimpleSAMLphp  

Para instalação:

1️⃣ Siga [**Guia de Instalação e Configuração do SimpleSAMLphp como proxy SAML**](Guia_de_Instalação_e_Configuração_do_SimpleSAMLphp_como_proxy_SAML.md).   

2️⃣ Após a instalação, edite o arquivo:     

- `simplesamlphp-config/default-ssl.conf`

3️⃣ Crie o diretório **certs-apache** e faça o build do Docker:

```
docker build .
```

4️⃣ Suba o container com:

```
docker compose up
```

---

### 🔹SATOSA

Para instalação:

1️⃣ Siga [**Guia Instalação Satosa**](Guia_Instalação_Satosa.md).


## Configurando seu próprio ambiente de degustação

Para instalação:

1️⃣ Siga [**Guia Instalação Keycloak**](Guia_Instalação_Keycloak.md).

2️⃣ Realize as relações de confiança para o *proxy* desejado, seguindo os passos de [**Guia Relação de Confiança**](Guia_Relação_de_Confiança.md).
