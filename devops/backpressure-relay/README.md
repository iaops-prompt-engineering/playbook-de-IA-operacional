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

## Metodo de criacao

- Prompt final parametrizavel, feito para receber estado, restricoes e alternativas no mesmo bloco.
- Curadoria feita sobre meta-prompting com foco em decisao comparativa.
- Escolha metodologica: matriz curta de alternativas mais recomendacao final combinada quando fizer sentido.

Justificativa:
- Este checkpoint nao pede "resposta certa", pede comparacao defensavel entre caminhos com custos diferentes.
- Sem contrato de comparacao, modelo tende a escolher uma solucao cedo demais e esconder trade-off.
- O contrato atual força restricoes inegociaveis, comparacao, descartes, rollout e metricas de sucesso.

## Casos de uso

- Incidentes de pico de telemetry.
- Planejamento de isolamento por tenant.
- Decisao entre prioridade de fila, autoscaling, buffering e segmentacao.

## Exemplo

Entrada: throughput sustentado, pico observado, consumidores e SLAs.

Saida: recomendacao, comparacao de alternativas, rollout, descartes e metricas de sucesso.

## Contrato de saida

- `DECISAO RECOMENDADA`: estrategia principal, podendo ser combinada.
- `RESTRICOES QUE GOVERNAM A DECISAO`: limites que nao podem ser violados.
- `COMPARACAO DE ALTERNATIVAS`: beneficios, risco/custo e adequacao.
- `ROLLOUT PROPOSTO`: ordem de implantacao.
- `DESCARTES`: o que nao fazer e por que.
- `METRICAS DE SUCESSO`: sinais objetivos para validar decisao.

## Modelo recomendado

- Modelo principal: GPT-4o mini para custo baixo em analise comparativa curta.
- Escalada opcional: GPT-4o quando houver mais alternativas, mais restricoes ou necessidade de texto executivo mais fino.

## Limitacoes

- O prompt nao calcula capacidade real de infraestrutura sem metricas adicionais.
- Requer revisao de engenharia antes de virar mudanca de arquitetura.
- Custos devem ser recalculados com precos reais do ambiente.
