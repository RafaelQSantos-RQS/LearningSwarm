# 01 - Introdução ao Docker Swarm

## Objetivos

- Compreender o que é Docker Swarm e quando usá-lo
- Diferenciar Docker Compose de Docker Swarm
- Conhecer os principais conceitos: nodes, services, stacks, tasks
- Inicializar um swarm básico

## Teoria

### O que é Docker Swarm?

Docker Swarm é a ferramenta de orquestração nativa do Docker. Permite agrupar múltiplos Docker Hosts em um único cluster virtual, facilitando o deploy, escala e gerenciamento de aplicações containerizadas.

### Compose vs Swarm

| Aspecto | Docker Compose | Docker Swarm |
|---------|----------------|--------------|
| Escopo | Hosts únicos | Múltiplos hosts |
| Orquestração | Limitada | Completa |
| Scaling | Manual | Automático |
| Service Discovery | Não nativo |built-in |
| Load Balancing | Não nativo | Ingress integrado |
| Comandos | `docker-compose` | `docker service`, `docker stack` |

### Conceitos Fundamentais

1. **Node**: Um Docker Engine participante do swarm
   - **Manager Node**: Gerencia o cluster (scheduling, state)
   - **Worker Node**: Executa as tarefas (tasks)

2. **Service**: Definição de como executar um container
   - Imagem a usar
   - Réplicas desejadas
   - Políticas de restart
   - Recursos (CPU, memória)

3. **Task**: Unitade de trabalho (container em execução)
   - Atribuída a um nó
   - Tem estado: pending, running, failed

4. **Stack**: Conjunto de serviços relacionados
   - Definido em arquivo YAML (como compose)
   - Deploy de toda aplicação de uma vez

### Quando usar Swarm?

- **Ideal para**: Apps médios/pequenos,microserviços simples, times que já usam Docker
- **Considere Kubernetes para**: Apps complexos, necessidade de ingress avanzado, ecosistema Enterprise

## Prática

### Referência de Comandos

Consulte a pasta `00_referencia_cli` para documentação completa de cada comando.

| Comando | Descrição | Referência |
|---------|-----------|------------|
| `docker swarm init` | Inicializa o swarm | [init](../00_referencia_cli/02_docker_swarm_init.md) |
| `docker node ls` | Lista nós do cluster | - |
| `docker service create` | Cria um service | - |
| `docker service ls` | Lista services | - |
| `docker service ps` | Lista tasks de um service | - |
| `docker service rm` | Remove um service | - |
| `docker service inspect` | Detalhes do service | - |
| `docker service logs` | Logs do service | - |
| `docker service scale` | Escala réplicas | - |
| `docker service update` | Atualiza service | - |

### Inicializando um Swarm

```bash
# Iniciar swarm (este nó vira manager)
docker swarm init

# Verificar status
docker info | grep -i swarm

# Listar nós do cluster
docker node ls
```

### RedesCriadas

Ao iniciar o Swarm, novas redes aparecem automaticamente:

```bash
docker network ls
# NETWORK ID     NAME              DRIVER    SCOPE
# ...           ingress           overlay   swarm   # <-- Load balancing do Swarm
# ...           docker_gwbridge   bridge    local   # <-- Bridge host<->containers
```

Ao sair do Swarm (`docker swarm leave`), as redes `ingress` e `docker_gwbridge` (escopo swarm) são removidas. A rede `docker_gwbridge` permanece porque é a bridge padrão do Docker para comunicação host ↔ containers.

> **Nota**: As redes `bridge`, `host` e `none` são redes padrão do Docker que existem em qualquer instalação, com ou sem Swarm.

### Primeiro Service

> **Quando usar `docker service create`**: Para criar um container isolado gerenciado pelo Swarm (sem compose)

```bash
# Criar um service básico (similar ao docker run)
docker service create --name nginx --replicas 1 -p 80:80 nginx:alpine

# Listar serviços
docker service ls

# Ver detalhes do service
docker service ps nginx

# Remover service
docker service rm nginx
```

### Comandos Essenciais

> **Quando usar `docker service inspect`**: Para debugar, ver configuração completa ou status de update
> 
> **Quando usar `docker service logs`**: Para ver logs agregados de todas as réplicas (diferente do docker logs)
> 
> **Quando usar `docker service scale`**: Para aumentar/diminuir réplicas rapidamente
> 
> **Quando usar `docker service update`**: Para alterar imagem, variáveis, portas, recursos (faz rolling update automático)

```bash
# Inspect service (detalhes completos)
docker service inspect nginx

# Logs do service
docker service logs nginx

# Escalar service
docker service scale nginx=3

# Atualizar service (imagem, replicas, etc)
docker service update --image nginx:latest nginx
```

## Exercícios

1. **Exercício 1**: Inicializar um swarm local e verificar os nodes
2. **Exercício 2**: Criar um service com nginx, expor numa porta e acessar pelo navegador
3. **Exercício 3**: Escalar o service para 3 réplicas e observar a distribuição
4. **Exercício 4**: Remover o service e o swarm

## Referências

### Comandos CLI
- [docker swarm](../00_referencia_cli/00_docker_swarm.md)
- [docker swarm init](../00_referencia_cli/02_docker_swarm_init.md)
- [docker swarm join](../00_referencia_cli/03_docker_swarm_join.md)
- [docker swarm join-token](../00_referencia_cli/04_docker_swarm_join-token.md)
- [docker swarm leave](../00_referencia_cli/05_docker_swarm_leave.md)

### Documentação
- [Docker Swarm Overview](https://docs.docker.com/engine/swarm/)
- [Swarm Mode Tutorial](https://docs.docker.com/engine/swarm/swarm-tutorial/)
- [Docker Engine Swarm](https://docs.docker.com/engine/swarm/key-concepts/)
