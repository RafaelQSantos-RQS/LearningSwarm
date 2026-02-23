# 05 - Networking

## Objetivos

- Compreender os tipos de networks no Swarm
- Criar e gerenciar overlay networks
- Entender o service discovery interno
- Configurar ingress para tráfego externo

## Teoria

### Tipos de Networks no Swarm

O Docker Swarm gera dois tipos de tráfego:

1. **Control and Management Plane**: Mensagens de gerenciamento (join, leave, etc). Sempre criptografado.
2. **Application Data Plane**: Tráfego dos containers e clientes.

### Redes Principais

| Tipo | Escopo | Uso |
|------|--------|-----|
| **ingress** | Swarm-wide | Load balancing do tráfego externo (routing mesh) |
| **docker_gwbridge** | Local | Bridge host ↔ containers |
| **overlay** | Multi-host | Comunicação entre serviços em diferentes nós |
| **host** | Local | Modo host (não recomendado) |

### Como funciona o Routing Mesh

Quando um nó recebe uma requisição na porta publicada:
1. O nó repassa para o módulo **IPVS** (Linux kernel)
2. O IPVS mantém registro de todos os IPs das tasks
3. Seleciona um IP e roteia a requisição

```
┌─────────────────────────────────────────────────────┐
│                    INGRESS NETWORK                   │
│                                                      │
│  Request ──► Node 1:8080 ──► (IPVS) ──► Node 3   │
│                  (não tem task)        (tem task)   │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Service Discovery

No Swarm, cada serviço obtém um **nome DNS** que resolve para todos os IPs das tasks:

```bash
# Do container web, você pode acessar:
ping db          # resolve para IPs das tasks de db
ping api         # resolve para IPs das tasks de api
```

### Firewall

Para o Swarm funcionar, os nós precisam se comunicar:
- **Port 7946** TCP/UDP: Para descoberta de rede dos containers
- **Port 4789** UDP: Para dados do overlay network (pode ser alterado)

## Prática

### Criando Networks

> **Quando usar `docker network create`**: Para criar uma rede overlay customizada

```bash
# Criar overlay network (criptografada)
docker network create \
  --driver overlay \
  --opt encrypted \
  minha-rede

# Criar network com subnet específica
docker network create \
  --driver overlay \
  --subnet 10.10.0.0/16 \
  minha-rede

# Listar networks
docker network ls

# Inspect network
docker network inspect minha-rede
```

### Networks em Stack

```yaml
version: "3.8"

services:
  web:
    image: nginx
    networks:
      - frontend

  api:
    image: myapi
    networks:
      - frontend
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    driver: overlay
  backend:
    driver: overlay
```

### Publicando Portas

> **Quando usar `-p` ou `--publish`**: Para expor portas

```bash
# Forma 1: Porta explícita (routing mesh)
docker service create -p 8080:80 nginx

# Forma 2: Porta aleatória (Swarm atribui)
docker service create -p 80 nginx

# Forma 3: Publicação em todos os nós (ingress)
docker service create --publish mode=ingress,target=80,published=8080 nginx

# Modo host (não recomendado)
docker service create --publish mode=host,target=80 nginx
```

### DNS e Service Discovery

```bash
# Criar serviços em uma network
docker network create minha-rede

docker service create --network minha-rede --name db postgres
docker service create --network minha-rede --name api myapi

# De dentro do container api, você pode acessar db pelo nome
# docker exec -it <api-container> ping db
```

## Exercícios

1. **Exercício 1**: Criar 2 serviços em networks diferentes e testar comunicação (deve falhar)
2. **Exercício 2**: Colocar os 2 serviços na mesma rede e testar ping por nome
3. **Exercício 3**: Criar uma stack com frontend, backend e redis, usando networks separadas
4. **Exercício 4**: Testar o ingress publicando em portas diferentes e verificar que funciona em qualquer nó

## Referências

### Documentação Oficial
- [networking](../00_docs/guia/networking.md) - Redes no Swarm
- [ingress](../00_docs/guia/ingress.md) - Routing mesh

### Comandos CLI
- [docker network create]()
- [docker network ls]()
- [docker network inspect]()
