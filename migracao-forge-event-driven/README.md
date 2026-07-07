# migracao-forge-event-driven

## Objetivo

Planejar a migracao incremental do Forge de batch para event-driven usando uma cadeia de prompts separados.

O foco nao e responder tudo em uma unica chamada. O objetivo e obrigar a analise a evoluir por etapas, com cada saida servindo como entrada real da etapa seguinte.

## Casos de uso

- Migracoes grandes demais para um prompt monolitico.
- Planejamento com convivencia temporaria entre arquitetura antiga e nova.
- Avaliacao de rollback, dual-run e compatibilidade de consumidores.
- Cenarios em que o diagnostico, a estrategia e o plano operacional precisam ser auditaveis separadamente.

## Cadeia

1. `prompt.md` executa apenas o Elo 1 e diagnostica o estado atual do Forge.
2. `elo-2-estrategia/prompt.md` recebe a saida do Elo 1 e transforma o diagnostico em estrategia incremental.
3. `elo-3-plano-reversivel/prompt.md` recebe a saida do Elo 2 e fecha o plano executavel, reversivel e acompanhavel.

Cada elo deve ser executado com a saida do anterior colada como entrada real, sem reescrever a cadeia dentro de uma unica chamada.

## Contrato de saida

- O Elo 1 precisa registrar diagnostico, invariantes, lacunas e a entrada exata para o Elo 2.
- O Elo 2 precisa registrar estrategia, alternativas consideradas, pontos de reversao, premissas e a entrada exata para o Elo 3.
- O Elo 3 precisa registrar plano, fases, rollback, riscos residuais, criterios de sucesso e um resumo final pronto para execucao.

## O que melhorar

- Se o diagnostico ficar abstrato demais, o Elo 2 tende a herdar pouca substancia.
- Se a estrategia nao explicitar alternativas, o Elo 3 vira uma sequencia de passos sem criterio de escolha.
- Se o plano nao separar fases e gates, perde-se o valor de reversibilidade exigido pelo exercicio.

## Exemplo

Entrada do Elo 1: descricao do batch atual, dependencias, fragilidades e garantias exigidas.

Saida esperada: diagnostico com lacunas e invariantes, estrategia incremental com reversao clara e plano executavel com rollout, rollback e criterios de sucesso.

## Limitacoes

- O plano precisa ser convertido em backlog tecnico e validado com engenharia de dados.
- A ausencia de metricas reais limita estimativas de capacidade e risco.
- O prompt nao substitui teste de carga, prova de compatibilidade de schema nem validacao com consumidores reais.
- A cadeia exige passagem manual da saida de um elo para a entrada do seguinte.
