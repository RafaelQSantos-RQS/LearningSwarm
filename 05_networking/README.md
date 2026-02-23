# 05 - Networking

## Objetivos

- Compreender os tipos de networks no Swarm
- Criar e gerenciar overlay networks
- Entender o service discovery interno
- Configurar ingress para tráfego externo

## Teoria

### Tipos de Networks no Swarm

| Tipo | Escopo | Uso |
|------|--------|-----|
| **overlay** | Multi-host | Comunicação entre serviços em diferentes nós |
| **ingress** | Swarm-wide | Load balancing do tráfego externo |
| **docker_gwbridge** | Local | Comunicação host ↔ containers |
| **host** | Local | Networking modo host (não recomendado) |

### Overlay Network

Rede que permite comunicação entre containers em diferentes hosts do swarm. O Swarm usa:
- **VXLAN**: Tunneling layer 2 sobre layer 3
- **Control plane**: Criptografado por padrão (AES-256)
- **Data plane**: Não criptografado por padrão (pode ativar com --opt encrypted)

### Ingress Network

Rede especial do Swarm que faz **load balancing** automático:
- Publica portas em todos os nós
- Redistribui tráfego para o nó que tem a task
- Usa IPVS (Linux kernel) para balanceamento

```
┌──────────────────────────────────────────────────────┐
│                    INGRESS NETWORK                   │
│                                                      │
│  Request ──► Node 1:80 ──► (IPVS) ──► Node 3:80     │
│                  (não tem task)       (tem task)    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### Service Discovery

No Swarm, cada serviço obtém um **nome DNS** que resolve para todos os IPs das tasks:

```bash
# Do container web, você pode acessar:
ping db          # resolve para IPs das tasks de db
ping api         # resolve para IPs das tasks de api

# DNS interno automática
```

### Redes no Compose vs Swarm

```yaml
# Compose (Docker Compose V1/V2)
services:
  web:
    networks:
      - frontend

# Swarm (mesma sintaxe, mas cria overlay)
services:
  web:
    networks:
      - frontend
```

## Prática

### Criando Networks

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
    # driver_opts:
    #   encrypted: "true"
  backend:
    driver: overlay
```

### Publicando Portas

```bash
# Forma 1: Porta explícita
docker service create -p 8080:80 nginx

# Forma 2: Porta aleatória ( Swarm atribui)
docker service create -p 80 nginx

# Forma 3: Publicação em todos os nós (ingress)
docker service create --publish mode=ingress,target=80,published=8080 nginx

# Modo host (não recomendado)
docker service create --publish mode=host,target=80 nginx
```

### VSF (Virtual IP) vs DNS-RR

```bash
# VIP (default): Requests distribuídas entre todas as tasks
docker service create --name vip-service nginx

# DNS Round Robin: DNS retorna todas as IPs diretamente
docker service create --name rr-service --endpoint-mode dnsrr nginx
```

## Exercícios

1. **Exercício 1**: Criar 2 serviços em networks diferentes e testar comunicação (deve falhar)
2. **Exercício 2**: Colocar os 2 serviços na mesma rede e testar ping por nome
3. **Exercício 3**: Criar uma stack com frontend, backend e redis, usando networks separadas
4. **Exercício 4**: Testar o ingress publicando em portas diferentes e verificar que funciona em qualquer nó

## Referências

- [Networking overview](https://docs.docker.com/network/)
- [Use overlay networks](https://docs.docker.com/network/overlay/)
- [Swarm service networking](https://docs.docker.com/engine/swarm/networking/)
