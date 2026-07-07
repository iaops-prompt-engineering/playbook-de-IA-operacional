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
- Usar o diagnostico real do Elo 1 como base, sem reabrir o problema do zero.
- Comparar abordagens de migracao e justificar a escolha de uma estrategia incremental.
- Propor convivencia entre batch e event-driven, com gates de validacao, reversao e migracao controlada de consumidores.
- Gerar uma entrada rica para o Elo 3, com decisoes e premissas claramente registradas.

Regras:
- Nao repita o diagnostico inteiro; sintetize apenas o que fundamenta a estrategia.
- Compare pelo menos duas opcoes relevantes quando isso ajudar a explicar a escolha.
- Mantenha Sentinel, Cerebro e billing funcionando durante a transicao.
- Preserve a compatibilidade das tabelas particionadas por hora ate a migracao dos consumidores.
- Deixe claros os pontos de reversao, dependencias e criterios de avancar para o Elo 3.
- Se uma decisao depender de dado ausente, marque como premissa ou lacuna.
- A saida precisa ser imediatamente util como entrada operacional do proximo elo.

Formato de saida:

```text
ELO 2 - ESTRATEGIA:
<estrategia incremental detalhada, com justificativa>

ALTERNATIVAS AVALIADAS:
<opcoes consideradas e porque foram aceitas ou rejeitadas>

PONTOS DE REVERSAO:
<onde voltar para batch sem perda de consistencia>

PREMISAS E LACUNAS:
<dados ainda faltantes que afetam a estrategia>

ENTRADA PARA O ELO 3:
<texto que sera usado como input real do Elo 3>
```
