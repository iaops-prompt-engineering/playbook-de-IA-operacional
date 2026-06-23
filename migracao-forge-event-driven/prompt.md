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

Voce vai conduzir uma cadeia de prompts para planejar a migracao do Forge de batch para processamento orientado a eventos.

Entrada inicial:

```text
{{estado_forge}}
```

Execute internamente tres elos e apresente a saida consolidada por elo. Cada elo deve usar a saida do elo anterior como entrada conceitual.

ELO 1 - Diagnostico do estado atual:
- Mapear fluxo atual, dependencias, gargalos, riscos e invariantes que nao podem quebrar.
- Nao propor solucao ainda alem de observacoes necessarias.

ELO 2 - Estrategia de migracao:
- Usar o diagnostico do Elo 1.
- Propor etapas incrementais, sem big-bang, com convivencia entre batch e event-driven.
- Explicitar pontos de reversao.

ELO 3 - Plano executavel e reversivel:
- Usar a estrategia do Elo 2.
- Transformar em plano operacional com passos, validacoes, feature flags ou dual-run quando aplicavel.
- Incluir criterios de sucesso, rollback e riscos residuais.

Regras:
- Mantenha Sentinel, Cerebro e billing funcionando durante a transicao.
- Preserve compatibilidade das tabelas particionadas por hora ate que consumidores migrem.
- Nao invente ferramentas especificas nao citadas se uma formulacao generica bastar.
- Se uma decisao depender de dado ausente, marque como premissa ou lacuna.

Formato de saida:

```text
ELO 1 - DIAGNOSTICO:
<diagnostico estruturado>

ELO 2 - ESTRATEGIA:
<etapas incrementais e reversiveis>

ELO 3 - PLANO EXECUTAVEL:
<passos, validacoes, rollback e riscos>

CURADORIA DE CADEIA:
<por que a decomposicao evita resposta rasa>
```
