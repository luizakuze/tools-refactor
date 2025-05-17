<!-- omit in toc -->
# Proxies em Ambientes de Gerenciamento de Identidades: Estudo de Casos e Avaliação Prática

Este repositório contém as documentações e orientações do ambiente de experimentação descrito no artigo **Proxies em Ambientes de Gerenciamento de Identidades: Estudo de Casos e Avaliação Prática**.

<!-- omit in toc -->
## 📚 Sumário 

- [🎯 Ambiente de Experimentação](#-ambiente-de-experimentação)
  - [⚙️ Ambiente de configuração](#️-ambiente-de-configuração)
  - [👤 Ambiente do usuário final](#-ambiente-do-usuário-final)
- [🔐 Contas de Utilização para os Proxies](#-contas-de-utilização-para-os-proxies)
  - [Entra ID](#entra-id)
  - [SimpleSAMLphp e SATOSA](#simplesamlphp-e-satosa)
- [🔁 Exemplo de Fluxo](#-exemplo-de-fluxo)
- [🧩 Configuração costumizada de instância de proxies](#-configuração-costumizada-de-instância-de-proxies)
  - [Shibboleth com Microsoft Entra ID](#shibboleth-com-microsoft-entra-id)
  - [SimpleSAMLphp](#simplesamlphp)
  - [SATOSA](#satosa)
- [🧩 Configuração costumizada do ambiente de degustação](#-configuração-costumizada-do-ambiente-de-degustação)

 

## 🎯 Ambiente de Experimentação

### ⚙️ Ambiente de configuração

1. Acesse: [https://keycloak.gidlab.rnp.br](https://keycloak.gidlab.rnp.br).
2. Clique em **`Administration Console`**.
3. Insira as credenciais administrativas.
4. No canto superior esquerdo, altere a **`REALM`** de `master` para `SBRC2025`.
5. Navegue até **`Clients`** e clique na **`Home URL`** da conta.
6. Clique em **`Sign in`** (canto superior direito).
7. Escolha o proxy desejado em **"`Or sign in with`"**.


### 👤 Ambiente do usuário final

1. Acesse: [https://keycloak.gidlab.rnp.br/realms/SBRC2025/account/#/](https://keycloak.gidlab.rnp.br/realms/SBRC2025/account/#/).
2. Clique em **`Sign in`**.
3. Escolha o proxy desejado em **`Or sign in with`**.
4. Acesse a seção [Contas de Utilização para os Proxies](#4-contas-de-utiliza%C3%A7%C3%A3o-para-os-proxies-entra-id-simplesamlphp-e-satosa) e utilize uma das contas disponíveis, conforme o proxy selecionado.


## 🔐 Contas de Utilização para os Proxies

### Entra ID

> ⚠️ Troque a senha inicial no primeiro acesso e registre um token no Microsoft Authenticator.

| Usuário | Login                                        | Senha Inicial |
| ------- | -------------------------------------------- | ------------- |
| 1       | `sbrc2025-01@uemgoutlookcom.onmicrosoft.com` | `Suha702122`  |
| 2       | `sbrc2025-02@uemgoutlookcom.onmicrosoft.com` | `Jowa134074`  |
| 3       | `sbrc2025-03@uemgoutlookcom.onmicrosoft.com` | `Dupu340104`  |
| 4       | `sbrc2025-04@uemgoutlookcom.onmicrosoft.com` | `Foya273444`  |
| 5       | `sbrc2025-05@uemgoutlookcom.onmicrosoft.com` | `Rapa083085`  |

### SimpleSAMLphp e SATOSA

> Utilizam os mesmos provedores de identidade: **IdP3** e **IdP4**.

| Provedor | Usuário | Senha        |
| -------- | ------- | ------------ |
| IdP3     | `aluno` | `aluno@idp3` |
| IdP4     | `aluno` | `aluno@idp4` |


## 🔁 Exemplo de Fluxo

A figura abaixo exemplifica o fluxo de autenticação utilizando o proxy SATOSA. Ao selecionar a opção Satosa na tela inicial do Keycloak, o usuário é redirecionado para o serviço de descoberta da federação CAFe Expresso. Em seguida, escolhe um dos provedores de identidade disponíveis (neste caso, o IdP3) e realiza o login com suas credenciais, completando assim o fluxo.

<p align="center"><img src="./fluxo.png" alt="Fluxo" /></p>
 

## 🧩 Configuração costumizada de instância de proxies

Esta seção apresenta tutoriais independentes para que você possa configurar de forma costumizada e testar os proxies abordados no artigo.  

### Shibboleth com Microsoft Entra ID

1. [Instalação do Shibboleth 5](Instalação_Shibboleth_5.md)
2. [Configuração do Azure com Shibboleth 5](Configuração_do_Azure_com_Shibboleth_5.md)
 
### SimpleSAMLphp

1. [Guia de Instalação e Configuração do SimpleSAMLphp como proxy SAML](Guia_de_Instalação_e_Configuração_do_SimpleSAMLphp_como_proxy_SAML.md)
2. Editar o arquivo `simplesamlphp-config/default-ssl.conf`.
3. Criar o diretório `certs-apache`, acessar esse diretório e fazer o build:

    ```bash
    mkdir certs-apache
    cd certs-apache
    docker build .
    ```

4. Subir o container:

    ```bash
    docker compose up
    ``` 

### SATOSA

1. [Guia de Instalação do SATOSA](Guia_Instalação_Satosa.md)
 

## 🧩 Configuração costumizada do ambiente de degustação

1. [Guia de Instalação do Keycloak](Guia_Instalação_Keycloak.md)
2. Configure a relação de confiança com o proxy desejado seguindo: [Guia de Relação de Confiança](Guia_Relação_de_Confiança.md)
 
> 💡 Para mais detalhes sobre os casos de uso e fluxos de autenticação com cada proxy, consulte o artigo completo em PDF incluso neste repositório.
