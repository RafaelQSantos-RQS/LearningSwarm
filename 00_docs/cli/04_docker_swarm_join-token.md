# docker swarm join-token

Gerencia tokens de entrada no Swarm.

## Uso

```bash
docker swarm join-token [OPTIONS] (worker|manager)
```

## Opções

| Opção | Padrão | Descrição |
|-------|---------|------------|
| `-q`, `--quiet` | false | Exibe apenas o token |
| `--rotate` | false | Rotaciona o token |

## Descrição

Gerencia os tokens que permitem nós entrarem no swarm:
- **worker**: para nós workers
- **manager**: para nós managers

## Exemplos

### Visualizar tokens

```bash
# Token de worker
docker swarm join-token worker
```

Output:
```
To add a worker to this swarm, run:

    docker swarm join \
    --token SWMTKN-1-xxx \
    172.17.0.2:2377
```

```bash
# Token de manager
docker swarm join-token manager
```

### Exibir apenas o token

```bash
docker swarm join-token -q worker
# SWMTKN-1-xxx
```

### Rotacionar token

```bash
# Rotacionar token de worker
docker swarm join-token --rotate worker

# Rotacionar token de manager
docker swarm join-token --rotate manager
```

## Quando usar

- **Visualizar token**: Para adicionar novos nós ao swarm
- **--rotate**: Quando token vazou, nó foi comprometido, ou por segurança periódica

## Segurança

> Tokens são segredos! Mantenha-os seguros.

- **Worker token**: Permite executar containers no swarm
- **Manager token**: Permite gerenciar o swarm (mais crítico)

Razões para rotacionar:
- Token exposto em version control
- Token roubado
- Nó comprometido
- Auditoria de segurança periódica

## Referência

[Documentação oficial](https://docs.docker.com/reference/cli/docker/swarm/join-token/)
