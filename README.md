# Aegis AI Operations Prompt Playbook

Colecao de prompts em Markdown organizada no formato do template `prompt-registry`. Cada prompt vive em sua propria pasta, com `prompt.md` parametrizavel e `README.md` documentado. Os placeholders `{{variavel}}` do prompt e o campo `inputs` do frontmatter sao a interface versionada do playbook.

## Como usar

1. Navegue ate a categoria de interesse.
2. Abra o `README.md` do prompt para entender objetivo, variaveis e limitacoes.
3. Copie o `prompt.md` e substitua os placeholders `{{nome_variavel}}`.

## Categorias

### DevOps

Prompts operacionais de Kubernetes, SRE, observabilidade, seguranca de rede e migracoes de plataforma.

- [triagem-de-pods](devops/triagem-de-pods/README.md)
- [nota-de-triagem](devops/nota-de-triagem/README.md)
- [causa-raiz](devops/causa-raiz/README.md)
- [backpressure-relay](devops/backpressure-relay/README.md)
- [migracao-forge-event-driven](devops/migracao-forge-event-driven/README.md)
- [networkpolicy-sentinel](devops/networkpolicy-sentinel/README.md)

## Avaliacao

Execute uma avaliacao individual:

```bash
npx promptfoo@latest eval -c devops/nota-de-triagem/promptfooconfig.yaml
```

Execute todos os configs:

```bash
npm run eval
```

Os configs usam limites operacionais de latencia e custo e combinam asserts determinísticos com LLM-as-judge nos prompts de saida aberta.

Para execucao local, exporte:

```bash
export OPENAI_API_KEY=...
export ANTHROPIC_API_KEY=...
```

## Entrega

Os outputs de execucao e curadoria dos checkpoints estao em `docs/executions/`.
