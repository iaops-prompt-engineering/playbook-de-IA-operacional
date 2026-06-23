---
nome: nota-de-triagem
descricao: Converte alerta cru em nota padronizada de triagem operacional.
versao: 1.0.0
tags:
  - devops
  - incident-response
  - observability
  - sre
inputs:
  - nome: alerta_cru
    descricao: Texto bruto do alerta disparado pelo Sentinel ou por outro sistema da plataforma.
---

# nota-de-triagem

## Objetivo

Padronizar notas de triagem para que a passagem de turno tenha sempre os mesmos campos, tom e nivel de detalhe.

## Casos de uso

- Converter alertas crus em nota de plantao.
- Reduzir variacao entre plantonistas.
- Acelerar comunicacao entre SRE, produto e times de plataforma.

## Exemplo

Entrada: alerta cru com timestamp, sistema, sintoma e contexto.

Saida: cinco linhas com `ALERTA`, `IMPACTO`, `HIPOTESE INICIAL`, `ACAO IMEDIATA` e `ESCALAR PARA`.

## Limitacoes

- O prompt pode inferir impacto errado se o alerta bruto vier sem escopo.
- Handles devem ser revisados se a organizacao dos times mudar.
- Dados de tenant devem ser anonimizados quando enviados a modelos externos.
