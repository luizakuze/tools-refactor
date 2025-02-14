# Guia de Instalação do Keycloak

## Pré-requisitos

Antes de iniciar, certifique-se de ter o **Docker** instalado e configurado corretamente em seu ambiente.  

## Configuração  

Acesse o diretório `keycloak-docker` e configure as seguintes variáveis de ambiente no arquivo `docker-compose.yml`. 

- `KEYCLOAK_ADMIN_PASSWORD` → Defina uma senha segura para o administrador.  
- `KC_HTTPS_CERTIFICATE_FILE` → Caminho para o arquivo do certificado **.cer**.  
- `KC_HTTPS_CERTIFICATE_KEY_FILE` → Caminho para o arquivo da chave **.key**.  

## Inicializando o Keycloak  

Para subir o serviço, execute:  

```sh
docker compose up