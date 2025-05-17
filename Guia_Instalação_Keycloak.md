# Guia de Instalação do Keycloak
Esse arquivo apresenta o passo a passo para instalação e execução do Keycloak.

## 📋 Pré-requisito

- [ ] Docker

## ⚙️ Configuração  

Acesse o diretório `tools/keycloak-docker` e configure as seguintes variáveis de ambiente no arquivo `docker-compose.yml`: 

- `KEYCLOAK_ADMIN_PASSWORD` → Defina uma senha segura para o administrador.  
- `KC_HTTPS_CERTIFICATE_FILE` → Caminho para o arquivo do certificado **.cer**.  
- `KC_HTTPS_CERTIFICATE_KEY_FILE` → Caminho para o arquivo da chave **.key**.  

## ▶️ Executar o Keycloak  

Para iniciar o serviço, execute o comando abaixo: ```docker compose up```