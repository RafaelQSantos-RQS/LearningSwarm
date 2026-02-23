# 06 - Secrets e Configs

## Objetivos

- Gerenciar dados sensíveis de forma segura no Swarm
- Criar e usar Docker Secrets
- Criar e usar Docker Configs
- Entender a diferença entre Secrets e Configs

## Teoria

### O que são Secrets?

Docker Secrets são dados sensíveis (senhas, tokens, chaves SSH, certificados) que:
- São criptografados em repouso e em trânsito
- São distribuídos automaticamente para todos os nós managers
- São montados em `/run/secrets/` nos containers
- Podem ser atualizados com rolling update

### O que são Configs?

Docker Configs são similares aos Secrets mas para dados não sensíveis:
- Arquivos de configuração (nginx.conf, app.json, etc)
- Não precisam de criptografia
- Montados em `/etc/config/`

### Compose vs Swarm

```yaml
# Compose: usa arquivo .env ou variáveis (não seguro)
# Compose: secrets são passados como variáveis de ambiente (NÃO USE EM PROD)

# Swarm: secrets são gerenciados nativamente
version: "3.8"

services:
  db:
    image: postgres
    secrets:
      - db_password
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password

configs:
  nginx_config:
    file: ./nginx.conf

secrets:
  db_password:
    file: ./db_password.txt
```

### Secrets no Compose vs Swarm

| Aspecto | Compose | Swarm |
|---------|---------|-------|
| Criptografia | Não | Sim (AES-256) |
| Escopo | Host | Cluster |
| Distribuição | Manual | Automática |
| Atualização | Recreate container | Rolling update |

## Prática

### Criando Secrets

```bash
# Criar secret a partir de arquivo
echo "minha-senha-secreta" | docker secret create db_password -

# Criar secret a partir do terminal (stdin)
docker secret create my_secret -

# Listar secrets
docker secret ls

# Inspect secret (só mostra nome e ID, não o valor)
docker secret inspect my_secret

# Remover secret
docker secret rm my_secret
```

### Criando Configs

```bash
# Criar config a partir de arquivo
docker config create nginx_config ./nginx.conf

# Criar config a partir do terminal
docker config create my_config -

# Listar configs
docker config ls

# Inspect config
docker config inspect nginx_config

# Remover config
docker config rm my_config
```

### Usando Secrets em Services

```bash
# Com service create
docker service create \
  --name db \
  --secret db_password \
  postgres:15

# Verificar secret dentro do container
docker exec -it <container-id> ls /run/secrets/
docker exec -it <container-id> cat /run/secrets/db_password

# Remover secret do service
docker service update --secret-rm db_password myservice
```

### Usando Secrets em Stacks

```yaml
version: "3.8"

services:
  web:
    image: nginx
    secrets:
      - source: nginx_htpasswd
        target: .htpasswd
      - api_key

  api:
    image: myapi
    environment:
      - DATABASE_PASSWORD=/run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    external: true  # já existe externamente

configs:
  nginx_htpasswd:
    file: ./configs/htpasswd
  nginx_conf:
    file: ./configs/nginx.conf
```

### Atualizando Secrets

```bash
# Atualizar secret (requer recreation do service)
docker secret create --label-alias updated db_password ./new_password.txt
docker service update --secret-rm old_secret --secret-add new_secret myservice
```

## Exercícios

1. **Exercício 1**: Criar um secret e usá-lo em um service PostgreSQL
2. **Exercício 2**: Criar um config e montá-lo num serviço nginx
3. **Exercício 3**: Criar uma stack com secrets e configs externos
4. **Exercício 4**: Simular rotação de secret (atualizar e fazer rolling update)

## Referências

- [Manage sensitive data](https://docs.docker.com/engine/swarm/secrets/)
- [Docker configs](https://docs.docker.com/engine/swarm/configs/)
- [Secret specs in compose](https://docs.docker.com/compose/compose-file/#secrets)
