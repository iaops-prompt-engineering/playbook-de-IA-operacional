# Leitura do repositório

**Caminho do repositório:** `/home/drolen/GitHub/iaops-prompt-engineering/playbook-de-IA-operacional`

## O que foi feito

O repositório organiza um **playbook de IA operacional** para a plataforma Aegis, no formato de biblioteca de prompts parametrizáveis.

A base do trabalho foi transformar cenários recorrentes de operação em artefatos reutilizáveis com:
- `prompt.md` com frontmatter e placeholders `{{variavel}}`
- `README.md` com objetivo, casos de uso, exemplo e limitações
- `promptfooconfig.yaml` para avaliação automatizada quando aplicável

## Estrutura identificada

Os cenários principais são:
- `triagem-de-pods`: triagem de saúde de pods Kubernetes a partir de snapshot colado no prompt
- `nota-de-triagem`: padronização de alerta bruto em nota operacional
- `causa-raiz`: análise de causa-raiz a partir de artefatos operacionais
- `backpressure-relay`: apoio à decisão de estratégia de backpressure
- `migracao-forge-event-driven`: cadeia de prompts para migração do Forge para event-driven
- `networkpolicy-sentinel`: geração e endurecimento de NetworkPolicy

## Padrão de construção

Os arquivos seguem um padrão consistente:
- versão inicial `1.0.0`
- frontmatter com nome, descrição, versão, tags e inputs
- foco em uso operacional real, com saída contratualizada
- limitações explícitas, especialmente sobre sanitização de dados sensíveis e dependência de artefatos colados no prompt

## O que os checkpoints mostram

Os arquivos em `docs/executions/` documentam a curadoria e a execução dos prompts.

Principais aprendizados registrados:
- `triagem-de-pods` foi curado para cruzar status, eventos e logs, evitando concluir só pelo status do pod
- `nota-de-triagem` foi moldado para produzir saída curta, padronizada e fácil de automatizar
- `causa-raiz` foi ajustado para separar causa de consequência e evitar confundir sintomas com raiz do problema
- `backpressure-relay` usa comparação explícita de alternativas e trade-offs
- `migracao-forge-event-driven` foi estruturado em cadeia de passos para permitir migração incremental e reversível
- `networkpolicy-sentinel` foi refinado com preocupação de segurança, default-deny e validação antes de produção

## Estratégia de validação

O repositório também formaliza uma estratégia de testes com `promptfoo`:
- asserts determinísticos para saídas fechadas e formatadas
- LLM-as-judge para respostas abertas, especialmente `causa-raiz`
- gates em CI para PR e push em `main`
- uso de matriz por config para isolar falhas por prompt

## Síntese do que aprendi

Este repositório não é apenas uma coleção de prompts. Ele é uma **biblioteca operacional com contrato**, construída para:
- padronizar respostas de IA em contextos de SRE e plataforma
- reduzir variação entre plantonistas
- apoiar decisão com evidências e trade-offs
- manter rastreabilidade entre prompt, avaliação e curadoria
- permitir evolução segura por meio de testes automatizados e gates de CI

## Observações práticas

- Os prompts dependem de entradas bem sanitizadas, principalmente em logs, nomes de tenant, hosts e namespaces internos
- Os arquivos de execução mostram que a qualidade esperada não é só textual, mas também operacional: causa, evidência, ação e limite de confiança
- O modelo de organização facilita reutilização e revisão por cenário, em vez de um prompt monolítico
