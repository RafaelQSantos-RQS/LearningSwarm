# 03 - Services

## Objetivos

- Criar e gerenciar services no Docker Swarm
- Compreender a diferença entre services, tasks e containers
- Configurar réplicas e políticas de update
- Entender o ciclo de vida de uma task
- Publicar portas e usar o routing mesh

## Teoria

### O que é um Service?

Um **Service** é a definição de como você quer executar uma aplicação no Swarm. É o modelo **declarativo** - você define o estado desejado e o Docker se encarrega de mantê-lo.

O estado desejado inclui:
- Imagem e tag
- Número de réplicas
- Portas expostas
- Recursos (CPU, memória)
- Políticas de restart
- Constraints de placement

### Service vs Task vs Container

```
┌─────────────────────────────────────────────────┐
│                    SERVICE                       │
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

#### Replicated
Número específico de réplicas distribuídas entre os nós. Use para web servers, APIs, etc.

#### Global
Uma task em cada nó disponível. Ideal para: monitoring agents, logging, security.

### Publishing Ports

O Swarm oferece duas formas de expor portas:

1. **Routing Mesh (default)**: Publica a porta em todos os nós. O tráfego é roteado para uma task.

```
┌─────────────────────────────────────────────────────┐
│              ROUTING MESH                            │
│                                                      │
│  Request ──► Node 1:8080 ──► (IPVS) ──► Node 3:80 │
│                  (não tem task)        (tem task)   │
│                                                      │
└─────────────────────────────────────────────────────┘
```

2. **Host Mode**: Publica a porta diretamente no nó onde a task está rodando.

### Imagens e Tags

Ao criar um service:
- Se não especificar tag, usa `latest`
- A tag é resolvida para um **digest** (hash) no momento da criação
- Todas as tasks usam o **mesmo digest** - mesma versão da imagem
- Para atualizar, use `docker service update --image`

### Recursos e Constraints

- **Resources**: CPU e memória (limits e reservations)
- **Constraints**: Limitar onde o service pode rodar (node.role, node.labels, etc)
- **Preferences**: Tentar distribuir igualmente (best-effort)

## Prática

### Criando Services

> **Quando usar `docker service create`**: Para criar um service gerenciado pelo Swarm

```bash
# Service básico
docker service create --name web --replicas 3 -p 80:80 nginx:alpine

# Com recursos
docker service create \
  --name api \
  --replicas 2 \
  --limit-cpu 0.5 \
  --limit-memory 512M \
  -e NODE_ENV=production \
  nginx:alpine

# Com constraints
docker service create \
  --name db \
  --constraint "node.role==manager" \
  -e POSTGRES_PASSWORD=teste@123 \
  postgres:15

# Global service
docker service create --mode global --name monitoring prom/prometheus
```

### Modos de Service

```bash
# Replicated (padrão)
docker service create --name web --replicas 3 nginx

# Global (uma task por nó)
docker service create --mode global --name agent myagent
```

### Publicando Portas

> **Quando usar `-p` ou `--publish`**: Para expor portas do service

```bash
# Routing mesh (todos os nós)
docker service create -p 8080:80 nginx

# Host mode (diretamente no nó)
docker service create --publish mode=host,target=80,published=8080 nginx
```

### Gerenciando Services

> **Quando usar `docker service ls`**: Para listar todos os serviços

> **Quando usar `docker service ps`**: Para ver as tasks de um serviço

```bash
# Listar services
docker service ls

# Ver tasks de um service
docker service ps web

# Inspect service (detalhes completos)
docker service inspect web
docker service inspect --pretty web  # formato legível

# Logs do service (agregado de todas as tasks)
docker service logs -f web

# Escalar service
docker service scale web=5

# Atualizar service
docker service update --image nginx:latest web
docker service update --replicas 3 web
docker service update --env-add NODE_ENV=production web

# Remover service
docker service rm web
```

### Atualizações e Rollbacks

> **Quando usar `docker service update`**: Para alterar qualquer configuração do service (faz rolling update)

> **Quando usar `docker service rollback`**: Para reverter para a configuração anterior

```bash
# Atualizar com política customizada
docker service update \
  --image nginx:1.25 \
  --update-delay 10s \
  --update-parallelism 1 \
  --update-failure-action pause \
  web

# Rollback
docker service rollback web
```

### Redes e Volumes

```bash
# Conectar a uma network overlay
docker service create --network minha-rede --name api myapi

# Adicionar network
docker service update --network-add minha-rede api

# Adicionar volume
docker service create --mount type=volume,src=meuvolume,dst=/data --name db postgres
```

## Exercícios

1. **Exercício 1**: Criar um service com 3 réplicas e observar a distribuição das tasks
2. **Exercício 2**: Publicar uma porta e testar o routing mesh (acessar por qualquer nó)
3. **Exercício 3**: Criar um global service e verificar que há uma task em cada nó
4. **Exercício 4**: Fazer um update do service (mudar imagem) e observar o rolling update
5. **Exercício 5**: Testar rollback após um update
6. **Exercício 6**: Criar service com limites de CPU/memória e verificar com `docker service inspect`

## Referências

### Documentação Oficial
- [services](../00_docs/conceitos/services.md) - Conceitos de services
- [Deploy services](../00_docs/conceitos/services.md) - Criar e gerenciar services

### Comandos CLI
- [docker service create]()
- [docker service]()
- [docker service update]()
- [docker service ps]()
