# Guess Game - Docker Compose

Este repositório implementa o aplicativo guess_game usando Docker Compose.
Baseado no repositório disponibilizado pelo professor Fernando Augusto para o curso de Pós-Graduação de Engenharia de IA e MLOps da PUC Minas em:
https://github.com/fams/guess_game


Inclui:

- backend Python/Flask
- banco de dados PostgreSQL com volume persistente do Docker
- frontend React servido pelo NGINX
- proxy reverso NGINX com balanceamento de carga entre instâncias de backend

## Arquitetura e design

### Serviços

- postgres
  - usa a imagem oficial postgres:16-alpine
  - armazena os dados em um volume persistente para manter os dados do jogo entre reinícios

- backend1 e backend2
  - executam o mesmo aplicativo Flask em containers separados
  - fornecem redundância e permitem balanceamento de carga
  - usam restart: always para reiniciar em caso de falha

- nginx
  - serve os arquivos estáticos do frontend React
  - encaminha requisições /api/ para o upstream de backend
  - usa um upstream chamado backend com backend1:5000 e backend2:5000

### Volumes e redes

- postgres_data
  - volume Docker dedicado usado pelo serviço postgres
  - garante que os dados do jogo persistam mesmo se o container for recriado

- guess-network
  - rede bridge Docker compartilhada por todos os serviços
  - isola a comunicação entre nginx, backend1, backend2 e postgres

### Estratégia de balanceamento de carga

- O NGINX define um upstream backend com duas instâncias Flask.
- Requisições para /api/ são encaminhadas para esse upstream.
- O upstream distribui as requisições entre backend1 e backend2.
- Isso melhora a resiliência e a capacidade para requisições simultâneas.

## Arquivos importantes

- docker-compose.yml - orquestra serviços, redes e volumes
- Dockerfile.backend - constrói o container do backend Flask
- frontend/Dockerfile - constrói o frontend React e a imagem NGINX
- frontend/default.conf - configura o proxy reverso e o balanceamento de carga
- .dockerignore e frontend/.dockerignore - excluem artefatos de build dos contextos Docker

## Instalar e executar

### Pré-requisitos

- Docker instalado
- Docker Compose disponível

### Executar a aplicação

A partir do diretório do projeto:

```powershell
cd c:\git_puc\guess_game
docker compose up --build --detach
```

### URL de acesso

Após iniciar o Docker Compose, abra:

```text
http://localhost:8080
```

### Verificação de saúde

O endpoint de health está disponível em:

```text
http://localhost:8080/api/health
```

Resposta esperada:

```json
{"status": "ok"}
```

## Atualizar serviços

### Atualizar backend

- Modifique Dockerfile.backend ou use uma nova versão da imagem do backend.
- Reconstrua e inicie o stack novamente:

```powershell
docker compose up --build --detach
```

### Atualizar frontend

- Altere frontend/Dockerfile ou os arquivos de origem do frontend.
- Reconstrua com:

```powershell
docker compose up --build --detach
```

### Atualizar banco de dados

- Altere a tag da imagem Postgres em docker-compose.yml.
- Reinicie o stack com:

```powershell
docker compose up --build --detach
```

## Decisões de design

- postgres:16-alpine foi escolhido por ser uma imagem leve e compatível.
- Dois containers de backend permitem balanceamento de carga e maior tolerância a falhas.
- O NGINX serve o frontend e encaminha apenas solicitações de API, mantendo o frontend independente.
- Um volume persistente dedicado mantém os dados do jogo seguros durante eventos de ciclo de vida do container.
- restart: always aumenta a resiliência dos serviços.

## Observações

- O código da aplicação não foi modificado além da integração com Docker.
- Arquivos criados ou modificados no projeto, para atender o desafio:
  - docker-compose.yml
  - Dockerfile.backend
  - frontend/Dockerfile
  - frontend/default.conf
  - .dockerignore
  - frontend/.dockerignore

## Resultado verificado

- O stack inicia corretamente.
- O frontend é servido em http://localhost:8080.
- http://localhost:8080/api/health retorna {"status": "ok"}.
- Os serviços backend1, backend2, nginx e postgres permanecem em execução.
