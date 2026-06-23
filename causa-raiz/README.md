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

## Casos de uso

- Degradacao de busca, ingestao, alerting ou pipeline.
- Pre-analise antes de escalonar para engenharia de plataforma.
- Registro de incidente com evidencias cruzadas.

## Exemplo

Entrada: configuracao do sistema, tabela de metricas e trecho de logs cobrindo a janela do problema.

Saida: causa-raiz, cadeia de evidencias, consequencias, acoes, hipoteses descartadas, lacunas e sanitizacao.

## Limitacoes

- A conclusao depende da completude dos artefatos.
- O prompt nao executa comandos nem consulta sistemas externos.
- Em producao, nomes de tenants, hosts, namespaces internos e identificadores devem ser mascarados.
