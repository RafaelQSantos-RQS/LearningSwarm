# docker swarm init

Inicializa um Swarm.

## Uso

```bash
docker swarm init [OPTIONS]
```

## Opções

| Opção | Padrão | Descrição |
|-------|---------|------------|
| `--advertise-addr` | auto | Endereço IP/interface para advertise (format: `<ip|interface>[:port]`) |
| `--autolock` | false | Habilita autolock dos managers (requer chave de unlock) |
| `--availability` | `active` | Disponibilidade do nó (`active`, `pause`, `drain`) |
| `--cert-expiry` | `2160h0m0s` | Validade dos certificados (ns\|us\|ms\|s\|m\|h) |
| `--data-path-addr` | - | Endereço para tráfego de dados (containers) |
| `--data-path-port` | `4789` | Porta UDP para tráfego de dados (1024-49151) |
| `--default-addr-pool` | - | Pool de subnets padrão (CIDR) |
| `--default-addr-pool-mask-length` | `24` | Máscara das pools de subnet |
| `--dispatcher-heartbeat` | `5s` | Frequência de heartbeat do dispatcher |
| `--external-ca` | - | URL de CA externa para certificados |
| `--force-new-cluster` | false | Força criação de novo cluster |
| `--listen-addr` | `0.0.0.0:2377` | Endereço de escuta |
| `--max-snapshots` | - | Número de snapshots Raft para reter |
| `--snapshot-interval` | `10000` | Intervalo entre snapshots Raft |
| `--task-history-limit` | `5` | Limite de histórico de tasks |

## Descrição

Inicializa um swarm. O Docker Engine se torna um manager no swarm de nó único criado.

Gera dois tokens aleatórios:
- **Worker token**: para adicionar workers
- **Manager token**: para adicionar managers

## Exemplos

### Inicialização básica

```bash
# Com IP específico
docker swarm init --advertise-addr 192.168.99.121

# Com interface específica
docker swarm init --advertise-addr eth0:2377
```

### Com autolock (recomendado em produção)

```bash
docker swarm init --autolock
```

Output:
```
Swarm initialized: current node (abc123) is now a manager.

To add a worker to this swarm, run:
    docker swarm join --token SWMTKN-1-xxx manager-ip:2377

To add a manager to this swarm, run:
    docker swarm join-token manager
```

### Com porta de dados customizada

```bash
docker swarm init --data-path-port=7777
```

### Com subnet pools customizadas

```bash
docker swarm init \
  --default-addr-pool 30.30.0.0/16 \
  --default-addr-pool 40.40.0.0/16
```

## Quando usar

- **Iniciar swarm**: Primeiro comando para criar um cluster
- **--autolock**: Ambientes de produção com múltiplos managers
- **--advertise-addr**: Múltiplos IPs ou interfaces de rede
- **--data-path-addr**: Separar tráfego de management de dados

## Referência

[Documentação oficial](https://docs.docker.com/reference/cli/docker/swarm/init/)
