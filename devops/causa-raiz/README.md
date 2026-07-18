---
nome: causa-raiz
descricao: Analise de causa-raiz a partir de configuracao, metricas e logs operacionais.
versao: 1.0.0
tags:
  - devops
  - sre
  - root-cause-analysis
  - observability
inputs:
  - nome: sistema
    descricao: Nome do sistema analisado.
  - nome: artefatos
    descricao: Pacote com configuracao, metricas, logs e contexto operacional colados no prompt.
---

# causa-raiz

## Objetivo

Levar o modelo a raciocinar sobre artefatos operacionais e apontar causa-raiz, sem confundir sintomas com causa.

## Metodo de criacao

- Prompt final parametrizavel, feito para receber um pacote unico de artefatos.
- Curadoria feita sobre meta-prompting com foco em cadeia causal.
- Escolha metodologica: estruturar a resposta por evidencias, consequencias, hipoteses descartadas e lacunas.

Justificativa:
- Este checkpoint mistura configuracao, metricas e logs de fontes diferentes.
- Sem um roteiro explicito, modelo tende a listar sintomas ou resumir cada artefato separado.
- O contrato atual força correlacao temporal e separa causa, mecanismo intermediario e efeito.

## Casos de uso

- Degradacao de busca, ingestao, alerting ou pipeline.
- Pre-analise antes de escalonar para engenharia de plataforma.
- Registro de incidente com evidencias cruzadas.

## Exemplo

Entrada: configuracao do sistema, tabela de metricas e trecho de logs cobrindo a janela do problema.

Saida: causa-raiz, cadeia de evidencias, consequencias, acoes, hipoteses descartadas, lacunas e sanitizacao.

## Contrato de saida

- `CAUSA-RAIZ`: explicacao principal em 1-2 frases.
- `CADEIA DE EVIDENCIAS`: ordem cronologica e sustentacao da causa.
- `SINTOMAS E CONSEQUENCIAS`: separar efeito de causa.
- `ACOES RECOMENDADAS`: imediata, curto prazo e prevencao.
- `HIPOTESES MENOS PROVAVEIS`: descartar alternativas com base nos artefatos.
- `LACUNAS E INCERTEZAS`: o que falta para confirmar ou quantificar.
- `SANITIZACAO ANTES DE MODELO EXTERNO`: o que mascarar antes de enviar a provedor externo.

## Modelo recomendado

- Modelo principal: GPT-4o mini para custo baixo com raciocinio suficiente neste pacote.
- Escalada opcional: GPT-4o quando artefatos ficarem maiores ou mais ambiguos.

## Limitacoes

- A conclusao depende da completude dos artefatos.
- O prompt nao executa comandos nem consulta sistemas externos.
- Em producao, nomes de tenants, hosts, namespaces internos e identificadores devem ser mascarados.
