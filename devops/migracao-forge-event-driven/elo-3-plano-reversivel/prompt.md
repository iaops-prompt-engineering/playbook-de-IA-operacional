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

Papel do Elo 3 na cadeia:
- Receber estrategia ja escolhida.
- Converter em fases, gates e rollback concreto.
- Manter batch e consumidores vivos durante transicao.
- Entregar resumo final pronto para backlog ou runbook.

Entrada real do Elo 2:

```text
{{estrategia_forge}}
```

Objetivo do Elo 3:
- Transformar a estrategia em um plano operacional, executavel e reversivel.
- Estruturar o rollout em etapas numeradas com validacoes, portas de decisao e criterios de passagem.
- Incluir shadow mode, dual-run, feature flags ou reconcilicao apenas quando fizer sentido para a estrategia recebida.
- Registrar rollback, riscos residuais, criterios de sucesso e o que precisa acontecer para a migracao ser considerada segura.

Regras:
- Nao reabra a analise de causa; assuma o diagnostico e a estrategia como dados de entrada reais.
- O plano precisa ser compatível com Sentinel, Cerebro e billing em funcionamento durante a transicao.
- Preserve compatibilidade de tabelas particionadas por hora ate os consumidores migrarem.
- Nao esconda riscos operacionais, dependencias ou pontos de corte.
- Nao transforme rollback em nota vaga; descreva acao pratica de reversao.
- Se faltar algum dado, explicite a lacuna em vez de inventar uma decisao.
- A saida precisa ser detalhada o bastante para orientar execucao tecnica e acompanhamento de rollout.

Formato de saida:

```text
ELO 3 - PLANO EXECUTAVEL:
<passos operacionais, validacoes e rollout>

FASES:
<quebrar o plano em fases claras, com objetivo e criterio de saida>

ROLLBACK:
<como voltar para batch>

RISCOS RESIDUAIS:
<riscos que continuam presentes>

CRITERIOS DE SUCESSO:
<condicoes observaveis para considerar a migracao segura>

ENTRADA FINAL PARA EXECUCAO:
<resumo curto do plano pronto para virar backlog ou runbook>
```
