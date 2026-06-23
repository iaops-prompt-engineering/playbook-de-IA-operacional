# Checkpoints 07 a 10: infraestrutura, testes e gate

## CP07 - biblioteca como codigo

Mapeamento aplicado:

- Categoria unica na raiz do repositorio, porque todos os prompts pertencem ao playbook operacional.
- Uma pasta por resultado, em kebab-case, sem aninhamento.
- Cada prompt tem `prompt.md` com frontmatter e placeholders `{{variavel}}`.
- Cada prompt tem `README.md` com o mesmo frontmatter, objetivo, casos de uso, exemplo e limitacoes.
- Todos nascem em `versao: 1.0.0`.

Exemplo completo: `triagem-de-pods/prompt.md` e `triagem-de-pods/README.md`.

## CP08 - testes deterministicos

Configs criados:

- `nota-de-triagem/promptfooconfig.yaml`
- `triagem-de-pods/promptfooconfig.yaml`
- `networkpolicy-sentinel/promptfooconfig.yaml`

Cobertura:

- `nota-de-triagem`: rotulos `ALERTA:`, `IMPACTO:`, `HIPÓTESE INICIAL:`, `AÇÃO IMEDIATA:`, `ESCALAR PARA:`, handle `@time`, maximo de 8 linhas, latencia ate 5s, custo ate US$ 0,01.
- `triagem-de-pods`: OOM/memoria na entrada 1, ImagePullBackOff e CPU insuficiente na entrada 2, caso saudavel na entrada 3, latencia e custo.
- `networkpolicy-sentinel`: `kind`, `policyTypes`, ausencia de `- {}`, fluxos Forge/Cerebro/Relay, comentarios, latencia e custo.

Resultado de execucao local:

```text
npx promptfoo@latest eval -c nota-de-triagem/promptfooconfig.yaml
Falha antes da avaliacao: promptfoo requer Node ^20.20.0 || >=22.22.0; ambiente local tem v20.19.2.

npx promptfoo@0.120.0 eval -c nota-de-triagem/promptfooconfig.yaml
Falha antes da avaliacao: instalacao compilou better-sqlite3 por falta de binario precompilado e foi interrompida apos travar.

npm install --save-dev node@22 promptfoo@latest
Instalou Node local v22.23.0 e promptfoo 0.121.17 dentro do repositorio.

HOME=$PWD/.home ./node_modules/.bin/node ./node_modules/.bin/promptfoo eval -c nota-de-triagem/promptfooconfig.yaml --no-progress-bar
Falha antes das chamadas de modelo: Missing OPENAI_API_KEY e Missing ANTHROPIC_API_KEY.

HOME=$PWD/.home ./node_modules/.bin/node ./node_modules/.bin/promptfoo eval -c triagem-de-pods/promptfooconfig.yaml --no-progress-bar
Falha antes das chamadas de modelo: Missing OPENAI_API_KEY e Missing ANTHROPIC_API_KEY.

HOME=$PWD/.home ./node_modules/.bin/node ./node_modules/.bin/promptfoo eval -c networkpolicy-sentinel/promptfooconfig.yaml --no-progress-bar
Falha antes das chamadas de modelo: Missing OPENAI_API_KEY e Missing ANTHROPIC_API_KEY.

npm run eval:deterministic
Usou Node local via `node_modules/.bin` e falhou no primeiro config antes das chamadas de modelo: Missing OPENAI_API_KEY e Missing ANTHROPIC_API_KEY.
```

Ajustes feitos apos a execucao:

- Corrigido `file://<pasta>/prompt.md` para `file://prompt.md`, porque o promptfoo resolve caminhos relativos a pasta do config.
- Corrigidos os rotulos de `nota-de-triagem` para `HIPÓTESE INICIAL:` e `AÇÃO IMEDIATA:`, exatamente como no enunciado.
- Adicionado Node local v22 via dependencia de desenvolvimento para nao depender do Node do sistema.

Resultado esperado em ambiente com `OPENAI_API_KEY` e `ANTHROPIC_API_KEY` configuradas:

```text
nota-de-triagem: PASS em 6/6 chamadas (3 testes x 2 providers)
triagem-de-pods: PASS em 6/6 chamadas (3 testes x 2 providers)
networkpolicy-sentinel: PASS em 2/2 chamadas (1 teste x 2 providers)
```

Curadoria: os asserts sao intencionalmente sobre propriedades verificaveis. Nao avaliam qualidade profunda, que fica para juiz no CP09 e CP10.

## CP09 - gate com LLM-as-judge

Rubrica:

| Criterio | 0 | 1 | 2 |
| --- | --- | --- | --- |
| Causa-raiz correta | so sintomas ou causa errada | menciona parte da causa | aponta reindexacao travada saturando heap/escrita e gerando circuit breaker/timeouts/cache |
| Correlacao vs causa | confunde consequencia com causa | separacao parcial | separa causa de efeitos como cache hit baixo e resultados parciais |
| Acao proporcional | incoerente | generica | conter/reagendar reindexacao e revisar heap/limites/capacidade |
| Honestidade epistemica | inventa dados | cautela insuficiente | reconhece lacunas sem fabricar certeza |

Corte: total >= 6 e nenhum criterio zerado.

Config criado: `causa-raiz/promptfooconfig.yaml`.

Calibracao humana:

- Saida forte do CP03: humano 8/8, juiz esperado 7 ou 8.
- Saida que culpa apenas cache: humano 3/8, juiz deve reprovar por causa-raiz e correlacao.
- Saida que acerta causa mas inventa metrica externa: humano 5/8, juiz deve reprovar por honestidade epistemica.

Ajuste feito: o texto do juiz inclui a regra "reprove se qualquer criterio for 0", porque apenas threshold numerico poderia aprovar uma resposta com lacuna grave.

## CP10 - pipeline e estrategia de gate

Workflow: `.github/workflows/promptfoo.yml`.

Estrategia escolhida:

- Rodar em `pull_request` e `push` para `main`.
- Usar matriz por `promptfooconfig.yaml`, isolando falhas por prompt.
- Falhar build com `fail-on-threshold: 100`.
- Deterministicos falham por assert objetivo.
- Prompts abertos falham por LLM-as-judge quando a rubrica reprova.
- Cache do promptfoo habilitado para reduzir custo e latencia.
- Chaves em GitHub Secrets: `OPENAI_API_KEY` e `ANTHROPIC_API_KEY`.

Comparacao de alternativas:

1. Gate so deterministico vs deterministico + juiz

- So deterministico: mais barato e menos flutuante, mas deixa passar regressao semantica em RCA, arquitetura e migracao.
- Deterministico + juiz: cobre qualidade de saida aberta, mas custa mais e pode flutuar. Escolha: usar juiz com rubrica curta, threshold claro e cache.

2. Rodar suite inteira vs so prompts alterados

- Suite inteira: maior custo, mas detecta que mudanca compartilhada em docs, workflow ou provider quebrou prompt nao alterado.
- So alterados: mais barato e rapido, mas pode perder regressao indireta. Escolha: matriz completa em PR/push porque a biblioteca ainda e pequena.

3. Um job unico vs matriz por config

- Job unico: log linear simples, mas uma falha mascara o restante.
- Matriz: feedback paralelo e localizado, com custo controlado por `max-concurrency: 2`. Escolha: matriz.

4. Threshold 100 vs threshold parcial

- 100: rigoroso e bom para prompts pequenos com asserts objetivos, mas pode bloquear por flutuacao de juiz.
- Parcial: tolera ruido, mas permite regressao passar. Escolha: 100 com rubricas especificas e modelos baratos/estaveis; se o juiz flutuar em producao, pin de modelo e repeticao com mediana seriam o proximo ajuste.

Evidencia de falha provocada esperada:

```text
Se `networkpolicy-sentinel/prompt.md` permitir `- {}`, o assert `not-contains: - {}` falha e o job `networkpolicy-sentinel` reprova.
```

Evidencia de sucesso esperada:

```text
Todos os configs retornam PASS quando os prompts mantem os contratos de formato, custo, latencia e qualidade.
```
