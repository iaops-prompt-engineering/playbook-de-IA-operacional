# migracao-forge-event-driven

## Objetivo

Planejar a migracao incremental do Forge de batch para event-driven usando encadeamento de prompts separados.

## Casos de uso

- Migracoes grandes demais para um prompt monolitico.
- Planejamento com convivencia temporaria entre arquitetura antiga e nova.
- Avaliacao de rollback, dual-run e compatibilidade de consumidores.

## Cadeia

1. `prompt.md` executa apenas o Elo 1, com diagnostico do estado atual.
2. `elo-2-estrategia/prompt.md` recebe a saida do Elo 1 e define a estrategia incremental.
3. `elo-3-plano-reversivel/prompt.md` recebe a saida do Elo 2 e fecha o plano executavel e reversivel.

Cada elo deve ser executado com a saida do anterior colada como entrada real, sem reescrever a cadeia dentro de uma unica chamada.

## Exemplo

Entrada do Elo 1: descricao do batch atual, dependencias, fragilidades e garantias exigidas.

Saida: diagnostico, estrategia incremental e plano executavel reversivel, cada um em sua propria etapa.

## Limitacoes

- O plano precisa ser convertido em backlog tecnico e validado com engenharia de dados.
- A ausencia de metricas reais limita estimativas de capacidade.
- O prompt nao substitui teste de carga nem prova de compatibilidade de schema.
- A cadeia exige passagem manual da saida de um elo para a entrada do seguinte.
