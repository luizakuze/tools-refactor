# Adicionando Novos Atributos no SATOSA Proxy

## ↔️ Mapeamento de Atributos

Os arquivos atuais possuem as configurações necessárias para o mapeamento de todos os atributos presentes nos diretórios de usuários atualmente utilizados na Federação CAFe Expresso.

Ao todos são quatro arquivos que precisam ser configurados:
1. `volumes/plugins/backends/saml2_backend.yaml`
2. `volumes/plugins/frontends/saml2_frontend.yaml`
3. `volumes/attributemaps/saml_uri.py`
4. `volumes/internal_attributes.yaml`


### 📌 1. Arquivo saml2_backend.yaml
No arquivo `saml2_backend.yaml` precisará adicionar o parâmetro `attribute_map_dir: attributemaps` identado como uma das configurações do `sp_config:`:

```
  ...
  sp_config:
    ...
    attribute_map_dir: attributemaps
    #allow_unknown_attributes: yes
    ...

```

A opção `allow_unknown_attributes: yes` poderá ser utilizada para testes e avaliação de quais atributos estão disponíveis no IdP de Origem / Federado.


### 📌 2. Arquivo saml2_frontend.yaml
No arquivo `saml2_frontend.yaml` precisará adicionar o parâmetro dois novos parâmetros.
O primeiro parâmetro é o `attribute_profile: saml` identado como uma das configurações do `config:`.

```
name: Saml2IDP
config:
  attribute_profile: saml

```

O segundo é o `attribute_map_dir: attributemaps` identado como uma das configurações do `idp_config:`:

```
  ...
  idp_config:
    ...
    attribute_map_dir: attributemaps
    ...

```

A opção `allow_unknown_attributes: yes` poderá ser utilizada para testes e avaliação de quais atributos estão disponíveis no IdP de Origem / Federado.


### 📌 3. Arquivo saml_uri.py
No arquivo `saml_uri.py` precisa reconhecer o  o mapeamento do Uniform Resource Names (URN) e seu Object Identifiers (OIDs). Para isso precisará adicionar o **urn:oid:** pai do atributo que deseja mapear. Por exemplo, o **brPersonCPF** tem o seguinte urn: **urn:oid:1.3.6.1.4.1.15996.100.1.1.1.1**. E para mapear ele nesse arquivo deverá selecionar sua parte inicial em uma variável, por exemplo:
`BREDUPERSON_OID_1 = 'urn:oid:1.3.6.1.4.1.15996.100.1.1.1.' # from rnp brPersonCPF`

Seguindo a mesma lógica do mapeamento do arquivo, vamos adicionar o identificador dele, o `1` no grupo `'fro': {` e terá o arquivo da seguinte forma:

```
...
MAP = {
    'identifier': 'urn:oasis:names:tc:SAML:2.0:attrname-format:uri',
    'fro': {
        ...
        BREDUPERSON_OID_1+'1': 'brPersonCPF',
        ...
```
Seguindo essa mesma lógica adicionando o encaminhamento desse atributo no grupo `'to': {` para ficar da seguinte forma:

````
...
   'to': {
        ...
        'brPersonCPF': BREDUPERSON_OID_1+'1',
        ...
````

Esse procedimento deverá ser realizado para cada novo **urn:oid** pai que os atributos que deseja mapear tenham. Para verificar mais informações sobre os atributos a serem mapeados poderá usar o link do **OID Repository**. Ex.:  http://www.oid-info.com/get/1.3.6.1.4.1.1466.115.121.1

### 📌 4. Arquivo internal_attributes.yaml

O arquivo `internal_attributes.yaml` faz mapeamento a partir de grupos que podem ter origens diferentes dos **urn:oid**, por exemplo, facebook, linkedin, orcid, github, openid e saml. 
Como estamos mapeando atributos **saml**, vamos usar esse grupo identificador para mapear os novos atributos. Adicionando ao arquivo, por exemplo, as seguintes linhas:

```
attributes:
  ...
  brPersonCPF:
    saml: [brPersonCPF, brpersoncpf]
  ...

```