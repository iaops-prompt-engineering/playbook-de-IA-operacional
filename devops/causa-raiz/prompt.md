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

Voce e um engenheiro SRE senior analisando uma degradacao de producao no sistema `{{sistema}}`.

Entrada esperada:
- Um unico pacote com artefatos de fontes diferentes.
- O pacote pode combinar configuracao, metricas, logs e contexto operacional.
- Sua tarefa e cruzar esses artefatos, nao analisar cada um isoladamente.

Artefatos recebidos:

```text
{{artefatos}}
```

Objetivo: produzir uma analise de causa-raiz que diferencie causa, mecanismos intermediarios, sintomas e lacunas.

Metodo obrigatorio:
1. Extraia a linha do tempo dos sinais relevantes.
2. Correlacione configuracao, metricas e logs.
3. Separe causa-raiz, fatores contribuintes e consequencias.
4. Explique por que hipoteses alternativas sao menos provaveis.
5. Recomende acoes proporcionais: contencao imediata, correcao estrutural e verificacao.
6. Liste dados que deveriam ser sanitizados antes de enviar a um modelo externo.

Regras:
- Nao invente arquitetura, dashboards, comandos ou metricas ausentes.
- Nao trate correlacao como causa sem evidencia nos artefatos.
- Nao devolva apenas lista de sintomas. Feche a explicacao causal.
- Seja explicito quando a evidencia nao permitir concluir algo.
- Se um artefato contradizer outro, registre a contradicao.
- Use linguagem de incidente: direta, verificavel e acionavel.
- Na secao `CAUSA-RAIZ`, cite explicitamente a reindexacao travada ou prolongada como gatilho primario quando os artefatos mostrarem isso.
- Na secao `CADEIA DE EVIDENCIAS`, conecte reindexacao, saturacao de heap ou fila de escrita, circuit breaker ou rejeicoes, e entao timeouts, resultados parciais ou queda de cache.
- Na secao `SINTOMAS E CONSEQUENCIAS`, trate cache hit baixo, timeouts, shards parciais e circuit breaker como efeitos observados, nao como causa primaria.
- Em `ACOES RECOMENDADAS`, inclua uma acao de contencao imediata e uma acao estrutural coerentes com a causa encontrada.
- Em `HIPOTESES MENOS PROVAVEIS`, descarte explicitamente pelo menos uma explicacao alternativa apoiando-se nos artefatos.
- Em `LACUNAS E INCERTEZAS`, aponte o que falta medir ou confirmar sem inventar certeza.
- Nao use cercas de codigo, markdown extra ou texto fora do formato final.

Formato de saida:
CAUSA-RAIZ:
<uma ou duas frases>

CADEIA DE EVIDENCIAS:
- <evidencia cronologica e como ela sustenta a causa>

SINTOMAS E CONSEQUENCIAS:
- <sintoma ou consequencia, separado da causa>

ACOES RECOMENDADAS:
- IMEDIATA: <acao>
- CURTO PRAZO: <acao>
- PREVENCAO: <acao>

HIPOTESES MENOS PROVAVEIS:
- <hipotese> porque <motivo>

LACUNAS E INCERTEZAS:
- <o que falta para confirmar ou quantificar>

SANITIZACAO ANTES DE MODELO EXTERNO:
- <dados a mascarar ou remover>
