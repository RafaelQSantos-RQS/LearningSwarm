# docker swarm join

Junta um nó ao Swarm.

## Uso

```bash
docker swarm join [OPTIONS] HOST:PORT
```

## Opções

| Opção | Padrão | Descrição |
|-------|---------|------------|
| `--advertise-addr` | auto | Endereço IP/interface para advertise |
| `--availability` | `active` | Disponibilidade do nó (`active`, `pause`, `drain`) |
| `--data-path-addr` | - | Endereço para tráfego de dados |
| `--listen-addr` | `0.0.0.0:2377` | Endereço de escuta |
| `--token` | - | Token de entrada no swarm |

## Descrição

Junta o nó ao swarm como manager ou worker, dependendo do token usado:
- **Manager token**: nó vira manager
- **Worker token**: nó vira worker

## Exemplos

### Como worker

```bash
docker swarm join --token SWMTKN-1-xxx 192.168.99.121:2377
```

Output:
```
This node joined a swarm as a worker.
```

### Como manager

```bash
docker swarm join --token SWMTKN-1-xxx 192.168.99.121:2377
```

Output:
```
This node joined a swarm as a manager.
```

### Com configuração customizada

```bash
# Com IP específico e availability drain
docker swarm join \
  --token SWMTKN-1-xxx \
  --advertise-addr 192.168.1.10:2377 \
  --availability drain \
  192.168.99.121:2377
```

## Quando usar

- **Adicionar worker**: Para aumentar capacidade de execução de containers
- **Adicionar manager**: Para alta disponibilidade (max 3-7 managers)
- **--availability=drain**: Para nós que não devem executar tasks (ex: managers dedicados)
- **--advertise-addr**: Em ambientes com múltiplas interfaces de rede

## Referência

[Documentação oficial](https://docs.docker.com/reference/cli/docker/swarm/join/)
