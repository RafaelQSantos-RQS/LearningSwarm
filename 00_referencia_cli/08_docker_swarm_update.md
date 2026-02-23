# docker swarm update

Atualiza configurações do Swarm.

## Uso

```bash
docker swarm update [OPTIONS]
```

## Opções

| Opção | Padrão | Descrição |
|-------|---------|------------|
| `--autolock` | - | Habilita/desabilita autolock (`true` ou `false`) |
| `--cert-expiry` | `2160h0m0s` | Validade dos certificados (ns\|us\|ms\|s\|m\|h) |
| `--dispatcher-heartbeat` | `5s` | Frequência de heartbeat do dispatcher |
| `--external-ca` | - | CA externa para certificados |
| `--max-snapshots` | - | Número de snapshots Raft para reter |
| `--snapshot-interval` | `10000` | Intervalo entre snapshots Raft |
| `--task-history-limit` | `5` | Limite de histórico de tasks |

## Descrição

Atualiza parâmetros do swarm após sua criação.

## Exemplos

### Habilitar autolock

```bash
docker swarm update --autolock=true
```

### Alterar validade dos certificados

```bash
docker swarm update --cert-expiry=720h
```

### Alterar heartbeat do dispatcher

```bash
docker swarm update --dispatcher-heartbeat=10s
```

### Alterar limite de histórico de tasks

```bash
docker swarm update --task-history-limit=10
```

## Quando usar

- **--autolock**: Para ambientes mais seguros
- **--cert-expiry**: Para alterar validade de certificados
- **--task-history-limit**: Para debugging (mais histórico) ou performance (menos)

## Referência

[Documentação oficial](https://docs.docker.com/reference/cli/docker/swarm/update/)
