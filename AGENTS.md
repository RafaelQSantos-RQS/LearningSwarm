# AGENTS.md - Guia para Assistentes de IA

## Estrutura das Aulas

Cada lição segue o padrão `XX_topico`:
- `00_docs` - Documentação oficial e referência CLI
- `01_introducao` - Conceitos básicos e init
- `02_nodes_cluster` - Nodes, manager/worker, quorum
- `03_services` - Services, réplicas, tasks
- `04_stacks` - Stack deploy, compose no Swarm
- `05_networking` - Overlay networks, ingress, discovery
- `06_secrets_configs` - Dados sensíveis
- `07_updates_rollbacks` - Rolling updates, rollbacks
- `08_constraints_labels` - Placement de serviços
- `09_troubleshooting` - Logs, inspect, debug
- `10_migracao_projetos` - Do Compose para Swarm

## Estrutura de Cada Pasta

Cada pasta de aula DEVE conter:
- `README.md` - Roteiro da lição (teoria + exercícios)
- `codigo/` - Exemplos de configuração
- `exercicios/` - Desafios práticos

### Pasta de Referência (00_docs)

Contém documentação oficial do Docker Swarm organizada em:

**`cli/`** - Referência de comandos CLI
- Cada comando tem seu próprio arquivo com: uso, opções, exemplos, "quando usar"

**`conceitos/`** - Conceitos fundamentais
- key-concepts.md, services.md, swarm-mode.md

**`arquitetura/`** - Como o Swarm funciona por dentro
- how-swarm-mode-works/, nodes, services, pki, raft

**`tutoriais/`** - Tutoriais oficiais passo a passo

**`guia/`** - Guias de administração
- admin_guide, networking, secrets, configs, etc

**Todas as aulas DEVEM referenciar esses arquivos na seção de prática.**

## Convenções

- Exemplos em YAML para compose/swarm
- Commits: conventional commits (feat:, fix:, docs:, etc.)
- Código em português (comentários)
- Cada conceito deve ter exemplo executável

## Roteiro do README

Cada README deve seguir:
1. **Objetivos** - O que o aluno vai aprender
2. **Teoria** - Conceitos fundamentais com analogias ao Compose
3. **Prática** - Comandos e exemplos
   - Incluir tabela de comandos usados com colunas: Comando, Descrição, Referência
   - Adicionar notas "Quando usar" para cada comando/key feature
4. **Exercícios** - Desafios propostos
5. **Referências** - Links oficiais + referência aos arquivos em 00_docs

## Fluxo de Trabalho

1. Criar/atualizar pasta da aula
2. Escrever README com roteiro
3. Implementar exemplos de código
4. Criar exercícios
5. Commitar com prefixo `feat:`

## Analogias Compose vs Swarm

| Compose | Swarm |
|---------|-------|
| `docker-compose up` | `docker stack deploy` |
| Container | Task |
| Networks | Overlay Networks |
| .env | Secrets |
| depends_on | deploy/constraints |
