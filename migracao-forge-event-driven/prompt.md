---
nome: migracao-forge-event-driven-elo-1-diagnostico
descricao: Diagnostico do estado atual do Forge antes da migracao para event-driven.
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

Voce vai produzir apenas o Elo 1 da cadeia de migracao do Forge.

Entrada:

```text
{{estado_forge}}
```

Objetivo do Elo 1:
- Mapear o fluxo atual do Forge, dependencias, gargalos, riscos e invariantes que nao podem quebrar.
- Nao propor a solucao ainda, alem das observacoes necessarias para sustentar a estrategia.
- Preparar a entrada real para o Elo 2, sem resumir demais nem misturar recomendacoes de migracao.

Regras:
- Mantenha Sentinel, Cerebro e billing funcionando durante a transicao, mas trate isso como restricao do contexto, nao como plano do Elo 1.
- Preserve a compatibilidade das tabelas particionadas por hora ate que os consumidores migrem.
- Nao invente ferramentas especificas se uma formulacao generica bastar.
- Se uma decisao depender de dado ausente, marque como premissa ou lacuna.

Formato de saida:

```text
ELO 1 - DIAGNOSTICO:
<diagnostico estruturado>

LACUNAS:
<dados ausentes que impactam a migracao>

ENTRADA PARA O ELO 2:
<texto que sera usado como input real do Elo 2>
```
