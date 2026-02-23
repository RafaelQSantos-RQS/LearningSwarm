# 07 - Updates e Rollbacks

## Objetivos

- Realizar rolling updates em serviços
- Configurar políticas de update
- Executar rollbacks quando necessário
- Implementar healthchecks para melhor controle

## Teoria

### Rolling Updates

Rolling updates permitem atualizar serviços **sem downtime**:
- Atualiza uma ou mais réplicas por vez
- Mantém serviço disponível durante update
- Pode fazer rollback automático em caso de falha

### Parâmetros de Update

| Parâmetro | Descrição | Default |
|-----------|-----------|---------|
| `--update-delay` | Tempo entre updates de cada task | 0s |
| `--update-parallelism` | Tasks atualizadas simultaneamente | 1 |
| `--update-failure-action` | Ação se update falhar | pause |
| `--update-max-failure-ratio` | % de falhas aceitas antes de parar | 0 |
| `--update-monitor` | Tempo para verificar se task está running | 0s |

### Rollback

```bash
# Rollback automático (se configurado)
--rollback-failure-action: rollback

# Rollback manual
docker service rollback myservice
```

### Parâmetros de Rollback

| Parâmetro | Descrição | Default |
|-----------|-----------|---------|
| `--rollback-delay` | Tempo entre rollbacks | 0s |
| `--rollback-parallelism` | Tasks roll-backed simultaneamente | 1 |
| `--rollback-failure-action` | Ação se rollback falhar | pause |
| `--rollback-max-failure-ratio` | % de falhas aceitas | 0 |

### Healthchecks

Importante para updates seguros:

```yaml
services:
  api:
    image: myapi
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

## Prática

### Atualizando Services

```bash
# Atualizar imagem
docker service update --image nginx:1.25 myservice

# Atualizar variáveis de ambiente
docker service update --env-add NODE_ENV=production myservice

# Atualizar réplicas
docker service update --replicas 5 myservice

# Atualizar recursos
docker service update --limit-memory 512M myservice

# Atualizar porta
docker service update --publish-rm 80 --publish-add 8080:80 myservice
```

### Atualização com Política Customizada

```bash
# Atualizar com delay de 10s entre cada task
docker service update \
  --image myapp:v2 \
  --update-delay 10s \
  --update-parallelism 1 \
  --update-failure-action pause \
  myservice

# Verificar status
docker service ps myservice
docker service inspect --pretty myservice
```

### Rollback

```bash
# Rollback para versão anterior
docker service rollback myservice

# Rollback com parâmetros específicos
docker service rollback \
  --rollback-delay 5s \
  --rollback-parallelism 1 \
  myservice

# Ver histórico de rollbacks
docker service inspect --pretty myservice | grep -A 20 "Update status"
```

### Healthchecks

```bash
# Service com healthcheck
docker service create \
  --name api \
  --health-cmd "curl -f http://localhost:3000/health" \
  --health-interval 30s \
  --health-timeout 10s \
  --health-retries 3 \
  --health-start-period 40s \
  myapi:v1

# Verificar saúde
docker service inspect --pretty api

# Atualizar com update_config usando health
docker service update \
  --update-delay 10s \
  --update-failure-action rollback \
  --update-monitor 15s \
  --update-order start-first \
  api
```

### Atualização em Stack

```yaml
version: "3.8"

services:
  api:
    image: myapi:v2
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
        monitor: 15s
        order: start-first
      rollback_config:
        parallelism: 1
        delay: 5s
        failure_action: pause
        order: stop-first
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
```

## Exercícios

1. **Exercício 1**: Criar um service, fazer update de imagem e observar o rolling update
2. **Exercício 2**: Configurar um update com delay e parallelism > 1
3. **Exercício 3**: Simular falha no update e ver o comportamento com --update-failure-action
4. **Exercício 4**: Criar service com healthcheck e verificar o comportamento durante update

## Referências

- [Update a service](https://docs.docker.com/engine/swarm/services/#update-a-service)
- [Rolling updates](https://docs.docker.com/engine/swarm/swarm-tutorial/rolling-update/)
- [Healthchecks](https://docs.docker.com/engine/swarm/configs/)
