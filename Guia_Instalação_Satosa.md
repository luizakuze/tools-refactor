# SATOSA CONTAINER

Repositório com diretórios e arquivos para implantar uma instância do Proxy `SATOSA` em contêiner com `docker-compose`.
O contêiner utiliza uma imagem do `SATOSA` baseada no [código fonte original](https://github.com/IdentityPython/SATOSA/tree/v8.4.0) com algumas alterações.

Este repositório possui também textos descrevendo algumas características do `SATOSA` e guias com as etapas para implantação contendo exemplos de configurações, conforme índice apresentado a seguir. Todo o conteúdo destes documentos foram baseados na wiki do repositório oficial do `SATOSA` e adaptados para o cenário desta instância.

---

### Índice:

1. [Características e Arquitetura.](./docs/conceitos.md)
2. [Instalação e Configuração inicial.](#configurações-iniciais)

---

## Características

O SATOSA é uma solução de proxy, mantida pela organização Identity Python, para tradução e mapeamento de atributos entre os principais protocolos de autenticação, tais como: SAML2, OpenID Connect e OAuth2. Essas funcionalidades permitem a integração dos serviços com os mecanismos de autenticação sem a necessidade de implementação de todos os protocolos. Mais informações em: [Características e Arquitetura.](./docs/conceitos.md)

## Instalação

As etapas a seguir descrevem a implantação de uma instância do `SATOSA` com ativação dos Plugins de `Backend` e `Frontend` para o protocolo SAML. Os arquivos de configurações presentes no diretório `./volumes` deste repositório apresentam o exemplo de um cenário com a utilização do `SATOSA` como um proxy de SAML para SAML onde a relação de confiança é estabelecida entre um SP e uma Federação SAML por meio do proxy. Também será descrito opções para realizar proxy SAML com Logins Sociais.

### Requisitos
* Servidor com FQDN ou IP válido.
* Chave privada e certificado TLS.
* Docker Engine.
* Docker Compose.

### Configurações Iniciais

1. Baixar o repositório e acessar o diretório raiz.

   ```
   git clone -n --depth=1 --filter=tree:0 https://github.com/luandalmazo/tools.git
   cd tools
   git sparse-checkout set --no-cone satosa-docker
   git checkout
   cd satosa-docker/

   ```

2. Alterar as variáveis nos arquivos `.env` e `.satosa_env` para informações do servidor.
3. Para habilitar o TLS no próprio Gunicorn, presente na imagem docker, inserir arquivos (chave privada e certificado) no diretório `./volumes/`, com os nomes `https.key` e `https.crt`.
4. Alterar as informações do frontend (IdP do Proxy) do arquivo `./volumes/plugins/frontends/saml2_frontend_gidlab.yaml.example` renomeando para `saml2_frontend.yaml`. Atenção principal a propriedade `metadata` dentro da chave `idp_config`. Aqui devem ser informados os metadados dos Provedores de Serviços (SPs) que utilizarão o proxy para se comunicar com a federação.
5. Alterar informações do backend (SP do Proxy) no arquivo `./volumes/plugins/backends/saml2_frontend_gidlab.yaml.example` renomeando para `saml2_frontend.yaml`.
6. Alterar informações de configurações gerais no arquivo `./volumes/proxy_conf.yaml`.

**Obs.:** Os arquivos configuram o proxy entre Federação CAFe Expresso e SPs com o protocolo SAML.

### Iniciar contêiner

Execute o comando `docker compose up` para acompanhar os logs no terminal ou `docker compose up -d` para inciar o compose em modo *deamon*.

## Relação de confiança

Como o Proxy `SATOSA` desta configuração possui 2 plugins SAML habilitados, será possível obter os metadados do frontend e do backend. Se a propriedade `entityid` não foi alterada, os respectivos arquivos podem ser acessados via web em:

1. Metadados:
   * Frontend - `https://FQDN/Saml2IDP/proxy`.
   * Backend - `https://FQDN/Saml2/proxy_saml2_backend`.
   
   **Obs.:** Altere o `FQDN` para o do servidor.

2. Configure o metadado do `Frontend` no(s) SP(s) SAML configurado(s) na etapa 5 das [Configurações Iniciais](#configurações-iniciais).
   
3. Encaminhe o metadado do `Backend` para o operador da federação.
    
    
## Mapeamento de Atributos

Os arquivos atuais possuem as configurações necessárias para o mapeamento de todos os atributos presentes nos diretórios de usuários atualmente utilizados na Federação CAFe Expresso.

Ao todos são quatro arquivos que precisam ser configurados:
1. `volumes/plugins/backends/saml2_backend.yaml`
2. `volumes/plugins/frontends/saml2_frontend.yaml`
3. `volumes/attributemaps/saml_uri.py`
4. `volumes/internal_attributes.yaml`

Se houver necessidade de adicionar novos atributos seguir o procedimento em: [Adicionando novos atributos.](./docs/novosatributos.md)

## Plugins Backend Logins Sociais

Os arquivos de exemplos estão em `./volumes/plugins/backends/`. Por exemplo, `google_backend.yaml.example` deverá ser renomeado para `google_backend.yaml` e preencher os campos `client_id: <client id>` e `client_secret: <client secret>`. Acompanhar os logs do Satosa para mapear os atributos recebidos para os devidos atributos SAML seguindo orientações de [Adicionando novos atributos.](./docs/novosatributos.md) Etapa 3. Arquivo internal_attributes.yaml.

## Comandos úteis 

1. Script para gerar metadado, executar os comandos de forma interativa no contêiner do Satosa.
`docker exec -i satosa /bin/bash`

```bash
# satosa-saml-metadata "${DATA_DIR}/proxy_conf.yaml" "${DATA_DIR}/frontend.key" "${DATA_DIR}/frontend.crt" --dir "${METADATA_DIR}" --split-frontend
```
