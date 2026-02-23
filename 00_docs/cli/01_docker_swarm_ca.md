# docker swarm ca

Exibe e rotaciona a CA raiz do Swarm.

## Uso

```bash
docker swarm ca [OPTIONS]
```

## Opções

| Opção | Padrão | Descrição |
|-------|---------|------------|
| `--ca-cert` | - | Caminho para o certificado CA raiz (PEM) |
| `--ca-key` | - | Caminho para a chave CA raiz (PEM) |
| `--cert-expiry` | `2160h0m0s` | Validade dos certificados (ns\|us\|ms\|s\|m\|h) |
| `-d`, `--detach` | false | Sai imediatamente sem esperar convergência |
| `--external-ca` | - | Especifica endpoints de assinatura externos |
| `-q`, `--quiet` | false | Suprime output de progresso |
| `--rotate` | false | Rotaciona a CA raiz |

## Descrição

Visualiza ou rotaciona o certificado CA raiz do swarm.

> **Nota**: Comando de gerenciamento de cluster, deve ser executado em nó manager.

## Exemplos

### Visualizar CA atual

```bash
docker swarm ca
```

### Rotacionar CA

```bash
# Rotacionar com novos certificados gerados
docker swarm ca --rotate

# Rotacionar com certificado e chave específicos
docker swarm ca --rotate --ca-cert /path/to/cert.pem --ca-key /path/to/key.pem
```

### Verificar status de rotação

```bash
docker node ls --format '{{.ID}} {{.Hostname}} {{.Status}} {{.TLSStatus}}'
```

## Quando usar

- **Rotacionar CA comprometida**: Se um ou mais managers foram comprometidos
- **Usar CA externa**: Migrar de CA interna para externa (ou vice-versa)
- **Renovação periódica**: Boas práticas de segurança para certificados

## Referência

[Documentação oficial](https://docs.docker.com/reference/cli/docker/swarm/ca/)
