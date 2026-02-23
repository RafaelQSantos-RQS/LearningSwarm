# docker swarm unlock

Desbloqueia um managerlocked do Swarm.

## Uso

```bash
docker swarm unlock
```

## Descrição

Desbloqueia um manager que foi travado (locked) após restart do Docker daemon, quando autolock está habilitado.

## Exemplos

```bash
docker swarm unlock
```

Input:
```
Enter unlock key:
```

## Quando usar

- **Autolock habilitado**: Quando `--autolock` foi usado no `docker swarm init`
- **Manager reiniciado**: Após restart do Docker em manager com autolock

## Como obter a chave

```bash
# Ver chave atual (apenas managers)
docker swarm unlock-key --print
```

## Referência

[Documentação oficial](https://docs.docker.com/reference/cli/docker/swarm/unlock/)
