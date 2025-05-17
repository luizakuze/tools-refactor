<!-- omit in toc -->
# Guia de Instalação do SATOSA


Este repositório fornece os arquivos e instruções para implantar uma instância do proxy `SATOSA` via `docker-compose`, utilizando uma imagem personalizada baseada no [repositório oficial](https://github.com/IdentityPython/SATOSA/tree/v8.4.0).

Além da estrutura de diretórios, o repositório inclui guias com exemplos de configuração e textos explicativos adaptados da wiki original, focados no cenário desta implantação.

<!-- omit in toc -->
## 📑 Sumário

- [ℹ️ Sobre](#ℹ️-sobre)
- [⚙️ Instalação](#️-instalação)
  - [⚙️ Pré-Requisitos](#️-pré-requisitos)
  - [⚙️ Procedimento de Instalação](#️-procedimento-de-instalação)
  - [⚙️ Iniciar contêiner](#️-iniciar-contêiner)
- [🔗 Relação de confiança](#-relação-de-confiança)
- [↔️ Mapeamento de Atributos](#️-mapeamento-de-atributos)
- [👥 Plugins Backend para Login Social](#-plugins-backend-para-login-social)
- [📦 Comandos úteis](#-comandos-úteis)


## ℹ️ Sobre 

O SATOSA é uma solução de proxy, mantida pela organização [Identity Python](https://github.com/IdentityPython), para tradução e mapeamento de atributos entre os principais protocolos de autenticação, tais como: SAML2, OpenID Connect e OAuth2. 
Essas funcionalidades permitem a integração dos serviços com os mecanismos de autenticação sem a necessidade de implementação de todos os protocolos. 
Mais informações sobre o SATOSA no arquivo referente à [Características e Arquitetura](satosa-docker/docs/conceitos.md).

## ⚙️ Instalação

- As etapas a seguir descrevem a implantação de uma instância do `SATOSA` com ativação dos plugins de `Backend` e `Frontend` para o protocolo SAML.

- Os arquivos de configuração no diretório `./volumes` apresentam um exemplo de uso do `SATOSA` como proxy SAML-to-SAML, onde a relação de confiança é estabelecida entre um SP e uma federação SAML por meio do proxy.

- Também são descritas opções para uso do `SATOSA` como proxy SAML com logins sociais.
 

### ⚙️ Pré-Requisitos
- [ ] Servidor com FQDN ou IP válido.
- [ ] Docker Engine e Docker Compose.


### ⚙️ Procedimento de Instalação

1. Baixar o repositório e acessar o diretório raíz do projeto
    ```
    git clone -n --depth=1 --filter=tree:0 https://github.com/luandalmazo/tools.git
    cd tools
    git sparse-checkout set --no-cone satosa-docker
    git checkout
    cd satosa-docker/
    ```

2. Antes de iniciar, altere os arquivos `.env` e `.satosa_env` com as informações do seu servidor:
      1. **.env**
          ```env
          DATA_VOL=./volumes         # Caminho local para os volumes de configuração
          SATOSA_PORT=443            # Porta HTTPS usada pelo SATOSA (ajuste se necessário)
          ```

      2.  **.satosa_env**
          ```env
          DATA_DIR=/satosa                           # Caminho dentro do contêiner onde os dados são montados
          PROXY_PORT=443                             # Porta exposta pelo proxy
          SATOSA_STATE_ENCRYPTION_KEY=trocar_enc_key_asdASD123  # Chave de criptografia para o estado interno (modifique!)
          SATOSA_USER_ID_HASH_SALT=trocar_user_id_salt          # Salt para geração de ID de usuário (modifique!)
          METADATA_DIR=/satosa/metadata              # Diretório onde os metadados serão armazenados
          ```


3. Para habilitar o TLS no próprio Gunicorn, presente na imagem docker, inserir arquivos de chave privada e certificado no diretório `./volumes/`, com os nomes `https.key` e `https.crt`
. Caso você não tenha esses arquivos, você pode utilizar o comando abaixo para gerar e colocar no diretório especificado:

    ```bash
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout ./volumes/https.key \
    -out ./volumes/https.crt \
    -subj "/C=XX/ST=Estado/L=Cidade/O=NomeDaOrganizacao/CN=dominio.exemplo/emailAddress=email@exemplo.com"
    ```
    > ✏️ A flag `-subj` define as informações do certificado. Substitua os valores conforme seu ambiente. 

4. Alterar as informações do frontend (IdP do Proxy) do arquivo `./volumes/plugins/frontends/saml2_frontend_gidlab.yaml.example` renomeando para `saml2_frontend.yaml`. 
    Das configurações nesse arquido:
    1. Propriedade `metadata` dentro da chave `idp_config`

        Aqui devem ser informados os metadados dos Provedores de Serviços (SPs) que utilizarão o proxy para se comunicar com a federação.
        ```yaml
        metadata:
        #   remote:
        #     - url: https://ds.cafeexpresso.rnp.br/metadata/ds-metadata.xml
            local: [gitlab-cafeexpresso3.xml]
        ```

    2. Alterar o `entityid`

        ```yaml
        entityid: <base_url>/<name>/shibboleth
        ```




5. Alterar informações do backend (SP do Proxy) no arquivo `./volumes/plugins/backends/saml2_frontend_gidlab.yaml.example` renomeando para `saml2_frontend.yaml`.

    1. Alterar o `entityid`

        ```yaml
        entityid: <base_url>/<name>/proxy_saml2_backend.xml
        ```

    2. Alterar `endpoints`

        ```yaml
        endpoints:
          assertion_consumer_service:
                - - <base_url>/<name>/acs/post
                  - urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST
              discovery_response:
                - - <base_url>/<name>/disco
                  - urn:oasis:names:tc:SAML:profiles:SSO:idp-discovery-protocol
        ```


6. Alterar informações de configurações gerais no arquivo `./volumes/proxy_conf.yaml`.

> 💡 Os arquivos configuram o proxy entre Federação CAFe Expresso e SPs com o protocolo SAML.

### ⚙️ Iniciar contêiner

Execute o comando abaixo para iniciar o serviço do proxy:

- Para acompanhar os logs no terminal:
  ```bash
  docker compose up
  ```

- Para executar em segundo plano (*modo daemon*):
  ```bash
  docker compose up -d
  ```

Após a inicialização do contêiner, o ideal é seguir diretamente para o procedimento de [🔗 Relação de confiança](#-relação-de-confiança), onde os metadados do SATOSA serão utilizados para configurar os demais participantes da federação.

## 🔗 Relação de confiança

Como o Proxy `SATOSA` desta configuração possui dois plugins SAML habilitados, é possível obter os metadados tanto do **Frontend** (IdP do Proxy) quanto do **Backend** (SP do Proxy). 

Após seguir o procedimento descrito em [⚙️ Procedimento de Instalação](#️-procedimento-de-instalação), siga o procedimento abaixo para a relação de confiança:

1. Obter os metadados:
   * Frontend - `https://FQDN/Saml2IDP/proxy`.
   * Backend - `https://FQDN/Saml2/proxy_saml2_backend`.
   
    > 📝 Altere o `FQDN` para o do servidor.
   
2. Encaminhar o metadado do `Backend` para o operador da federação, pois ele será responsável por registrar e aprovar o SP na federação.
    
    
## ↔️ Mapeamento de Atributos

Os arquivos atuais possuem as configurações necessárias para o mapeamento de todos os atributos presentes nos diretórios de usuários atualmente utilizados na Federação CAFe Expresso.

Ao todos são quatro arquivos que precisam ser configurados:
1. `volumes/plugins/backends/saml2_backend.yaml`
2. `volumes/plugins/frontends/saml2_frontend.yaml`
3. `volumes/attributemaps/saml_uri.py`
4. `volumes/internal_attributes.yaml`

Se houver necessidade de adicionar novos atributos seguir o procedimento em: [Adicionando novos atributos.](satosa-docker/docs/novosatributos.md)

## 👥 Plugins Backend para Login Social


- Os arquivos de exemplo estão localizados em `./volumes/plugins/backends/`.  
  Por exemplo, o arquivo `google_backend.yaml.example` deve ser renomeado para `google_backend.yaml`. Em seguida, preencha os campos obrigatórios com as credenciais da aplicação:

  ```yaml
  client_id: <client id>
  client_secret: <client secret>
  ```

- Após configurar o backend, inicie o contêiner e acompanhe os logs do SATOSA para verificar quais atributos estão sendo recebidos do provedor social (Google, Facebook etc.).

- Esses atributos devem ser mapeados para atributos SAML no arquivo internal_attributes.yaml.
Consulte a documentação ["Adicionando novos atributos"](./docs/novosatributos.md), especialmente a Etapa 3, para realizar esse mapeamento.

## 📦 Comandos úteis

Para executar comandos interativos dentro do contêiner do SATOSA, utilize:

```bash
docker exec -it satosa /bin/bash
```

Uma vez dentro do contêiner, você pode gerar os metadados com o seguinte comando:

```bash
# satosa-saml-metadata "${DATA_DIR}/proxy_conf.yaml" "${DATA_DIR}/frontend.key" "${DATA_DIR}/frontend.crt" --dir "${METADATA_DIR}" --split-frontend
```
> 💡 Este comando gera os metadados do Frontend e Backend do SATOSA, dividindo eles em arquivos separados no diretório especificado por ${METADATA_DIR}.