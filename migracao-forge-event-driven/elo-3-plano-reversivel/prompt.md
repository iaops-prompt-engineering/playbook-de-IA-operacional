---
nome: migracao-forge-event-driven-elo-3-plano-reversivel
descricao: Plano executavel e reversivel para migracao do Forge com base na estrategia anterior.
versao: 1.0.0
tags:
  - devops
  - data-platform
  - migration
  - event-driven
inputs:
  - nome: estrategia_forge
    descricao: Saida real do Elo 2 com a estrategia incremental de migracao.
---

Voce vai produzir apenas o Elo 3 da cadeia de migracao do Forge.

Entrada real do Elo 2:

```text
{{estrategia_forge}}
```

Objetivo do Elo 3:
- Transformar a estrategia em um plano operacional, executavel e reversivel.
- Incluir passos, validacoes, feature flags ou dual-run quando aplicavel.
- Registrar criterios de sucesso, rollback e riscos residuais.

Regras:
- Nao reabra a analise de causa; assuma o diagnostico e a estrategia como dados de entrada reais.
- O plano precisa ser compatível com Sentinel, Cerebro e billing em funcionamento durante a transicao.
- Preserve compatibilidade de tabelas particionadas por hora ate os consumidores migrarem.
- Se faltar algum dado, explicite a lacuna em vez de inventar uma decisao.

Formato de saida:

```text
ELO 3 - PLANO EXECUTAVEL:
<passos, validacoes e rollout>

ROLLBACK:
<como voltar para batch>

RISCOS RESIDUAIS:
<riscos que continuam presentes>

CRITERIOS DE SUCESSO:
<condicoes observaveis para considerar a migracao segura>
```
