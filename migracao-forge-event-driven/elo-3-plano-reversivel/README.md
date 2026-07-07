# elo-3-plano-reversivel

## Objetivo

Receber a saida real do Elo 2 e converter a estrategia em um plano executavel e reversivel.

Este elo e o ponto em que a cadeia deixa de ser analitica e vira proposta de implementacao. A saida precisa ser suficientemente concreta para ser transformada em backlog, runbook ou sequencia de rollout.

## Entrada

- `estrategia_forge`: saida real do Elo 2.
- A entrada precisa carregar estrategia, alternativas, pontos de reversao e premissas.

## Saida

- passos operacionais
- fases ou marcos de rollout
- rollback
- riscos residuais
- criterios de sucesso
- resumo final pronto para execucao

## Leitura Esperada

- O plano deve deixar claro como iniciar em shadow mode, quando promover dual-run e quando liberar consumidores.
- O rollback precisa ser pratico, curto e coerente com a estrategia anterior.
- Os criterios de sucesso devem ser observaveis, nao abstratos.

## Limitacoes

- Depende da estrategia anterior estar consistente.
- Nao reabre a analise de causa nem reescreve o diagnostico.
- Nao deve esconder riscos residuais em linguagem vaga.
