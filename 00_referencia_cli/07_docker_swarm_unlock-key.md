# docker swarm unlock-key

Gerencia a chave de desbloqueio do Swarm.

## Uso

```bash
docker swarm unlock-key [OPTIONS]
```

## Opções

| Opção | Padrão | Descrição |
|-------|---------|------------|
| `-q`, `--quiet` | false | Exibe apenas a chave |
| `--rotate` | false | Rotaciona a chave |

## Descrição

Gerencia a chave necessária para desbloquear managers após restart do Docker, quando autolock está habilitado.

## Exemplos

### Visualizar chave

```bash
docker swarm unlock-key
```

Output:
```
To unlock a swarm manager after it restarts, run the `docker swarm unlock` command and provide the following key:

    SWMKEY-1-xxx

Remember to store this key in a password manager, since without it you will not be able to restart the manager.
```

### Exibir apenas a chave

```bash
docker swarm unlock-key -q
# SWMKEY-1-xxx
```

### Rotacionar chave

```bash
docker swarm unlock-key --rotate
```

## Quando usar

- **Visualizar**: Para backup seguro da chave
- **--rotate**: Quando a chave foi comprometida, ou por segurança periódica

## Referência

[Documentação oficial](https://docs.docker.com/reference/cli/docker/swarm/unlock-key/)
