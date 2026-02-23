# 03 - Services

## Objetivos

- Criar e gerenciar services no Docker Swarm
- Compreender a diferença entre services, tasks e containers
- Configurar réplicas e políticas de update
- Entender o ciclo de vida de uma task

## Teoria

### O que é um Service?

Um **Service** é a definição de como você quer executar uma aplicação no Swarm. É similar ao conceito de service no Docker Compose, mas com superpoderes de orquestração.

```yaml
# Service vs Compose
# Compose: você define "replicas" manualmente (se supported)
# Swarm: replicas é built-in, gerenciado automaticamente
```

### Service vs Task vs Container

```
┌─────────────────────────────────────────────────┐
│                    SERVICE                      │
│   "Quero 3 réplicas do nginx na porta 80"      │
└─────────────────────┬───────────────────────────┘
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
    ┌────────┐    ┌────────┐    ┌────────┐
    │ TASK 1 │    │ TASK 2 │    │ TASK 3 │
    │(nginx) │    │(nginx) │    │(nginx) │
    └────────┘    └────────┘    └────────┘
        │             │             │
        ▼             ▼             ▼
    ┌────────┐    ┌────────┐    ┌────────┐
    │CONTAINER│   │CONTAINER│   │CONTAINER│
    └────────┘    └────────┘    └────────┘
```

### Tipos de Service

1. **Replicated**: Número específico de réplicas
   - Distribuídas entre os nós
   - Load balancing automático

2. **Global**: Uma task em cada nó
   - Ideal para: monitoring agents, logging, security
   - Útil para serviços que precisam estar em todos os nós

### Ciclo de Vida de uma Task

```
NEW → ASSIGNED → PREPARING → RUNNING → COMPLETE
                ↓
              FAILED
```

- **NEW**: Task criada
- **ASSIGNED**: Atribuída a um nó
- **PREPARING**: Preparando ambiente (download image, etc)
- **RUNNING**: Container em execução
- **COMPLETE**: Task finalizada com sucesso
- **FAILED**: Task falhou

### update & rollback policies

- **update-delay**: Tempo entre updates de cada replicas
- **update-parallelism**: Quantas réplicas atualizar simultaneamente
- **update-failure-action**: O que fazer se falhar (continue, pause, rollback)
- **rollback-delay**: Tempo entre rollbacks
- **rollback-parallelism**: Quantas réplicas fazer rollback simultaneamente

## Prática

### Criando Services

```bash
# Service básico
docker service create --name web --replicas 3 -p 80:80 nginx:alpine

# Com recursos (CPU/memória)
docker service create \
  --name api \
  --replicas 2 \
  --limit-cpu 0.5 \
  --limit-memory 512M \
  -e NODE_ENV=production \
  nginx:alpine

# Com constraints (veremos mais em aula 08)
docker service create \
  --name db \
  --replicas 1 \
  --constraint "node.role==manager" \
  postgres:15
```

### Gerenciando Services

```bash
# Listar services
docker service ls

# Ver tasks de um service
docker service ps web

# Inspect service (JSON completo)
docker service inspect web

# Ver apenas nome e imagem
docker service inspect --pretty web

# Logs do service (importante: aggregate de todas as tasks)
docker service logs -f web

# Remover service
docker service rm web

# Escalar (maneira antiga - ainda funciona)
docker service scale web=5

# Atualizar image
docker service update --image nginx:latest web
```

### Replicated vs Global

```bash
# Replicated (padrão)
docker service create --name web --replicas 3 nginx

# Global (uma task por nó)
docker service create --mode global --name monitoring-agent prometheus/prometheus
```

### Atualizando Services

```bash
# Atualizar réplicas
docker service update --replicas 5 web

# Atualizar variáveis de ambiente
docker service update --env-add NODE_ENV=production web

# Atualizar portas
docker service update --publish-add 8080:80 web

# Rollback para versão anterior
docker service rollback web

# Atualizar com política customizada
docker service update \
  --image nginx:1.25 \
  --update-delay 10s \
  --update-parallelism 1 \
  --update-failure-action pause \
  web
```

## Exercícios

1. **Exercício 1**: Criar um service com 3 réplicas e observar a distribuição das tasks
2. **Exercício 2**: Forçar uma task a falhar (docker service update com --force) e observar o comportamento
3. **Exercício 3**: Criar um global service e verificar que há uma task em cada nó
4. **Exercício 4**: Fazer um update do service (mudar imagem) e observar o rolling update

## Referências

- [How services work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/services/)
- [Create services](https://docs.docker.com/engine/swarm/services/)
- [Service configuration](https://docs.docker.com/engine/reference/commandline/service_create/)
