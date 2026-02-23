# 10 - Migração de Projetos

## Objetivos

- Migrar projetos do Compose para Swarm
- Identificar configurações incompatíveis
- Adaptar compose files para Swarm
- Deployar projeto migrado em produção

## Teoria

### Compatibilidade Compose → Swarm

O Docker Compose v3+ é compatível com Swarm, mas há diferenças:

### O que NÃO funciona no Swarm

| Comando | Swarm | Solução |
|---------|-------|---------|
| `build:` | ❌ | Build prévio, use imagens prontas |
| `depends_on` | ⚠️ | Use healthchecks |
| `links` | ❌ | Use networks |
| `pid: host` | ❌ | Não suportado |
| `net: host` | ❌ | Use modo host com cuidado |
| `volumes_from` | ❌ | Use volumes nomeados |

### Configurações que Mudam

```yaml
# COMPOSE (docker-compose.yml)
services:
  web:
    build: .
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production
    volumes:
      - ./data:/app/data
    restart: always

# SWARM (docker-compose.yml com deploy)
services:
  web:
    image: myapp:latest   # pré-build!
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production
    volumes:
      - mydata:/app/data
    deploy:              # NOVO!
      replicas: 2
      restart_policy:
        condition: on-failure
      resources:
        limits:
          cpus: "0.5"
          memory: 512M

volumes:
  mydata:
```

### Estratégia de Migração

1. **Build local** → Build em CI/CD, push para registry
2. **Ambiente local** → Variáveis via Secrets/Configs
3. **Volumes** → Named volumes + backups
4. **Ports** → Verificar conflitos em produção

### Checklist de Produção

- [ ] Imagens em registry privado
- [ ] Secrets configurados
- [ ] Recursos (CPU/memória) definidos
- [ ] Constraints para DB (manager)
- [ ] Healthchecks implementados
- [ ] Políticas de restart
- [ ] Networks overlay
- [ ] Volumes com backup
- [ ] Logging configurado

## Prática

### Passo 1: Preparar Imagens

```bash
# Antes (compose)
docker-compose build

# Depois (Swarm)
# Build em CI/CD
docker build -t myregistry.io/myapp:v1 .
docker push myregistry.io/myapp:v1
```

### Passo 2: Adaptar Compose File

```yaml
# docker-compose.swarm.yml
version: "3.8"

services:
  app:
    image: myregistry.io/myapp:${VERSION:-latest}
    environment:
      - DATABASE_URL=${DATABASE_URL}
    secrets:
      - db_password
    networks:
      - app-network
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          cpus: "1"
          memory: 1G
        reservations:
          cpus: "0.5"
          memory: 512M

  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - app-network
    secrets:
      - db_password
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
      resources:
        limits:
          cpus: "2"
          memory: 2G

volumes:
  pgdata:

networks:
  app-network:
    driver: overlay

secrets:
  db_password:
    external: true
```

### Passo 3: Criar Secrets

```bash
# Criar secrets baseados em .env
docker secret create db_password .env.db_password
docker secret create api_key .env.api_key
```

### Passo 4: Deploy

```bash
# Deploy
export VERSION=v1.0.0
docker stack deploy -c docker-compose.swarm.yml minhaapp

# Verificar
docker stack services minhaapp

# Escalar
docker service scale minhaapp_app=5
```

### Comandos de Migração

```bash
# Converter compose para versão válida no Swarm
docker-compose version
docker-compose convert

# Validar compose file
docker-compose config -q

# Verificar se tudo é compatível
docker-compose config
```

## Exercícios

1. **Exercício 1**: Pegar um compose file simples e converter para Swarm
2. **Exercício 2**: Migrar um projeto com build para usar imagem pré-construída
3. **Exercício 3**: Criar stack completa com app + db + redis + nginx
4. **Exercício 4**: Criar CI/CD básica que build e deploy no Swarm

## Referências

### Documentação Oficial
- [Compose file reference](https://docs.docker.com/compose/compose-file/)
- [Deploy to Swarm](https://docs.docker.com/engine/swarm/stacks/)
- [Best practices](https://docs.docker.com/engine/swarm/best-practices/)

### Comandos CLI
- [docker stack deploy]()
- [docker secret create]()
