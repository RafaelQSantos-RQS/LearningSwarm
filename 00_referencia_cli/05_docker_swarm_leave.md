# docker swarm leave

Faz um nó sair do Swarm.

## Uso

```bash
docker swarm leave [OPTIONS]
```

## Opções

| Opção | Padrão | Descrição |
|-------|---------|------------|
| `-f`, `--force` | false | Força o nó a sair, ignorando avisos |

## Descrição

- **Worker**: Sai normalmente do swarm
- **Manager**: Precisa de `--force` (ou demover antes)

## Exemplos

### Worker sai do swarm

```bash
# No nó worker
docker swarm leave
```

Output:
```
Node left the swarm.
```

### Forçar manager a sair

```bash
# No nó manager
docker swarm leave --force
```

> **Atenção**: Isso pode quebrar o quorum se não houver outros managers!

## Quando usar

- **Remover worker**: Quando não precisa mais do nó
- **--force**: Para manager em swarm de nó único, ou em emergências
- **Remover nó inativo**: Após sair, usar `docker node rm` para limpar lista

## Boas práticas para remover manager

1. Demover para worker: `docker node demote <node-id>`
2. Drain: `docker node update --availability drain <node-id>`
3. Sair: `docker swarm leave` (sem --force)

## Referência

[Documentação oficial](https://docs.docker.com/reference/cli/docker/swarm/leave/)
