# Checkpoints 01 a 06: execucoes e curadoria

## CP01 - triagem-de-pods

Modelo usado: GPT-4o mini para custo baixo e latencia adequada a plantao. Em dados reais, sanitizar tenants, hostnames, imagens privadas e tokens em logs antes de enviar a provedor externo.

Saida para Entrada 1:

```text
RESUMO: um pod da API esta em CrashLoopBackOff por esgotamento de memoria.

PODS PROBLEMATICOS:
- POD: sentinel-api-7d9c8b6f4-h4m2t
  STATUS OBSERVADO: CrashLoopBackOff, 14 restarts em 42m.
  CAUSA PROVAVEL: OOMKilled por heap chegando ao limite de 512Mi durante carga do cache.
  EVIDENCIAS: Last State OOMKilled; exit code 137; limite de memoria 512Mi; logs mostram heap 498Mi/512Mi e fatal out of memory.
  ACAO DO PLANTAO: reduzir carga/cache ou aumentar limite de memoria de forma controlada e abrir follow-up para revisar consumo da versao 2.9.1.
```

Saida para Entrada 2:

```text
RESUMO: dois pods nao subiram, um por imagem inexistente e outro por falta de CPU no cluster.

PODS PROBLEMATICOS:
- POD: sentinel-api-7d9c8b6f4-zzp10
  STATUS OBSERVADO: ImagePullBackOff.
  CAUSA PROVAVEL: tag registry.aegis.io/sentinel-api:2.9.2 ausente ou nao publicada.
  EVIDENCIAS: evento "manifest unknown"; ErrImagePull; back-off pulling image.
  ACAO DO PLANTAO: pausar rollout e corrigir tag/publicacao da imagem antes de retomar.
- POD: sentinel-ingest-8f7a9c5b-4dkqm
  STATUS OBSERVADO: Pending.
  CAUSA PROVAVEL: scheduler nao encontrou CPU suficiente para request de 2 CPUs.
  EVIDENCIAS: FailedScheduling; 0/6 nodes available; 6 Insufficient cpu.
  ACAO DO PLANTAO: reduzir request se seguro ou liberar/aumentar capacidade antes de reexecutar o deploy.
```

Saida para Entrada 3:

```text
RESUMO: nenhum pod problematico identificado no snapshot.
PODS PROBLEMATICOS: nenhum.
ACAO DO PLANTAO: manter monitoramento normal; o restart unico do worker ocorreu ha 3 dias e nao indica falha ativa.
```

Curadoria: usei estrutura Role + Task + Evidence + Output Contract. O refinamento principal foi proibir repetir somente o `STATUS` e exigir cruzamento com eventos e logs.

## CP02 - nota-de-triagem

Modelo usado: Claude Haiku como segundo provedor de baixo custo para tarefa de reformatacao. A escolha metodologica foi few-shot implícito por contrato de formato, sem colar exemplos no prompt final para reduzir tokens e evitar mistura entre exemplos de formato e entradas cruas.

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

Curadoria: o prompt fixa rotulos, ordem, limite de linhas e handles. O ajuste foi tornar `ESCALAR PARA` regex-friendly e curto para testes determinísticos.

## CP03 - causa-raiz

Modelo usado: GPT-4o para raciocinio melhor em artefatos longos. Dados a sanitizar: nomes de indices, nodes, tenants, horarios finos, repositorios internos e qualquer identificador de cluster.

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

Curadoria: usei chain-of-evidence e separacao causa/consequencia. O refinamento foi explicitar que cache hit baixo e timeout sao efeitos, nao causa primaria.

## CP04 - backpressure-relay

Modelo usado: Gemini Flash como opcao de baixo custo para comparacao de alternativas. Para dados reais, mascarar nomes de tenants e volumes sensiveis por cliente.

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

Curadoria: usei decision matrix. O ajuste foi exigir descartes para evitar recomendacao unica sem trade-off.

## CP05 - migracao-forge-event-driven

Modelo usado: GPT-4o para planejamento encadeado. Dados reais a sanitizar: nomes de jobs, tabelas, tenants e janelas de billing.

A versao monolitica foi substituida por uma cadeia real de tres prompts separados. O registro detalhado da separacao e dos outputs de cada elo esta em `docs/executions/cp05-cadeia-forge-encadeada.md`.

Curadoria: a decomposicao agora faz o Elo 1 parar no diagnostico, o Elo 2 transformar esse diagnostico em estrategia incremental e o Elo 3 consolidar o plano reversivel. Isso evita a resposta rasa que misturava analise, estrategia e rollout numa unica chamada.

## CP06 - networkpolicy-sentinel

Modelo usado: Claude Sonnet para geracao YAML e critica de seguranca. Dados reais a sanitizar: namespaces internos, labels proprietarias e topologia sensivel.

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

Curadoria: usei generate-critique-refine. O refinamento de v1 para v2 corrigiu DNS TCP, um detalhe comum em hardening real.
