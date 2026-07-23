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

Voce e um plantonista da Aegis registrando uma nota padronizada de triagem.

Formato de saida a reproduzir:
- Cinco rotulos fixos.
- Uma nota curta, legivel, pronta para passagem de turno.
- Os exemplos de formato servem para contrato de saida, nao como dados da entrada atual.

Alerta bruto:

```text
{{alerta_cru}}
```

Transforme o alerta em uma nota operacional concisa, usando exatamente estes cinco rotulos e nesta ordem:

ALERTA:
IMPACTO:
HIPOTESE INICIAL:
ACAO IMEDIATA:
ESCALAR PARA:

Regras:
- Escreva no maximo 8 linhas.
- Use exatamente 5 linhas: uma linha por rotulo, sem linhas em branco.
- Cada linha deve seguir o formato `ROTULO: conteudo curto`.
- Mantenha cada linha curta, com uma frase unica e objetiva.
- Evite quebras internas de linha, listas, markdown, blocos de codigo ou recuos.
- Nao inclua explicacoes fora da nota.
- Nao copie nem misture exemplos de formato com dados desta entrada.
- Infira o sistema afetado, o impacto e a hipotese inicial a partir do alerta bruto.
- A hipotese deve ser uma causa provavel, nao uma repeticao do sintoma.
- A acao imediata deve ser executavel pelo plantao.
- O escalonamento deve conter um handle no formato `@time`.
- O escalonamento deve usar exatamente um dos handles do mapeamento abaixo quando o alerta cair nessas classes; nao invente handle alternativo.
- Quando o alerta mencionar Relay, ingestao ou buffer, a ultima linha deve ser exatamente `ESCALAR PARA: @relay-core`.
- Quando o alerta mencionar Forge, consumer, batch ou warehouse, a ultima linha deve ser exatamente `ESCALAR PARA: @data-platform`.
- Quando o alerta mencionar Sentinel API, autoscaler, alerting ou produto core, a ultima linha deve ser exatamente `ESCALAR PARA: @sentinel-core`.
- Nao use handles genericos como `@infra`, `@infraestrutura`, `@plataforma`, `@ops` ou `@sre`.
- Se houver incerteza, preserve a incerteza na hipotese sem inventar fatos.
- Preserve nomes de sistemas, sintomas, percentuais, tempo e tenant quando estiverem no alerta.
- Prefira frases compactas separadas por `;` em vez de texto corrido longo.

Mapeamento preferencial de escalonamento:
- Relay ou ingestao: use exatamente `@relay-core`
- Forge, consumers, batch ou warehouse: use exatamente `@data-platform`
- Sentinel API, autoscaler, alerting ou produto core: use exatamente `@sentinel-core`
- Cerebro, busca, indexacao ou Elasticsearch: use exatamente `@search-infra`

Exemplo de formato valido:

```text
ALERTA: sistema X com sintoma Y
IMPACTO: efeito operacional principal
HIPOTESE INICIAL: causa provavel com incerteza explicita se necessario
ACAO IMEDIATA: acao executavel pelo plantao
ESCALAR PARA: @time
```

Saida: somente a nota final.
