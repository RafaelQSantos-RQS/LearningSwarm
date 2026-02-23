# 02 - Nodes e Cluster

## Objetivos

- Compreender a arquitetura de nodes (manager e worker)
-icionar e remover nodes Ad do cluster
- Entender o conceito de quorum e sua importância
- Gerenciar disponibilidade de nodes

## Teoria

### Arquitetura de Nodes

No Docker Swarm, um **Node** é uma instância do Docker Engine que participa do cluster. Existem dois tipos:

#### Manager Node
- Mantém o estado do cluster (raft consensus)
- Responde às APIs (docker service create, etc)
- Executa o scheduler de tarefas
- Recomenda-se 3 ou 5 managers para alta disponibilidade

#### Worker Node
- Executa as tasks (containers)
- Não participa do gerenciamento de estado
- Comunicação via manager
- Pode ter 0+N workers

### Quorum

O Swarm usa o algoritmo **Raft Consensus** para manter consistência:
- Precisa de maioria simples de managers ativos
- Com 3 managers: tolera 1 falha
- Com 5 managers: tolera 2 falhas
- **Regra**: Sempre mantenha número ímpar de managers

### Availability

Cada node pode ter três estados:
- **Active**: Aceita novas tarefas (padrão)
- **Paused**: Não recebe novas tarefas, mantém as existentes
- **Drained**: Não recebe novas tarefas, tarefas existentes são redistribuídas

## Prática

### Gerenciando Nodes

```bash
# Ver todos os nodes (execute no manager)
docker node ls

# Informações detalhadas de um node
docker node inspect self
docker node inspect node-id

# Atualizar disponibilidade do node
docker node update --availability drain node-id
docker node update --availability active node-id

# Promoting/Demoting nodes
docker node promote node-id      # worker -> manager
docker node demote node-id       # manager -> worker
```

### Adicionando Nodes

```bash
# No manager: gerar token para adicionar worker
docker swarm join-token worker

# No manager: gerar token para adicionar manager
docker swarm join-token manager

# No novo nó: executar o comando retornado
docker swarm join --token SWMTKN-1-xxx manager-ip:2377
```

### Removendo Nodes

```bash
# No worker: sair do swarm
docker swarm leave

# No manager: remover worker do cluster
docker node rm node-id

# No manager: forçar manager a sair (se necessário)
docker node demote node-id
docker node rm node-id
```

### Demonstração Visual

```
┌─────────────────────────────────────────────────┐
│                   CLUSTER                       │
│                                                 │
│  ┌─────────────┐     ┌─────────────┐           │
│  │   Manager   │◄────│   Manager   │           │
│  │   (leader)  │     │   (replica) │           │
│  └──────┬──────┘     └──────┬──────┘           │
│         │                   │                  │
│         │    ┌──────────────┴──────┐          │
│         │    │   Raft Consensus    │          │
│         │    └──────────────┬───────┘          │
│         │                   │                  │
│    ┌────┴────┐         ┌────┴────┐            │
│    │ Worker  │         │ Worker  │            │
│    │ (node-1)│         │ (node-2)│            │
│    └─────────┘         └─────────┘            │
└─────────────────────────────────────────────────┘
```

## Exercícios

1. **Exercício 1**: Se ainda não tem swarm, inicialize um. Se já tem, adicione mais um worker (pode ser no mesmo host para teste)
2. **Exercício 2**: Liste os nodes e observe seus status e roles
3. **Exercício 3**: Faça um node worker virar manager e observe a mudança
4. **Exercício 4**:Simule uma falha: coloque um manager em drain e observe o comportamento dos serviços

## Referências

- [How nodes work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/)
- [Manage nodes in a swarm](https://docs.docker.com/engine/swarm/manage-nodes/)
- [Swarm administration guide](https://docs.docker.com/engine/swarm/admin_guide/)
