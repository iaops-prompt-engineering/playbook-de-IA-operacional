# Aegis AI Operations Prompt Playbook

Biblioteca de prompts parametrizáveis para operações da plataforma Aegis, organizada no formato do template `prompt-registry`.

## Estrutura

- `triagem-de-pods`: triagem de saúde de pods Kubernetes com snapshot colado no prompt.
- `nota-de-triagem`: padronização de notas de alerta do Sentinel.
- `causa-raiz`: análise de causa-raiz a partir de artefatos operacionais.
- `backpressure-relay`: apoio a decisão de estratégia de backpressure.
- `migracao-forge-event-driven`: cadeia de prompts para migração do Forge.
- `networkpolicy-sentinel`: geração e refino de NetworkPolicy endurecida.

Cada pasta contém:

- `prompt.md`: frontmatter e prompt parametrizável com placeholders `{{variavel}}`.
- `README.md`: documentação humana, casos de uso, exemplo e limitações.
- `promptfooconfig.yaml`: testes promptfoo quando aplicável.

## Avaliação

Execute uma avaliação individual:

```bash
npx promptfoo@latest eval -c nota-de-triagem/promptfooconfig.yaml
```

Execute todos os configs:

```bash
npm run eval
```

Os configs usam limites operacionais de latência e custo e combinam asserts determinísticos com LLM-as-judge nos prompts de saída aberta.

## Entrega

Os outputs de execução e curadoria dos checkpoints estão em `docs/executions/`.
