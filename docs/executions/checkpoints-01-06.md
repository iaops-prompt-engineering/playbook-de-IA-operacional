# Checkpoints 01 a 06: execucoes e curadoria

## CP01 - triagem-de-pods

Prompt final: `devops/triagem-de-pods/prompt.md`

Metodo: meta-prompting com ciclo gerar -> criticar -> refinar. Curadoria final manteve papel claro de SRE, proibiu repetir apenas `STATUS`, obrigou cruzamento de status + eventos + logs e fechou contrato de saida curto para plantao.

Modelo usado na execucao curada: GPT-4o mini, escolhido por custo baixo e latencia adequada a plantao. Em dados reais, sanitizar tenants, hostnames, imagens privadas e tokens em logs antes de enviar a provedor externo.

Saida para Entrada 1:

```text
RESUMO: um pod da API esta em CrashLoopBackOff por esgotamento de memoria.

PODS PROBLEMATICOS:
- POD: sentinel-api-7d9c8b6f4-h4m2t
  STATUS OBSERVADO: CrashLoopBackOff, 14 restarts em 42m.
  CAUSA PROVAVEL: OOMKilled por heap chegando ao limite de 512Mi durante carga do cache.
  EVIDENCIAS:
  - Last State OOMKilled com exit code 137.
  - Limite de memoria do container em 512Mi.
  - Logs mostram heap 498Mi/512Mi e fatal out of memory logo antes da queda.
  ACAO DO PLANTAO: reduzir carga/cache ou aumentar limite de memoria de forma controlada e abrir follow-up para revisar consumo da versao 2.9.1.
```

Saida para Entrada 2:

```text
RESUMO: dois pods nao subiram, um por imagem inexistente e outro por falta de CPU no cluster.

PODS PROBLEMATICOS:
- POD: sentinel-api-7d9c8b6f4-zzp10
  STATUS OBSERVADO: ImagePullBackOff.
  CAUSA PROVAVEL: tag registry.aegis.io/sentinel-api:2.9.2 ausente ou nao publicada.
  EVIDENCIAS:
  - Evento "manifest unknown" ao tentar baixar `registry.aegis.io/sentinel-api:2.9.2`.
  - Evento `ErrImagePull`.
  - Kubelet entrou em back-off para nova tentativa de pull.
  ACAO DO PLANTAO: pausar rollout e corrigir tag/publicacao da imagem antes de retomar.
- POD: sentinel-ingest-8f7a9c5b-4dkqm
  STATUS OBSERVADO: Pending.
  CAUSA PROVAVEL: scheduler nao encontrou CPU suficiente para request de 2 CPUs.
  EVIDENCIAS:
  - Evento `FailedScheduling`.
  - Scheduler reporta `0/6 nodes are available`.
  - Motivo objetivo: `6 Insufficient cpu`.
  ACAO DO PLANTAO: reduzir request se seguro ou liberar/aumentar capacidade antes de reexecutar o deploy.
```

Saida para Entrada 3:

```text
RESUMO: nenhum pod problematico identificado no snapshot.
PODS PROBLEMATICOS: nenhum.
ACAO DO PLANTAO: manter monitoramento normal; o restart unico do worker ocorreu ha 3 dias e nao indica falha ativa.
```

Curadoria:
- Estrutura usada: Role + Objective + Rules + Output Contract.
- Refinamento principal: separar "pod problematico" de "restart antigo sem falha ativa".
- Ganho real: reduzir resposta vaga do tipo "CrashLoopBackOff = problema" e forcar inferencia apoiada em evidencia operacional.
- Limite assumido: sem snapshot de describe/logs suficientes, o prompt deve responder "indeterminada com os dados fornecidos", nao inventar causa.

## CP02 - nota-de-triagem

Prompt final: `devops/nota-de-triagem/prompt.md`

Metodo: meta-prompting com ciclo gerar -> criticar -> refinar. Escolha metodologica: ensinar por contrato fixo de saida e regras curtas, sem few-shot literal no prompt final.

Justificativa:
- Este checkpoint separa dois blocos: referencia de formato e alertas crus.
- Few-shot literal com exemplos completos aumenta risco de o modelo vazar conteudo do exemplo para a nota final.
- Contrato fixo com cinco rotulos reduz tokens, aumenta estabilidade e deixa o teste deterministico mais simples.

Modelo usado na execucao curada: GPT-4o mini. Claude Haiku segue como segundo provedor opcional de baixo custo para comparar consistencia de formatacao.

Saida para Entrada 1:

```text
ALERTA: Sentinel - autoscaler no maximo com fila do Relay crescendo 2k/min
IMPACTO: alerting em risco de atraso para tenant com volume 4x acima da linha de base
HIPOTESE INICIAL: onboarding de nova regiao do tenant saturou sentinel-api mesmo com 60 replicas
ACAO IMEDIATA: aplicar mitigacao de trafego/limite temporario para o tenant e acompanhar CPU/fila
ESCALAR PARA: @sentinel-core se fila ou CPU nao estabilizarem em 10min
```

Saida para Entrada 2:

```text
ALERTA: Relay - taxa de rejeicao de ingestao em 6% por 8min
IMPACTO: telemetry do tenant afetado pode chegar incompleta ou atrasada
HIPOTESE INICIAL: deploy das 02:55 saturou o buffer de ingestao
ACAO IMEDIATA: iniciar rollback do deploy e monitorar queda da rejeicao
ESCALAR PARA: @relay-core se rejeicao nao cair em 10min
```

Saida para Entrada 3:

```text
ALERTA: Forge - lag de consumer em 9min e subindo
IMPACTO: dados do Sentinel com atraso crescente por falha no batch anterior
HIPOTESE INICIAL: forge-batch-ingest atrasou por acumulacao apos job anterior falhar
ACAO IMEDIATA: verificar job anterior, liberar/reexecutar batch e acompanhar lag
ESCALAR PARA: @data-platform se lag nao estabilizar em 20min
```

Curadoria:
- Estrutura usada: Output Contract + Constraints + Escalation Mapping.
- Refinamento principal: explicitar que exemplos de formato nao sao dados da entrada atual.
- Ganho real: evitar mistura entre notas-modelo e alerta cru, problema comum quando o prompt aprende estilo por proximidade textual.
- Limite assumido: se alerta vier pobre em contexto, impacto e hipotese podem ficar mais conservadores, mas sem inventar fatos.

## CP03 - causa-raiz

Prompt final: `devops/causa-raiz/prompt.md`

Metodo: meta-prompting com ciclo gerar -> criticar -> refinar. Escolha metodologica: usar roteiro explicito de cadeia causal, porque o pacote mistura configuracao, metricas e logs de fontes diferentes.

Justificativa:
- Sem roteiro, modelo tende a listar sintomas ou resumir cada artefato separado.
- O contrato final obriga linha do tempo, correlacao entre fontes e separacao entre causa, mecanismo intermediario e consequencia.
- Tambem obriga bloco de sanitizacao antes de mandar artefatos de producao para modelo externo.

Modelo usado na execucao curada: GPT-4o mini para custo baixo. Escalada recomendada para GPT-4o quando artefatos ficarem maiores, mais ambíguos ou com mais hipoteses concorrentes.

Saida:

```text
CAUSA-RAIZ:
A reindexacao agendada das 02:00 ficou travada por horas e competiu com a carga normal de indexacao, saturando memoria e filas de escrita do Elasticsearch. Isso levou a circuit breaker, throttling, timeouts de busca, queda de cache e resultados parciais.

CADEIA DE EVIDENCIAS:
- A configuracao espera reindex de 90min, mas os logs mostram 38% as 08:02 e 41% as 09:58.
- Entre 08:00 e 10:00, indexed_docs_per_s sobe de 4200 para 12400 enquanto heap_used_pct sobe de 61% para 94%.
- A fila de escrita cresce ate 200/200 e passa a rejeitar bulk, coerente com saturacao de escrita.
- O heap chega a 7.7GB/8GB e o parent circuit breaker dispara, seguido de queries acima de 5s e resultados parciais 11/12 shards.

SINTOMAS E CONSEQUENCIAS:
- search_p99_ms subiu ate 6700ms.
- cache_hit_pct caiu para 29%, provavelmente efeito de eviction e pressao de memoria, nao causa primaria.
- buscas incompletas e falhas de shard surgiram quando memoria e filas ja estavam saturadas.

ACOES RECOMENDADAS:
- IMEDIATA: pausar ou reduzir prioridade da reindexacao e proteger trafego de busca.
- CURTO PRAZO: reagendar reindex para janela realista e limitar concorrencia/bulk size.
- PREVENCAO: revisar heap, capacidade por shard, refresh_interval durante reindex e alertas de progresso lento.

HIPOTESES MENOS PROVAVEIS:
- Problema primario de cache, porque a queda de hit ratio aparece junto com evictions apos aumento de heap e indexing pressure.
- Falha isolada de query, porque write queue, GC, circuit breaker e reindex travada aparecem antes dos timeouts.

LACUNAS E INCERTEZAS:
- Falta distribuicao por outros nodes e volume real de queries.
- Falta saber por que a reindexacao ficou lenta desde 02:00.

SANITIZACAO ANTES DE MODELO EXTERNO:
- Mascarar nomes de nodes, indices, tenants, hostnames, repositorios e qualquer dado de cliente em logs.
```

Curadoria:
- Estrutura usada: timeline -> causal chain -> consequences -> actions -> rejected hypotheses -> gaps -> sanitization.
- Refinamento principal: explicitar que cache hit baixo, timeout e resultado parcial sao efeitos da pressao de heap/escrita, nao causa primaria.
- Ganho real: impedir resposta rasa do tipo "Elasticsearch lento por heap alto" sem conectar reindex travada, fila de escrita, throttling, breaker e falha de shard.
- Dados a tratar antes de modelo externo: nomes de nodes, indices, tenants, namespaces internos, hostnames, repositorios, IDs de task e qualquer dado de cliente nos logs.

## CP04 - backpressure-relay

Prompt final: `devops/backpressure-relay/prompt.md`

Metodo: meta-prompting com ciclo gerar -> criticar -> refinar. Escolha metodologica: matriz curta de alternativas seguida de recomendacao combinada, porque este checkpoint exige comparacao e nao resposta unica precoce.

Justificativa:
- Existe mais de um caminho defensavel e cada um cobra custo, complexidade e impacto operacional diferentes.
- Sem contrato de comparacao, modelo tende a escolher uma opcao cedo demais e esconder trade-offs.
- O contrato final força restricoes inegociaveis, recomendacao, rollout, descartes e metricas de sucesso.

Modelo usado na execucao curada: GPT-4o mini para custo baixo. Escalada recomendada para GPT-4o quando houver mais alternativas, dados de custo mais detalhados ou necessidade de sintese executiva mais refinada.

Saida:

```text
DECISAO RECOMENDADA:
Combinar prioridade de consumo para Sentinel, isolamento por tenant barulhento e fila duravel de backlog para Forge, com autoscaling limitado por teto de custo.

RESTRICOES QUE GOVERNAM A DECISAO:
- Sentinel tem SLA de atraso maximo de 60s.
- Forge tolera ate 15min de atraso.
- Perda de telemetry e inaceitavel.
- Orcamento ja esta 8% acima do previsto.

COMPARACAO DE ALTERNATIVAS:
| Alternativa | Beneficio | Risco/custo | Adequacao |
| --- | --- | --- | --- |
| Prioridade ao Sentinel | protege alerting em tempo real | Forge acumula backlog | alta |
| DLQ/backlog duravel para Forge | preserva telemetry | precisa replay e observabilidade | alta |
| Relay por tenant | isola cliente barulhento | aumenta complexidade | alta para top tenants |
| Autoscaling de consumidores | absorve picos | custo e limite fisico do Relay | media |

ROLLOUT PROPOSTO:
1. Criar prioridade logica de filas para Sentinel.
2. Adicionar backlog duravel para Forge com replay idempotente.
3. Isolar top tenants com maior variancia.
4. Ativar autoscaling com teto e alerta de custo.

DESCARTES:
- Drop de mensagens, porque viola a premissa do produto.
- Autoscaling puro, porque o pico 320k msgs/s excede muito o sustentado e o custo ja esta pressionado.

METRICAS DE SUCESSO:
- atraso p99 do Sentinel abaixo de 60s.
- lag do Forge abaixo de 15min.
- zero perda de mensagens confirmada por contadores de ingestao e replay.
```

Curadoria:
- Estrutura usada: restrictions -> alternatives matrix -> rollout -> discards -> success metrics.
- Refinamento principal: forcar resposta combinada quando uma unica alavanca nao resolve SLA, custo e perda zero ao mesmo tempo.
- Ganho real: impedir recomendacao rasa como "autoscale tudo" ou "priorize Sentinel" sem enfrentar backlog do Forge, custo trimestral e isolamento de tenant barulhento.
- Dados a tratar antes de modelo externo: nomes de tenants, topologia interna do barramento, limites reais de capacidade, custos unitarios e qualquer identificador de cliente.

## CP05 - migracao-forge-event-driven

Prompts finais:
- `devops/migracao-forge-event-driven/prompt.md`
- `devops/migracao-forge-event-driven/elo-2-estrategia/prompt.md`
- `devops/migracao-forge-event-driven/elo-3-plano-reversivel/prompt.md`

Metodo: meta-prompting com ciclo gerar -> criticar -> refinar em tres elos encadeados. Escolha metodologica: separar diagnostico, estrategia e plano reversivel para evitar resposta monolitica rasa.

Justificativa:
- O problema mistura estado atual, convivencia temporaria entre batch e event-driven, migracao de dependentes e rollback.
- Um unico prompt tende a misturar analise, decisao e execucao cedo demais.
- A cadeia obriga passagem real da saida anterior para o proximo elo e deixa o raciocinio auditavel.

Modelo usado na execucao curada: GPT-4o mini em cada elo para custo baixo. Escalada recomendada para GPT-4o quando houver schemas reais, metricas de volume ou mais consumidores.

A versao monolitica foi substituida por uma cadeia real de tres prompts separados. O registro detalhado da separacao e dos outputs de cada elo esta em `docs/executions/cp05-cadeia-forge-encadeada.md`.

Curadoria:
- Elo 1 para no diagnostico e prepara entrada concreta para o Elo 2.
- Elo 2 escolhe estrategia incremental, compara alternativas e registra pontos de reversao.
- Elo 3 converte estrategia em fases, gates e rollback pratico.
- Dados a tratar antes de modelo externo: nomes de jobs, tabelas, schemas, tenants, consumidores internos e janelas de billing.

## CP06 - networkpolicy-sentinel

Prompt final: `devops/networkpolicy-sentinel/prompt.md`

Metodo: meta-prompting com ciclo gerar -> criticar -> refinar. Escolha metodologica: registrar iteracoes explicitas v1 -> verificacao -> v2, porque manifesto de seguranca nao deve sair na primeira tentativa.

Justificativa:
- O manifesto barrado e um allow-all disfarçado: `podSelector: {}` seleciona todos os pods do namespace e `- {}` libera tudo.
- Sem rodada de verificacao, detalhes criticos passam batido, como DNS incompleto em egress.
- O contrato final força YAML aplicavel, auto-critica e correcoes ponto a ponto.

Modelo usado na execucao curada: GPT-4o mini para custo baixo. Segundo provedor opcional: Claude Haiku para comparar clareza do YAML e da verificacao.

Saida resumida:

V1:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: sentinel-default-deny
  namespace: sentinel-prod
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: sentinel-allow-required-flows
  namespace: sentinel-prod
spec:
  podSelector:
    matchLabels:
      app: sentinel
  policyTypes: [Ingress, Egress]
  ingress:
    # Allow Relay to deliver telemetry events to Sentinel.
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: relay-prod
          podSelector:
            matchLabels:
              app: relay
    # Allow platform API gateway to reach Sentinel APIs.
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: edge
          podSelector:
            matchLabels:
              app: api-gateway
  egress:
    # Allow Sentinel to query Forge warehouse on PostgreSQL.
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: forge-prod
          podSelector:
            matchLabels:
              app: forge
      ports:
        - protocol: TCP
          port: 5432
    # Allow Sentinel to query Cerebro Elasticsearch.
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: cerebro-prod
          podSelector:
            matchLabels:
              app: cerebro
      ports:
        - protocol: TCP
          port: 9200
    # Allow Sentinel DNS queries over UDP.
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
```

Verificacao apontou que faltava DNS TCP/53 e que os comentarios precisavam cobrir todas as regras.

V2 adicionou regra DNS TCP/53 e manteve default-deny separado. Nao ha `- {}` e o allow usa `app=sentinel`.

Curadoria:
- Estrutura usada: generate -> security review -> refine.
- Refinamento principal: corrigir DNS para TCP e UDP/53 e confirmar que `podSelector: {}` fica restrito ao default-deny.
- Ganho real: impedir falso endurecimento que ainda quebra resolucao DNS ou ainda carrega seletor amplo demais.
- Validacao metodologica: doc oficial de Kubernetes confirma que seletor vazio seleciona todos os pods do namespace e que default-deny de egress bloqueia DNS ate liberacao explicita.
- Dados a tratar antes de modelo externo: namespaces internos, labels proprietarias, topologia de servicos, portas internas nao publicas e qualquer convencao sensivel do cluster.
