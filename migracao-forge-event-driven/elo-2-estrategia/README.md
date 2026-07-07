# elo-2-estrategia

## Objetivo

Receber a saida real do Elo 1 e transformar o diagnostico em estrategia incremental para a migracao do Forge.

## Entrada

- `diagnostico_forge`: saida real do Elo 1.

## Saida

- estrategia incremental
- pontos de reversao
- entrada preparada para o Elo 3

## Limitacoes

- Nao deve repetir o diagnostico inteiro.
- Depende da qualidade da saida do Elo 1.
