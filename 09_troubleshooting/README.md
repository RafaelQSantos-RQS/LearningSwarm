# 09 - Troubleshooting

## Objetivos

- Diagnosticar problemas em serviços Swarm
- Usar logs efetivamente
- Inspect e debug de services e tasks
- Entender comandos de debug

## Teoria

### Hierarquia de Debug

```
Stack
  └── Service
        └── Task (container)
```

### Problemas Comuns

1. **Service não inicia**: Ver image, resources, constraints
2. **Task falhando**: Ver logs, inspect da task
3. **Rede não funciona**: Ver networks, DNS resolution
4. **Service não escala**: Ver resources disponíveis
5. **Manager indisponível**: Ver quorum

### Estados de Tasks

```
NEW → ASSIGNED → PREPARING → RUNNING → COMPLETE
                ↓
              FAILED
```

## Prática

### Verificando Status

> **Quando usar `docker service ls`**: Para ver todos os serviços e status

> **Quando usar `docker service ps`**: Para ver as tasks de um serviço

```bash
# Ver todos os serviços e status
docker service ls

# Ver tasks de um serviço (mais detalhado)
docker service ps myservice

# Ver apenas tasks com problemas
docker service ps myservice | grep -E "Rejected|New|Pending"
```

### Inspect

> **Quando usar `docker service inspect`**: Para debug detalhado

```bash
# Inspect service (resumido)
docker service inspect --pretty myservice

# Inspect service (JSON completo)
docker service inspect myservice

# Inspect nó
docker node inspect self

# Inspect container específico
docker container inspect <container-id>
```

### Logs

> **Quando usar `docker service logs`**: Para ver logs agregados de todas as réplicas

```bash
# Logs do service (todas as tasks)
docker service logs myservice

# Logs com timestamp
docker service logs -t myservice

# Logs de task específica
docker service logs myservice --task <task-id>

# Follow logs em tempo real
docker service logs -f myservice

# Ver últimos 50 linhas
docker service logs --tail 50 myservice
```

### Debug de Problemas

```bash
# Ver por que task está rejeitada
docker service ps myservice
docker node inspect <node-id>

# Ver logs de task específica
docker logs <task-id>

# Entrar no container (se estiver rodando)
docker exec -it <container-id> /bin/sh

# Ver eventos do Docker
docker events

# Ver consumo de recursos
docker stats $(docker ps --format {{.Names}})
```

### Rede

```bash
# Listar networks
docker network ls

# Inspect network
docker network inspect ingress

# Testar conectividade entre serviços
docker exec -it <container-id> ping <service-name>
docker exec -it <container-id> nslookup <service-name>

# Ver DNS resolution
docker exec -it <container-id> cat /etc/resolv.conf
```

### Volumes

```bash
# Listar volumes
docker volume ls

# Inspect volume
docker volume inspect <volume-name>
```

### Recuperação

> **Quando usar `docker service update --force`**: Para forçar recriação de todas as tasks

```bash
# Forçar recreation de todas as tasks
docker service update --force myservice

# Remover service problemática e recriar
docker service rm myservice
docker service create ...

# Restart de nó (se problema for nó específico)
docker node ls
docker node demote <node-id>
docker node rm <node-id>
```

### Comandos de Emergência

```bash
# Verificar quorum
docker node ls

# Forçar novo cluster (ÚLTIMO RECURSO)
docker swarm init --force-new-cluster

# Sair do swarm (nó)
docker swarm leave --force
```

## Exercícios

1. **Exercício 1**: Criar um service que intencionalmente falha e usar os comandos de debug
2. **Exercício 2**: Forçar constraint impossível e observar o comportamento
3. **Exercício 3**: Criar 2 serviços em redes diferentes e usar os comandos de rede para debugar
4. **Exercício 4**: Simular um nó falho e usar inspect para diagnosticar

## Referências

### Documentação Oficial
- [Troubleshooting](https://docs.docker.com/engine/swarm/troubleshooting/)
- [Debug a service](https://docs.docker.com/engine/swarm/services/#debug-a-service)

### Comandos CLI
- [docker service ps]()
- [docker service logs]()
- [docker service inspect]()
- [docker node inspect]()
