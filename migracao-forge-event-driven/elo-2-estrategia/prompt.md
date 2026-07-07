---
nome: migracao-forge-event-driven-elo-2-estrategia
descricao: Estrategia incremental para migrar o Forge de batch para event-driven.
versao: 1.0.0
tags:
  - devops
  - data-platform
  - migration
  - event-driven
inputs:
  - nome: diagnostico_forge
    descricao: Saida real do Elo 1 com o diagnostico do estado atual do Forge.
---

Voce vai produzir apenas o Elo 2 da cadeia de migracao do Forge.

Entrada real do Elo 1:

```text
{{diagnostico_forge}}
```

Objetivo do Elo 2:
- Usar o diagnostico do Elo 1 como base real para a estrategia.
- Propor uma migracao incremental, sem big-bang, com convivencia entre batch e event-driven.
- Explicitar pontos de reversao e dependencias de validacao antes de seguir para o plano.

Regras:
- Nao repita o diagnostico inteiro; extraia apenas os pontos que sustentam a decisao.
- Mantenha Sentinel, Cerebro e billing funcionando durante a transicao.
- Preserve a compatibilidade das tabelas particionadas por hora ate a migracao dos consumidores.
- Se uma decisao depender de dado ausente, marque como premissa ou lacuna.

Formato de saida:

```text
ELO 2 - ESTRATEGIA:
<etapas incrementais e reversiveis>

PONTOS DE REVERSAO:
<onde voltar para batch sem perda de consistencia>

ENTRADA PARA O ELO 3:
<texto que sera usado como input real do Elo 3>
```
