# 01 - Introdução ao Docker Swarm

## Objetivos

- Compreender o que é Docker Swarm e quando usá-lo
- Diferenciar Docker Compose de Docker Swarm
- Conhecer os principais conceitos: nodes, services, stacks, tasks
- Inicializar um swarm básico
- Entender como o Swarm mantém o estado desejado

## Teoria

### O que é Docker Swarm?

Docker Swarm é a ferramenta de orquestração **nativa** do Docker. Faz parte do Docker Engine (desde a versão 1.12) e permite agrupar múltiplos Docker Hosts em um único cluster virtual.

> O Swarm é construído usando [SwarmKit](https://github.com/docker/swarmkit/), um projeto separado que implementa a camada de orquestração do Docker.

### Compose vs Swarm

| Aspecto | Docker Compose | Docker Swarm |
|---------|----------------|--------------|
| Escopo | Hosts únicos | Múltiplos hosts |
| Orquestração | Limitada | Completa |
| Scaling | Manual | Automático |
| Service Discovery | Não nativo | Built-in |
| Load Balancing | Não nativo | Ingress integrado |
| Comandos | `docker-compose` | `docker service`, `docker stack` |
| State Management | Não | Estado desejado (desired state) |

### Conceitos Fundamentais

#### Node

Um **node** é uma instância do Docker Engine participante do swarm. Você pode executar um ou mais nodes em um único computador ou servidor cloud.

```
┌─────────────────────────────────────────────────────┐
│                    SWARM CLUSTER                    │
│                                                     │
│  ┌─────────────┐         ┌─────────────┐           │
│  │   Manager   │◄───────►│   Manager   │           │
│  │  (leader)   │         │  (replica)  │           │
│  └──────┬──────┘         └──────┬──────┘           │
│         │                        │                  │
│         │    ┌──────────────────┘                  │
│         │    │                                       │
│         ▼    ▼                                       │
│  ┌─────────────┐         ┌─────────────┐           │
│  │   Worker    │         │   Worker    │           │
│  └─────────────┘         └─────────────┘           │
└─────────────────────────────────────────────────────┘
```

Existem dois tipos de nodes:

- **Manager Node**: Gerencia o cluster (scheduling, state, consensus Raft)
- **Worker Node**: Executa as tarefas (tasks)

> Por padrão, nodes managers também executam tasks. Você pode configurá-los para serem exclusivamente managers.

#### Service

Um **service** é a definição das tasks a serem executadas nos nodes. É a estrutura central do swarm e a principal forma de interação do usuário.

Quando você cria um service, especifica:
- Imagem do container
- Comandos a executar
- Número de réplicas
- Redes e volumes
- Políticas de restart

#### Task

Uma **task** carrega um container Docker e os comandos a serem executados dentro dele. É a **unidade atômica de agendamento** do swarm.

> Uma task não pode mover para outro node após ser atribuída. Ela só pode executar no node atribuído ou falhar.

#### Stack

Uma **stack** é um conjunto de serviços relacionados. Definido em arquivo YAML (como compose), permite o deploy de toda aplicação de uma vez.

### Estado Desejado (Desired State)

Uma das principais vantagens do Swarm sobre containers standalone:

> Você pode modificar a configuração de um service (redes, volumes, etc) sem precisar reiniciar manualmente. O Docker atualiza a configuração, para as tasks desatualizadas e cria novas tasks com a nova configuração.

### Quando usar Swarm?

- **Ideal para**: Apps médios/pequenos, microserviços simples, times que já usam Docker
- **Considere Kubernetes para**: Apps complexos, necessidade de ingress avançado, ecosistema Enterprise

## Prática

### Redes Criadas

Ao iniciar o Swarm, novas redes aparecem automaticamente:

```bash
docker network ls
# NETWORK ID     NAME              DRIVER    SCOPE
# ...           ingress           overlay   swarm   # <-- Load balancing do Swarm
# ...           docker_gwbridge   bridge    local   # <-- Bridge host<->containers
```

Ao sair do Swarm (`docker swarm leave`), as redes `ingress` e `docker_gwbridge` (escopo swarm) são removidas.

> **Nota**: As redes `bridge`, `host` e `none` são redes padrão do Docker que existem em qualquer instalação.

### Inicializando um Swarm

> **Quando usar `docker swarm init`**: Para criar um novo cluster swarm (primeiro comando)

```bash
# Iniciar swarm (este nó vira manager)
docker swarm init

# Verificar status
docker info | grep -i swarm

# Listar nós do cluster
docker node ls
```

### Primeiro Service

> **Quando usar `docker service create`**: Para criar um container gerenciado pelo Swarm (sem compose/stack)

```bash
# Criar um service básico
docker service create --name nginx --replicas 1 -p 80:80 nginx:alpine

# Listar serviços
docker service ls

# Ver detalhes do service (tasks)
docker service ps nginx

# Remover service
docker service rm nginx
```

### Comandos Essenciais

> **Quando usar `docker service inspect`**: Para debugar, ver configuração completa ou status de update

> **Quando usar `docker service logs`**: Para ver logs agregados de todas as réplicas (diferente do docker logs)

> **Quando usar `docker service scale`**: Para aumentar/diminuir réplicas rapidamente

> **Quando usar `docker service update`**: Para alterar imagem, variáveis, portas, recursos (faz rolling update automático)

```bash
# Inspect service (detalhes completos)
docker service inspect nginx

# Logs do service
docker service logs -f nginx

# Escalar service
docker service scale nginx=3

# Atualizar service (imagem)
docker service update --image nginx:latest nginx

# Ver histórico de updates
docker service inspect --pretty nginx
```

## Exercícios

1. **Exercício 1**: Inicializar um swarm local e verificar os nodes
2. **Exercício 2**: Observar as redes criadas pelo Swarm (`docker network ls`)
3. **Exercício 3**: Criar um service com nginx, expor numa porta e acessar pelo navegador
4. **Exercício 4**: Escalar o service para 3 réplicas e observar a distribuição (`docker service ps nginx`)
5. **Exercício 5**: Remover o service e o swarm

## Referências

### Documentação Oficial
- [key-concepts](../00_docs/conceitos/key-concepts.md) - Conceitos fundamentais
- [swarm-mode](../00_docs/conceitos/swarm-mode.md) - Modo Swarm
- [Docker Swarm Overview](https://docs.docker.com/engine/swarm/)

### Comandos CLI
- [docker swarm init](../00_docs/cli/02_docker_swarm_init.md)
- [docker service create](../00_docs/cli/00_docker_swarm.md)
- [docker service](../00_docs/cli/00_docker_swarm.md)
