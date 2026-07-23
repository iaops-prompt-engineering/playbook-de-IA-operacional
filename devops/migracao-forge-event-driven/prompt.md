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

Papel do Elo 1 na cadeia:
- Ler o estado atual.
- Explicar por que a migracao e delicada.
- Preparar entrada concreta para o Elo 2.
- Nao pular para estrategia ou plano.

Entrada:

```text
{{estado_forge}}
```

Objetivo do Elo 1:
- Diagnosticar o estado atual do Forge e o problema central da migracao: a dependencia de um fluxo unico de batch que ainda precisa ser convertido em uma cadeia de prompts separada.
- Mapear o fluxo atual, dependencias, gargalos, invariantes, restricoes operacionais e lacunas de informacao.
- Explicitar o que precisa ser preservado durante a transicao e o que ainda nao pode ser decidido nesta etapa.
- Preparar uma entrada real e reutilizavel para o Elo 2, sem misturar estrategia ou plano executavel.

Regras:
- Nao proponha solucao, rollout ou rollout order.
- Mantenha Sentinel, Cerebro e billing funcionando durante a transicao, mas trate isso como restricao do contexto, nao como plano do Elo 1.
- Preserve a compatibilidade das tabelas particionadas por hora ate que os consumidores migrem.
- Diga explicitamente quais dependencias e contratos tornam a migracao delicada.
- Se uma decisao depender de dado ausente, marque como premissa ou lacuna.
- A saida precisa ser concreta o suficiente para ser colada como entrada do Elo 2 sem reinterpretracao.
- Nao use cercas de codigo, markdown extra ou texto fora do formato final.

Formato de saida:
ELO 1 - DIAGNOSTICO:
<diagnostico estruturado do estado atual, do acoplamento e das restricoes>

INVARIANTES:
<o que nao pode quebrar durante a transicao>

LACUNAS:
<dados ausentes que impactam a migracao>

CRITERIOS PARA O ELO 2:
<condicoes e focos que a estrategia precisa considerar>

ENTRADA PARA O ELO 2:
<texto que sera usado como input real do Elo 2>
