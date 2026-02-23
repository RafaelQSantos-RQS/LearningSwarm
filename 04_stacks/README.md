# 04 - Stacks

## Objetivos

- Compreender o que é uma Stack e quando usá-la
- Deployar aplicações multi-service usando Stack
- Entender as diferenças entre docker-compose e docker-compose.yml no Swarm
- Gerenciar ciclo de vida de stacks

## Teoria

### O que é uma Stack?

Uma **Stack** é um grupo de serviços inter-relacionados que podem ser deployados e gerenciados juntos. É o equivalente a "docker-compose up" no mundo Swarm.

### Compose vs Stack

| Aspecto | docker-compose | docker stack |
|---------|----------------|--------------|
| Escopo | Host único | Multi-host |
| Orchestration | Limitada | Completa |
| Scaling | Manual | Automático |
| Command | `docker-compose up` | `docker stack deploy` |
| Definição | `docker-compose.yml` | `docker-compose.yml` (com nuances) |

### Quando usar Stacks?

- Aplicações com múltiplos serviços (frontend, backend, db, cache)
- Necessidade de deploy atômico de toda aplicação
- Orquestração complexa entre serviços

### key Differences no docker-compose.yml

```yaml
version: "3.8"  # Use 3.x para Swarm

services:
  web:
    image: nginx
    ports:
      - "80:80"
    # SWARM: usa 'deploy' para configurações de orquestração
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: "0.5"
          memory: 256M
        reservations:
          cpus: "0.25"
          memory: 128M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
      rollback_config:
        parallelism: 1
        delay: 10s
      placement:
        constraints:
          - node.role == worker

  api:
    image: myapi:latest
    environment:
      - DATABASE_URL=postgres://db:5432
    depends_on:
      - db
    deploy:
      replicas: 3

  db:
    image: postgres:15
    volumes:
      - db-data:/var/lib/postgresql/data
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager

volumes:
  db-data:

networks:
  frontend:
  backend:
```

### Configurações Ignoradas pelo Swarm

Algumas configurações do compose **não funcionam** no Swarm:
- `build:` - imagens devem estar pré-construídas
- `depends_on` - não garante ordem de start (use healthchecks)
- `links` - use networks inverse
- `pid` - não suportado
- `net: host` - não suportado

## Prática

### Deploy de uma Stack

> **Quando usar `docker stack deploy`**: Para fazer deploy de toda uma aplicação de uma vez

```bash
# Criar arquivo docker-compose.yml primeiro (exemplo em codigo/)

# Deploy da stack
docker stack deploy -c docker-compose.yml mystack

# Listar stacks
docker stack ls

# Listar serviços da stack
docker stack services mystack

# Listar tasks (como service ps)
docker stack ps mystack

# Ver logs agregados da stack
docker stack logs mystack

# Remover stack (remove todos os serviços)
docker stack rm mystack
```

### Comandos Úteis

```bash
# Verificar se stack está toda UP
docker stack services mystack | grep -v Running

# Inspect de um service específico da stack
docker service inspect mystack_web

# Visualizar formato (o que será deployado)
docker stack config -c docker-compose.yml

# Atualizar stack (redeploy)
docker stack deploy -c docker-compose.yml mystack
```

## Exercícios

1. **Exercício 1**: Criar uma stack com 2 serviços (nginx + redis) e fazer deploy
2. **Exercício 2**: Adicionar volumes e networks ao compose e testar
3. **Exercício 3**: Fazer deploy, editar algo no compose, e usar --resolve-image changed
4. **Exercício 4**: Criar uma stack com pelo menos 3 serviços (ex: app + db + cache)

## Referências

### Documentação Oficial
- [stack-deploy](../00_docs/guia/stack-deploy.md) - Deploy de stacks

### Comandos CLI
- [docker stack deploy]()
- [docker stack ls]()
- [docker stack services]()
- [docker stack ps]()
