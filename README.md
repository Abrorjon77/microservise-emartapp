# Emart Microservices App

A full-stack e-commerce application built with a microservices architecture, containerised with Docker and deployed via a GitOps CI/CD pipeline.

## Architecture

```
                        ┌──────────────┐
                        │  Angular     │  :4200
                        │  Client      │
                        └──────┬───────┘
                               │
                        ┌──────▼───────┐
                        │    Nginx     │  :80
                        │ API Gateway  │
                        └──────┬───────┘
               ┌───────────────┴───────────────┐
        ┌──────▼──────┐                 ┌──────▼──────┐
        │  Node API   │  :5000          │  Java API   │  :9000
        │  (Express)  │                 │  (Spring)   │
        └──────┬──────┘                 └──────┬──────┘
        ┌──────▼──────┐                 ┌──────▼──────┐
        │   MongoDB   │  :27017         │    MySQL    │  :3306
        └─────────────┘                 └─────────────┘
```

| Service | Tech | Responsibility |
|---------|------|----------------|
| `client` | Angular | Frontend UI |
| `api` (nodeapi) | Node.js / Express / MongoDB | Users, products, orders, auth |
| `webapi` (javaapi) | Spring Boot / MySQL | Books catalogue |
| `nginx` | Nginx | API gateway / reverse proxy |
| `emongo` | MongoDB 4 | Document store for Node API |
| `emartdb` | MySQL 8 | Relational store for Java API |

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) & Docker Compose
- Git

## Running Locally

```bash
git clone https://github.com/YOUR_USERNAME/microservise-emartapp.git
cd microservise-emartapp
```

Create the required config files (see [Configuration](#configuration) below), then:

```bash
docker compose up --build
```

The app will be available at `http://localhost`.

## Configuration

Before running, create the following files with your own values:

**`nodeapi/config/keys.js`**
```js
module.exports = {
  mongoURI: "mongodb://emongo:27017/epoc",
  secretOrKey: "YOUR_JWT_SECRET"
};
```

**`javaapi/src/main/resources/application.properties`**
```properties
server.port=9000
spring.datasource.url=jdbc:mysql://emartdb:3306/books?allowPublicKeyRetrieval=true&useSSL=False
spring.datasource.username=root
spring.datasource.password=YOUR_DB_PASSWORD
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.hibernate.ddl-auto=update
```

> These files are excluded from version control — never commit secrets.

## CI/CD Pipeline

Each service has its own GitHub Actions workflow in [.github/workflows/](.github/workflows/):

| Workflow | Trigger | Action |
|----------|---------|--------|
| `client.yml` | Push to `main` (client changes) | Build & push Docker image to DockerHub |
| `nodeapi.yml` | Push to `main` (nodeapi changes) | Build & push Docker image to DockerHub |
| `javaapi.yml` | Push to `main` (javaapi changes) | Build & push Docker image to DockerHub |

After a successful image push, each workflow updates the image tag in a separate GitOps config repo (`emart-k8s-config`) which triggers a Kubernetes deployment.

### Required GitHub Secrets

| Secret | Description |
|--------|-------------|
| `DOCKERHUB_USERNAME` | Your DockerHub username |
| `DOCKERHUB_TOKEN` | DockerHub access token |
| `CONFIG_REPO_TOKEN` | GitHub PAT with write access to the k8s config repo |

## Project Structure

```
.
├── client/          # Angular frontend
├── nodeapi/         # Node.js / Express API
├── javaapi/         # Spring Boot API
├── nginx/           # Nginx gateway config
├── docker-compose.yaml
└── .github/
    └── workflows/   # CI/CD pipelines
```
trigger