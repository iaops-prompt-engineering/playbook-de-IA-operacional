# elo-2-estrategia

## Objetivo

Receber a saida real do Elo 1 e transformar o diagnostico em estrategia incremental para a migracao do Forge.

Este elo existe para impedir duas falhas comuns: reexplicar o problema inteiro e pular direto para o plano operacional. Ele precisa manter o raciocinio intermediario visivel.

## Entrada

- `diagnostico_forge`: saida real do Elo 1.
- A entrada precisa carregar o diagnostico, as invariantes, as lacunas e a entrada preparada para o Elo 2.

## Saida

- estrategia incremental
- alternativas consideradas
- pontos de reversao
- premissas e lacunas que ainda afetam a decisao
- entrada preparada para o Elo 3

## Leitura Esperada

- O Elo 2 deve justificar por que a estrategia incremental e melhor que uma migracao big-bang.
- Deve deixar claro o que ainda permanece no batch e o que entra em convivio com o event-driven.
- Deve explicitar o que acontece se a reconciliacao falhar ou se houver divergencia de schema, ordering ou volume.

## Limitacoes

- Nao deve repetir o diagnostico inteiro.
- Depende da qualidade da saida do Elo 1.
- Nao deve virar checklist de execucao prematuro.
