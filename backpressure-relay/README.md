---
nome: backpressure-relay
descricao: Apoia decisao de estrategia de backpressure para o Relay comparando alternativas.
versao: 1.0.0
tags:
  - devops
  - architecture
  - sre
  - event-driven
inputs:
  - nome: cenario_relay
    descricao: Estado atual do Relay, restricoes de SLA, custo, consumidores e alternativas em avaliacao.
---

# backpressure-relay

## Objetivo

Apoiar decisoes de arquitetura sobre backpressure no barramento Relay com comparacao explicita de alternativas.

## Casos de uso

- Incidentes de pico de telemetry.
- Planejamento de isolamento por tenant.
- Decisao entre prioridade de fila, autoscaling, buffering e segmentacao.

## Exemplo

Entrada: throughput sustentado, pico observado, consumidores e SLAs.

Saida: recomendacao, comparacao de alternativas, rollout, descartes e metricas de sucesso.

## Limitacoes

- O prompt nao calcula capacidade real de infraestrutura sem metricas adicionais.
- Requer revisao de engenharia antes de virar mudanca de arquitetura.
- Custos devem ser recalculados com precos reais do ambiente.
