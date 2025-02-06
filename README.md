# Proxies em Ambientes de Gerenciamento de Identidades: Estudo de Casos e Avaliação Prática

Este repositório contém as documentações dos casos de usos apresentados no artigo.

O repositório contém os seguintes arquivos e diretórios:

* **Instalação Shibboleth 5:** Guia de instalação para um Shibboleth 5. Este deve ser o primeiro guia a ser consultado.

* **Configuração do Azure com Shibboleth 5:** Guia geral de configuração do Azure considerando instalação do Shibboleth 5.
* **Guia de Instalação e Configuração do SimpleSAMLphp como proxy SAML:** Guia de instalação para o SimpleSAMLphp (SSP).  Para seguir este guia, considere o diretório abaixo:
  * **simplesamlphp-config**: Contém um exemplo de Docker para a aplicação SimpleSAMLphp. Para o funcionamento adequado, os certificados do Apache devem estar dentro de `simplesamlphp-config/certs-apache`. Além disso, para executar o Docker, é necessário instalar o SSP conforme descrito no guia acima.



## Instruções gerais

Cada guia deste repositório contém exemplos de execução e as dependências necessárias para o funcionamento das ferramentas.