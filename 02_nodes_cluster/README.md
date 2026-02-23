# 02 - Nodes e Cluster

## Objetivos

- Compreender a arquitetura de nodes (manager e worker)
- Entender o algoritmo Raft e quorum
- Adicionar e remover nodes do cluster
- Gerenciar disponibilidade de nodes
- Promover/demover nodes entre roles

## Teoria

### Arquitetura de Nodes

No Docker Swarm, um **Node** é uma instância do Docker Engine que participa do swarm. Existem dois tipos:

#### Manager Node

Os manager nodes desempenham funções de gerenciamento e orquestração do cluster:

- Mantêm o estado do cluster usando **Raft Consensus**
- Respondem às APIs (`docker service create`, etc)
- Executam o scheduler de tarefas
- Um líder (Leader) toma decisões de orquestração

> Se o Leader ficar indisponível, os outros managers elegem um novo líder.

#### Worker Node

- Executam as tasks (containers)
- Não participam do gerenciamento de estado
- Se comunicam com os managers
- Por padrão, managers também executam tasks

```
┌─────────────────────────────────────────────────────┐
│                   CLUSTER                            │
│                                                      │
│  ┌─────────────┐       ┌─────────────┐              │
│  │   Manager   │◄──────│   Manager   │              │
│  │   (Leader)  │       │  (Reachable)│              │
│  └──────┬──────┘       └──────┬──────┘              │
│         │                      │                      │
│         │    ┌────────────────┴───────┐             │
│         │    │    Raft Consensus     │             │
│         │    │    (estado igual)     │             │
│         │    └────────────────┬───────┘             │
│         │                      │                      │
│    ┌────┴────┐           ┌────┴────┐               │
│    │ Worker  │           │ Worker  │               │
│    │(node-1)│           │(node-2) │               │
│    └─────────┘           └─────────┘               │
└─────────────────────────────────────────────────────┘
```

### Algoritmo Raft e Quorum

O Swarm usa o **Raft Consensus Algorithm** para manter consistência:

> O objetivo é garantir que todos os managers armazenem o mesmo estado consistente. Se o Leader morrer, qualquer outro Manager pode retomar as tarefas.

#### Quorum

- O swarm precisa de **maioria simples** de managers ativos
- Com 3 managers: tolera 1 falha (2 disponíveis)
- Com 5 managers: tolera 2 falhas (3 disponíveis)

| Managers | Falhas Toleradas | quorum |
|----------|------------------|--------|
| 1 | 0 | 1 |
| 3 | 1 | 2 |
| 5 | 2 | 3 |
| 7 | 3 | 4 |

> **Regra**: Mantenha sempre número ímpar de managers!

> Se o quorum for perdido, o swarm **não consegue** agendar novas tarefas ou gerenciar o cluster.

### Availability

Cada node pode ter três estados de disponibilidade:

| Estado | Descrição |
|--------|-----------|
| **Active** | Aceita novas tarefas (padrão) |
| **Paused** | Não aceita novas tarefas, mantém as existentes |
| **Drain** | Não aceita novas tarefas, redistribui as existentes para outros nós |

### O que acontece ao juntar um nó

Quando um nó entra no swarm (`docker swarm join`):

1. Docker Engine muda para modo Swarm
2. Solicita certificado TLS do manager
3. Nomeia o nó com o hostname da máquina
4. Define disponibilidade como `Active`
5. Estende a rede `ingress` para o nó

## Prática

### Adicionando Nodes

> **Quando usar `docker swarm join-token`**: Para obter o comando de entrada de novos nós

```bash
# No manager: gerar token para worker
docker swarm join-token worker

# No manager: gerar token para manager
docker swarm join-token manager
```

> **Quando usar `docker swarm join`**: Para adicionar um nó ao swarm (como worker ou manager)

```bash
# No novo nó: executar o comando retornado
docker swarm join --token SWMTKN-1-xxx manager-ip:2377
```

### Gerenciando Nodes

> **Quando usar `docker node ls`**: Para listar todos os nós e ver status, disponibilidade e papel

```bash
# Ver todos os nodes (execute no manager)
docker node ls
```

Output:
```
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
46aqrk4e473hjbt745z53cr3t    node-5    Ready   Active        Reachable
61pi3d91s0w3b90ijw3deeb2q    node-4    Ready   Active        Reachable
a5b2m3oghd48m8eu391pefq5u    node-3    Ready   Active        
e7p8btxeu3ioshyuj6lxiv6g0    node-2    Ready   Active        
ehkv3bcimagdese79dn78otj5 *  node-1    Ready   Active        Leader
```

> **Quando usar `docker node inspect`**: Para ver detalhes de um nó específico

```bash
# Informações detalhadas de um nó
docker node inspect node-1 --pretty
```

> **Quando usar `docker node update --availability`**: Para alterar disponibilidade (manutenção, drenar tasks)

```bash
# Drain: para de receber tasks e redistribui as existentes
docker node update --availability drain node-1

# Paused: não recebe novas tasks, mantém as existentes
docker node update --availability pause node-1

# Active: volta ao normal
docker node update --availability active node-1
```

> **Quando usar `docker node promote`**: Para promover worker → manager

```bash
docker node promote node-3
```

> **Quando usar `docker node demote`**: Para rebaixar manager → worker

```bash
docker node demote node-3
```

### Removendo Nodes

> **Quando usar `docker swarm leave`**: Para um nó sair do swarm

```bash
# No worker: sai normalmente
docker swarm leave

# No manager: força saída (use com cuidado!)
docker swarm leave --force
```

> **Quando usar `docker node rm`**: Para remover nó inativo da lista

```bash
# Remover nó que já saiu (status: Down)
docker node rm node-id
```

## Exercícios

1. **Exercício 1**: Se ainda não tem swarm, inicialize um. Se já tem, adicione mais um worker (pode ser no mesmo host para teste)
2. **Exercício 2**: Liste os nodes e observe seus status, roles e manager status
3. **Exercício 3**: Execute `docker node inspect` em um nó e explore os campos
4. **Exercício 4**: Faça um node worker virar manager e observe a mudança
5. **Exercício 5**: Simule manutenção: coloque um node em `drain`, observe as tasks sendo redistribuídas
6. **Exercício 6**: Entenda o quorum: tente sair do swarm forçadamente com um manager e observe o aviso

## Referências

### Documentação Oficial
- [manage-nodes](../00_docs/guia/manage-nodes.md) - Gerenciar nós
- [join-nodes](../00_docs/guia/join-nodes.md) - Adicionar nós
- [raft](../00_docs/guia/raft.md) - Algoritmo Raft

### Comandos CLI
- [docker swarm join-token](../00_docs/cli/04_docker_swarm_join-token.md)
- [docker swarm join](../00_docs/cli/03_docker_swarm_join.md)
- [docker swarm leave](../00_docs/cli/05_docker_swarm_leave.md)
