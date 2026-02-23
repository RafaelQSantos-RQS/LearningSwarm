# 08 - Constraints e Labels

## Objetivos

- Controlar onde as tasks são executadas usando constraints
- Usar labels em nós para organização
- Entender placement de serviços
- Implementar estratégias de affinity/anti-affinity

## Teoria

### Constraints

Constraints permitem **restringir** em quais nós um serviço pode rodar:

```bash
# Sintaxe
--constraint "node.attribute==value"   # igual
--constraint "node.attribute!=value"   # diferente
```

### Atributos de Nós

| Atributo | Descrição | Exemplo |
|----------|-----------|---------|
| `node.id` | ID único do nó | node.id==abc123 |
| `node.hostname` | Hostname da máquina | node.hostname==server1 |
| `node.role` | manager ou worker | node.role==manager |
| `node.labels.*` | Labels personalizadas | node.labels.zone==prod |
| `engine.labels.*` | Labels do Docker | engine.labels.storage==ssd |

### Labels em Nós

Você pode adicionar labels a nós para organização:

```bash
# Adicionar label a um nó
docker node update --label-add zone=prod node-1
docker node update --label-add ssd=true node-1
docker node update --label-add region=us-east node-1

# Remover label
docker node update --label-rm zone node-1
```

### Placement Strategies

```yaml
# spread: distribui igualmente (default)
# binpack: concentra em menos nós
# random: aleatório

deploy:
  placement:
    preferences:
      - spread: node.labels.zone
```

### Replicas vs Placement

| Cenário | Solução |
|---------|---------|
| DB só em nodes de storage | constraint: node.labels.storage==ssd |
| API em workers | constraint: node.role==worker |
|Cache em nós específicos | constraint: node.labels.cache==true |
| Monitoring em todos | service mode: global |

## Prática

### Constraints Básicos

```bash
# Criar service só em managers
docker service create \
  --name db \
  --constraint "node.role==manager" \
  postgres:15

# Criar service só em workers
docker service create \
  --name web \
  --constraint "node.role==worker" \
  nginx

# Evitar nó específico
docker service create \
  --name api \
  --constraint "node.hostname!=node-3" \
  myapi
```

### Usando Labels

```bash
# Adicionar labels aos nós
docker node update --label-add zone=us-east-1 node-1
docker node update --label-add zone=us-west-2 node-2
docker node update --label-add zone=europe-1 node-3

# Criar service com constraint usando label
docker service create \
  --name api \
  --constraint "node.labels.zone==us-east-1" \
  myapi

# Multiple constraints (AND)
docker service create \
  --name api \
  --constraint "node.role==worker" \
  --constraint "node.labels.zone==prod" \
  --constraint "node.labels.ssd==true" \
  myapi
```

### Placement em Stack

```yaml
version: "3.8"

services:
  db:
    image: postgres:15
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager
          - node.labels.storage == ssd

  redis:
    image: redis:alpine
    deploy:
      replicas: 2
      placement:
        constraints:
          - node.labels.cache == "true"

  web:
    image: nginx
    deploy:
      replicas: 3
      placement:
        preferences:
          - spread: node.labels.zone
```

### Global Services

```bash
# Service que roda em TODOS os nós
docker service create \
  --mode global \
  --name monitoring \
  prom/node-exporter

# Com constraints (global + constraint)
docker service create \
  --mode global \
  --constraint "node.role==worker" \
  --name agent \
  myagent
```

### Preferences (Spread)

```yaml
# Preferir nós em diferentes zonas
deploy:
  placement:
    preferences:
      - spread: node.labels.zone

# Multiple preferences
deploy:
  placement:
    preferences:
      - spread: node.labels.az
      - spread: node.labels.rack
```

## Exercícios

1. **Exercício 1**: Adicionar labels a seus nós e criar services que usem essas labels
2. **Exercício 2**: Criar um service que rode apenas em workers
3. **Exercício 3**: Criar um global service para monitoring em todos os nós
4. **Exercício 4**: Criar stack com placement distribuído entre zonas

## Referências

- [Placement constraints](https://docs.docker.com/engine/swarm/services/#placement-constraints)
- [Docker service create](https://docs.docker.com/engine/reference/commandline/service_create/)
- [Placement preferences](https://docs.docker.com/compose/compose-file/compose-file-v3/#placement)
