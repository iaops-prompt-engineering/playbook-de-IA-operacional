---
nome: migracao-forge-event-driven
descricao: Cadeia de prompts para migrar o Forge de batch para processamento orientado a eventos.
versao: 1.0.0
tags:
  - devops
  - data-platform
  - migration
  - event-driven
inputs:
  - nome: estado_forge
    descricao: Estado atual do Forge, dependencias, fragilidades e garantias exigidas para a migracao.
---

# migracao-forge-event-driven

## Objetivo

Planejar a migracao incremental do Forge de batch para event-driven usando encadeamento de prompts.

## Casos de uso

- Migracoes grandes demais para um prompt monolitico.
- Planejamento com convivencia temporaria entre arquitetura antiga e nova.
- Avaliacao de rollback, dual-run e compatibilidade de consumidores.

## Exemplo

Entrada: descricao do batch atual, dependencias, fragilidades e garantias exigidas.

Saida: diagnostico, estrategia incremental e plano executavel reversivel.

## Limitacoes

- O plano precisa ser convertido em backlog tecnico e validado com engenharia de dados.
- A ausencia de metricas reais limita estimativas de capacidade.
- O prompt nao substitui teste de carga nem prova de compatibilidade de schema.
