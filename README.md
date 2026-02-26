# 🌐 Hackathon — Infraestrutura e Orquestração

Este repositório contém a **infraestrutura e orquestração completa**, utilizando **Docker Compose** para integrar os microserviços de Autenticação, Vídeo e o Worker de Processamento, juntamente com serviços de infraestrutura como PostgreSQL, RabbitMQ e Redis. O **Kong API Gateway** é utilizado para roteamento e segurança, protegendo as APIs com autenticação JWT.

---

## 🧾 Objetivo do Projeto

Orquestrar o ecossistema completo da aplicação, provendo um ambiente de desenvolvimento e testes consistente e fácil de configurar. O objetivo é integrar todos os componentes da Fase 5 do Hackathon, garantindo a comunicação entre os serviços, a persistência de dados, a mensageria assíncrona e a segurança através de um API Gateway.

> 📚 **Wiki do Projeto:** <br/> > https://github.com/FIAP-SOAT-G129/.github/wiki/Fase-5

---

## 🚀 Tecnologias Utilizadas

- **Docker** (Containerização)
- **Docker Compose** (Orquestração de containers)
- **Kong API Gateway** (Roteamento, gerenciamento de APIs e segurança)
- **PostgreSQL** (Bancos de dados para Auth MS e Video MS)
- **RabbitMQ** (Mensageria assíncrona para Video MS e Worker)
- **Redis** (Cache para Video MS)
- **Java 21 & Spring Boot 3** (Microserviços)

---

## 🧠 Arquitetura Geral

A arquitetura do sistema é baseada em microserviços, orquestrados pelo Docker Compose e expostos através do Kong API Gateway. O fluxo de requisições e processamento é o seguinte:

1.  **Kong API Gateway**: Ponto de entrada para todas as requisições externas, responsável por roteamento, balanceamento de carga, autenticação e outras políticas de API.
2.  **Auth MS**: Microserviço de autenticação e autorização, gerencia usuários e emite tokens JWT.
3.  **Video MS**: Microserviço de gerenciamento de vídeos, interage com PostgreSQL para metadados, Redis para cache e RabbitMQ para mensageria assíncrona com o Worker.
4.  **Worker**: Consome mensagens do RabbitMQ, processa vídeos (extração de frames, compactação) e notifica o Video MS sobre o status do processamento.
5.  **PostgreSQL**: Bancos de dados dedicados para Auth MS e Video MS.
6.  **RabbitMQ**: Broker de mensagens para comunicação assíncrona entre Video MS e Worker.
7.  **Redis**: Utilizado pelo Video MS para cache de informações.

---

## 🚦 Mapeamento de Portas

Para acessar os serviços expostos, utilize as seguintes portas no seu `localhost`:

| Serviço                 | Porta Host | Descrição                                      |
|:------------------------|:-----------|:-----------------------------------------------|
| **Kong Proxy**          | `8000`     | **Ponto de entrada do API Gateway**            |
| **Kong Admin**          | `8001`     | API de administração do Kong                   |
| **RabbitMQ Management** | `15672`    | Interface de gerenciamento do RabbitMQ         |

---

## 🛣 Rotas do API Gateway (Kong)

O Kong API Gateway roteia as requisições para os microserviços internos e aplica políticas de segurança:

| Caminho   | Serviço de Destino | Proteção JWT | Descrição                                       |
|:----------|:-------------------|:-------------|:------------------------------------------------|
| `/auth`   | `auth-ms`          | Não          | Rotas de autenticação (registro, login)         |
| `/videos` | `video-ms`         | Sim          | Rotas de gerenciamento de vídeos (requer token) |

---

## 🔐 Fluxo de Segurança com Kong e JWT

Todas as rotas do `video-ms` (`/videos`) são protegidas pelo plugin JWT do Kong. Para acessar essas rotas, o cliente deve:

1.  **Autenticar-se no Auth MS**: Enviar credenciais para `http://localhost:8000/auth/login` para obter um token JWT.
2.  **Incluir o JWT nas Requisições**: O token JWT deve ser enviado no cabeçalho `Authorization` como `Bearer <seu_token>` para as rotas do `video-ms` (ex: `http://localhost:8000/videos/...`).

O Kong interceptará a requisição, validará o JWT usando o segredo configurado (`qAkwER/knmSv1FZ0qzH+E9EEj5YsLn5zm9M/fY8RH9c=`) e, se válido, encaminhará a requisição para o `video-ms`.

---

## ⚙️ Como Rodar o Projeto

### ✅ Pré-requisitos
- `Docker`
- `Docker Compose`

### 🔧 Configuração

As configurações dos serviços são definidas no `docker-compose.yml` e no `kong/kong.yml`. As variáveis de ambiente para os microserviços são passadas diretamente no `docker-compose.yml`.

Caso deseje alterar, as principais variáveis de ambiente são:

```env
VIDEO_DB_NAME=video_db
VIDEO_DB_USER=user
VIDEO_DB_PASSWORD=my_password

NOTIFICATION_MAIL_HOST=smtp.example.com
NOTIFICATION_MAIL_PORT=587
NOTIFICATION_MAIL_USERNAME=guest
NOTIFICATION_MAIL_PASSWORD=guest

AUTH_DB_NAME=auth_db
AUTH_DB_USER=user
AUTH_DB_PASSWORD=my_password

JWT_SECRET=qAkwER/knmSv1FZ0qzH+E9EEj5YsLn5zm9M/fY8RH9c=
JWT_EXPIRATION=360000
JWT_ISSUER=hackathon-issuer
```

### 🐳 Executando o ecossistema completo

No terminal, navegue até a raiz deste repositório (`hackathon-infra-fase5`) e execute:

```bash
docker compose up --build -d
```

Este comando irá:
- Construir as imagens dos microserviços (se necessário) ou puxá-las do DockerHub.
- Iniciar todos os serviços (PostgreSQL, RabbitMQ, Redis, Auth MS, Video MS, Worker, Kong) na ordem correta, aguardando a saúde de cada dependência.

#### ⏹️ Parando os containers

Para parar e remover todos os containers e redes criadas pelo Docker Compose, execute:

```bash
docker compose down
```

---

## 👥 Equipe

Desenvolvido pela equipe **FIAP SOAT - G129** como parte do projeto de Arquitetura de Software.

---

## 📄 Licença

Este projeto é parte de um trabalho acadêmico da FIAP.
