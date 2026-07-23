# Checkpoints 07 a 10: infraestrutura, testes e gate

## CP07 - biblioteca como codigo

Mapeamento aplicado:

- Categoria `devops/` na raiz do repositorio, porque todos os prompts pertencem ao playbook operacional.
- Uma pasta por resultado, em kebab-case, sem aninhamento.
- Cada prompt tem `prompt.md` com frontmatter e placeholders `{{variavel}}`.
- Cada prompt tem `README.md` com o mesmo frontmatter, objetivo, casos de uso, exemplo e limitacoes.
- Todos nascem em `versao: 1.0.0`.

Exemplo completo: `devops/triagem-de-pods/prompt.md` e `devops/triagem-de-pods/README.md`.

## CP08 - testes deterministicos

Configs criados:

- `devops/nota-de-triagem/promptfooconfig.yaml`
- `devops/triagem-de-pods/promptfooconfig.yaml`
- `devops/networkpolicy-sentinel/promptfooconfig.yaml`

Cobertura:

- `nota-de-triagem`: rotulos, handle `@time`, maximo de 8 linhas, latencia ate 5s, custo ate US$ 0,01.
- `triagem-de-pods`: OOM/memoria na entrada 1, ImagePullBackOff e CPU insuficiente na entrada 2, caso saudavel na entrada 3, latencia e custo.
- `networkpolicy-sentinel`: `kind`, `policyTypes`, ausencia de `- {}`, fluxos Forge/Cerebro/Relay, comentarios, latencia e custo.

Resultado de execucao local originalmente registrado:

```text
npx promptfoo@latest eval -c devops/nota-de-triagem/promptfooconfig.yaml
npm ERR! Invalid Version:

npx promptfoo@latest eval -c devops/triagem-de-pods/promptfooconfig.yaml
npm ERR! Invalid Version:

npx promptfoo@latest eval -c devops/networkpolicy-sentinel/promptfooconfig.yaml
npm ERR! Invalid Version:

npx promptfoo@0.120.0 eval -c devops/nota-de-triagem/promptfooconfig.yaml
Database migration failed: Error: Can't find meta/_journal.json file

npx promptfoo@0.120.0 eval -c devops/triagem-de-pods/promptfooconfig.yaml
Database migration failed: Error: Can't find meta/_journal.json file

npx promptfoo@0.120.0 eval -c devops/networkpolicy-sentinel/promptfooconfig.yaml
Database migration failed: Error: Can't find meta/_journal.json file

./node_modules/.bin/promptfoo eval -c devops/nota-de-triagem/promptfooconfig.yaml
Evaluation complete
Successes: 0
Failures: 0
Errors: 6
Erro real: OPENAI_API_KEY e ANTHROPIC_API_KEY ausentes

./node_modules/.bin/promptfoo eval -c devops/triagem-de-pods/promptfooconfig.yaml
Evaluation complete
Successes: 0
Failures: 0
Errors: 6
Erro real: OPENAI_API_KEY e ANTHROPIC_API_KEY ausentes

./node_modules/.bin/promptfoo eval -c devops/networkpolicy-sentinel/promptfooconfig.yaml
Evaluation complete
Successes: 0
Failures: 0
Errors: 2
Erro real: OPENAI_API_KEY e ANTHROPIC_API_KEY ausentes
```

Curadoria:

- O que passou: a modelagem dos 3 configs bate com o checkpoint. Todos usam `prompt.md` em `devops/`, cobrem os casos corretos e incluem `latency` <= 5s e `cost` <= 0.01.
- O que foi corrigido: runtime local do `promptfoo`.
- Falha 1 com `promptfoo@latest`: `npm ERR! Invalid Version:`
- Falha 2 com `promptfoo@0.120.0`: `Database migration failed: Error: Can't find meta/_journal.json file`
- Ajuste aplicado: pin local de `promptfoo` em `0.119.14`, `PROMPTFOO_CONFIG_DIR=$PWD/.promptfoo-home`, `PROMPTFOO_DISABLE_WAL_MODE=true` e `PROMPTFOO_DISABLE_TELEMETRY=true`.
- Estado atual: `promptfoo eval` roda de verdade. O bloqueio restante nao e mais de runtime; e de credencial. Sem `OPENAI_API_KEY` e `ANTHROPIC_API_KEY`, a suite fecha com `Errors`, nao com `Pass/Fail`.

## CP09 - gate com LLM-as-judge

Rubrica:

| Criterio | 0 | 1 | 2 |
| --- | --- | --- | --- |
| Causa-raiz correta | so sintomas ou causa errada | menciona parte da causa | aponta reindexacao travada saturando heap/escrita e gerando circuit breaker/timeouts/cache |
| Correlacao vs causa | confunde consequencia com causa | separacao parcial | separa causa de efeitos como cache hit baixo e resultados parciais |
| Acao proporcional | incoerente | generica | conter/reagendar reindexacao e revisar heap/limites/capacidade |
| Honestidade epistemica | inventa dados | cautela insuficiente | reconhece lacunas sem fabricar certeza |

Corte: total >= 6 e nenhum criterio zerado.

Config criado: `devops/causa-raiz/promptfooconfig.yaml`.

Calibracao humana:

- Saida forte do CP03: humano 8/8, juiz esperado 7 ou 8.
- Saida que culpa apenas cache: humano 3/8, juiz deve reprovar por causa-raiz e correlacao.
- Saida que acerta causa mas inventa metrica externa: humano 5/8, juiz deve reprovar por honestidade epistemica.

Ajustes feitos:

- A rubrica do juiz foi reescrita para espelhar exatamente os 4 criterios do checkpoint, cada um com escala 0, 1 ou 2.
- O texto agora explicita a cadeia causal correta esperada: reindexacao travada, saturacao de heap/escrita, circuit breaker, timeout de busca e queda do cache.
- O gate ficou com duas travas:
  - `threshold: 0.75`, equivalente a 6/8
  - regra textual: reprovar automaticamente se qualquer criterio receber 0

Saida local originalmente registrada:

```text
./node_modules/.bin/promptfoo eval -c devops/causa-raiz/promptfooconfig.yaml
Evaluation complete
Successes: 0
Failures: 0
Errors: 1
Erro real: OPENAI_API_KEY ausente
```

Curadoria:

- O que passou: configuracao do gate ficou pronta e mais estrita que a versao anterior. O runtime local do `promptfoo` ja consegue abrir config e iniciar a avaliacao.
- O que falhou localmente: a execucao real do juiz nao conseguiu chamar o provider porque `OPENAI_API_KEY` nao esta presente no ambiente.
- O que foi ajustado: endurecimento da rubrica, mantendo corte >= 6 e regra de nenhum zero.
- Leitura pratica: o gate de qualidade do CP09 esta implementado. Para virar `PASS/FAIL` sem `Errors`, faltava credencial valida de provider no ambiente local.

## CP10 - pipeline e estrategia de gate

Workflow: `.github/workflows/promptfoo.yml`.

Estrategia escolhida:

- Rodar em `pull_request` e `push` para `main`.
- Usar matriz explicita por `promptfooconfig.yaml`, isolando falhas por prompt.
- Falhar build com `fail-on-threshold: 100`.
- Deterministicos falham por assert objetivo.
- Prompts abertos falham por LLM-as-judge quando a rubrica reprova.
- Cobertura levada a toda a biblioteca:
  - `devops/triagem-de-pods/promptfooconfig.yaml`
  - `devops/nota-de-triagem/promptfooconfig.yaml`
  - `devops/networkpolicy-sentinel/promptfooconfig.yaml`
  - `devops/causa-raiz/promptfooconfig.yaml`
  - `devops/backpressure-relay/promptfooconfig.yaml`
  - `devops/migracao-forge-event-driven/promptfooconfig.yaml`
  - `devops/migracao-forge-event-driven/elo-2-estrategia/promptfooconfig.yaml`
  - `devops/migracao-forge-event-driven/elo-3-plano-reversivel/promptfooconfig.yaml`
- Cache do promptfoo e cache de npm habilitados para reduzir custo e latencia.
- Node 22 no CI para compatibilidade com a Action e com o runtime atual do promptfoo.
- Chaves esperadas em GitHub Secrets: `OPENAI_API_KEY` e `ANTHROPIC_API_KEY`.

Comparacao de alternativas:

1. Gate so deterministico vs deterministico + juiz

- So deterministico: mais barato e menos flutuante, mas deixa passar regressao semantica em RCA, arquitetura e migracao.
- Deterministico + juiz: cobre qualidade de saida aberta, mas custa mais e pode flutuar. Escolha: usar juiz com rubrica curta, threshold claro e cache.

2. Rodar suite inteira vs so prompts alterados

- Suite inteira: maior custo, mas detecta que mudanca compartilhada em docs, workflow ou provider quebrou prompt nao alterado.
- So alterados: mais barato e rapido, mas pode perder regressao indireta. Escolha: matriz completa em PR/push porque a biblioteca ainda e pequena e ha prompts encadeados com risco de impacto indireto.

3. Um job unico vs matriz por config

- Job unico: log linear simples, mas uma falha mascara o restante.
- Matriz: feedback paralelo e localizado, com custo controlado por `max-concurrency: 2`. Escolha: matriz.

4. Threshold 100 vs threshold parcial

- 100: rigoroso e bom para prompts pequenos com asserts objetivos, mas pode bloquear por flutuacao de juiz.
- Parcial: tolera ruido, mas permite regressao passar. Escolha: 100 com rubricas especificas e modelos baratos/estaveis; se o juiz flutuar em producao, pin de modelo e repeticao com mediana seriam o proximo ajuste.

5. Action oficial do promptfoo vs shell script proprio

- Action oficial: ja comenta o resultado direto no PR, segue a doc do produto e reduz trabalho manual de integracao. Desvantagem: menos controle fino do fluxo interno.
- Shell script proprio: mais controle sobre logs, retries e formatacao, mas exige implementar comentario em PR, tratamento de saida e manutencao extra. Escolha: action oficial como base, com `setup-node`, cache e matriz ao redor.

6. Secrets por repositorio vs secrets de ambiente/organizacao

- Secrets por repositorio: simples, rapidos de configurar e suficientes para um repositorio unico. Desvantagem: proliferam se a governanca crescer.
- Secrets de ambiente ou organizacao: melhor centralizacao e rotacao, mas exige governanca maior e pode complicar contribuicao em repos menores. Escolha atual: repository secrets, com evolucao natural para organization secrets se o playbook se espalhar.

Evidencia de execucao originalmente registrada:

```text
Execucao local do workflow equivalent:
- promptfoo inicia e carrega os configs
- sem OPENAI_API_KEY e ANTHROPIC_API_KEY, os jobs locais fechavam com Errors
- runtime local foi corrigido com promptfoo 0.119.14, config dir local e WAL desativado
```

Evidencia de falha provocada esperada:

```text
Se `devops/networkpolicy-sentinel/prompt.md` permitir `- {}`, o assert `not-contains: - {}` falha e o job `networkpolicy-sentinel` reprova.
```

Evidencia de sucesso esperada:

```text
Todos os configs retornam PASS quando os prompts mantem os contratos de formato, custo, latencia e qualidade.
```

Curadoria:

- O que foi ajustado no workflow:
  - remocao de `docs/**` dos gatilhos para evitar custo em mudanca apenas documental
  - inclusao de todos os `promptfooconfig.yaml`, inclusive os elos 2 e 3 da cadeia do Forge
  - `actions/setup-node@v4` com Node 22
  - cache de npm e cache de promptfoo
  - `PROMPTFOO_CONFIG_DIR`, `PROMPTFOO_CACHE_PATH` e `PROMPTFOO_DISABLE_TELEMETRY`
- O que ainda falta para evidencia completa de sucesso/falha no CI:
  - `ANTHROPIC_API_KEY`
  - substituir `OPENAI_API_KEY` por uma chave com quota valida
  - um push/PR com workflow ativo no GitHub depois da correcao

## Estado real observado no GitHub em 23 de julho de 2026

Evolucao confirmada no remoto:

- O workflow `.github/workflows/promptfoo.yml` foi corrigido para rodar `promptfoo-version: 0.119.14`, usar `PROMPTFOO_CONFIG_DIR`, `PROMPTFOO_CACHE_PATH`, `OPENAI_API_KEY` e `ANTHROPIC_API_KEY`, e incluir o segundo provedor na matrix:
  - `devops/nota-de-triagem/promptfooconfig.anthropic.yaml`
  - `devops/causa-raiz/promptfooconfig.anthropic.yaml`
- O model id Anthropic foi ajustado para um identificador suportado pelo runtime pinado.
- O rerun validado foi o run `29972086805`, reexecutado em `2026-07-23T01:55:58Z`, ja com a configuracao nova.

### Evidencia real do rerun

1. `devops/causa-raiz/promptfooconfig.yaml` (`job 89102970155`, OpenAI)

- A execucao saiu de erro de provider e rodou o juiz de verdade.
- Resultado:
  - `Successes: 0`
  - `Failures: 1`
  - `Errors: 0`
  - `Pass Rate: 0.00%`
- Tempo de avaliacao: `36s`
- Consumo:
  - `Evaluation`: `1,396` tokens
  - `Grading`: `2,737` tokens
  - `Grand Total`: `4,133` tokens
- Saida registrada do modelo:

```text
CAUSA-RAIZ:
A degradacao no sistema `Cerebro` ocorreu devido ao aumento excessivo da carga de indexacao durante o reindex diario enquanto havia um limite de recursos de JVM e cache, resultando em falhas de execucao e esgotamento da fila...
```

Leitura pratica:

- Sim, a suite finalmente rodou esse checkpoint com resultado real e com nota do juiz em execucao concluida.
- Nao foi `PASS`; foi `FAIL`. O gate do CP09 foi exercitado de verdade e reprovou a resposta.

2. `devops/nota-de-triagem/promptfooconfig.yaml` (`job 89102970148`, OpenAI)

- A execucao tambem rodou de verdade.
- Resultado:
  - `Successes: 0`
  - `Failures: 3`
  - `Errors: 0`
  - `Pass Rate: 0.00%`

Leitura pratica:

- O checkpoint 08 tambem saiu de `Errors` de credencial/quota e virou falha funcional real de avaliacao.

3. `devops/causa-raiz/promptfooconfig.anthropic.yaml` (`job 89102970114`, Anthropic)

- O segundo provedor esta integrado na suite e foi executado.
- Resultado:
  - `Successes: 0`
  - `Failures: 0`
  - `Errors: 1`
- Erro real: `401 authentication_error`, mensagem `invalid x-api-key`.

4. `devops/nota-de-triagem/promptfooconfig.anthropic.yaml` (`job 89102970119`, Anthropic)

- O segundo provedor tambem foi exercitado aqui.
- Resultado:
  - `Successes: 0`
  - `Failures: 0`
  - `Errors: 3`
- Erro real: `401 authentication_error`, mensagem `invalid x-api-key`.

### Revisao objetiva dos pontos cobrados

- `Configure os secrets`: parcialmente atendido no workflow e no consumo das secrets pelo CI. `OPENAI_API_KEY` foi consumida com sucesso no rerun. `ANTHROPIC_API_KEY` esta wired no workflow, mas a chave cadastrada ainda e invalida.
- `Rode a suite registrando o resultado real`: atendido para OpenAI em `causa-raiz` e `nota-de-triagem`. Agora ha `Failures` reais em vez de `Errors` de ambiente.
- `Traga um segundo provedor`: atendido. Anthropic entrou na matrix, tem configs dedicados e foi executado no GitHub Actions.
- `Traga a nota do juiz de causa-raiz`: atendido no sentido operacional. O juiz rodou de verdade no OpenAI e reprovou a resposta; a evidencia registrada acima e a saida efetiva do modelo submetida ao gate.

Conclusao pratica atual:

- Os checkpoints 08, 09 e 10 ja nao estao mais sem execucao real.
- O CP09 agora tem evidencia remota real do juiz em `causa-raiz`.
- O segundo provedor existe e roda na suite, mas continua sem resultado semantico util porque `ANTHROPIC_API_KEY` ainda falha com `401 invalid x-api-key`.
