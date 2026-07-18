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

## Metodo de criacao

- Prompt final parametrizavel, feito para receber alerta cru por parametro.
- Curadoria feita sobre meta-prompting com foco em contrato de saida fixo.
- Escolha metodologica: ensinar por contrato de formato e regras curtas, sem colar exemplos de entrada dentro do prompt final.

Justificativa:
- Este checkpoint tem dois conjuntos diferentes: referencia de formato e alertas crus.
- Colar exemplos completos no prompt final aumenta risco de o modelo misturar formato de referencia com conteudo da entrada.
- Contrato fixo com cinco rotulos resolve melhor: menos tokens, menos vazamento entre exemplos, mais estabilidade para teste.

## Casos de uso

- Converter alertas crus em nota de plantao.
- Reduzir variacao entre plantonistas.
- Acelerar comunicacao entre SRE, produto e times de plataforma.

## Exemplo

Entrada: alerta cru com timestamp, sistema, sintoma e contexto.

Saida: cinco linhas com `ALERTA`, `IMPACTO`, `HIPOTESE INICIAL`, `ACAO IMEDIATA` e `ESCALAR PARA`.

## Contrato de saida

- `ALERTA`: sistema e sintoma principal.
- `IMPACTO`: quem sofre e como.
- `HIPOTESE INICIAL`: causa provavel, nao repeticao do sintoma.
- `ACAO IMEDIATA`: proxima acao executavel pelo plantao.
- `ESCALAR PARA`: handle de time e gatilho de escalonamento.

## Modelo recomendado

- Modelo principal: GPT-4o mini para custo baixo e latencia adequada.
- Segundo provedor opcional: Claude Haiku para comparar consistencia de formatacao.

## Limitacoes

- O prompt pode inferir impacto errado se o alerta bruto vier sem escopo.
- Handles devem ser revisados se a organizacao dos times mudar.
- Dados de tenant devem ser anonimizados quando enviados a modelos externos.
