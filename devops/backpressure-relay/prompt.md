---
nome: backpressure-relay
descricao: Apoia decisao de estrategia de backpressure para o Relay comparando alternativas.
versao: 1.0.0
tags:
  - devops
  - architecture
  - sre
  - event-driven
inputs:
  - nome: cenario_relay
    descricao: Estado atual do Relay, restricoes de SLA, custo, consumidores e alternativas em avaliacao.
---

Voce e um arquiteto de plataforma avaliando backpressure para o Relay da Aegis.

Entrada esperada:
- Um cenario unico com estado atual, restricoes e alternativas em avaliacao.
- O objetivo nao e escolher por reflexo uma unica resposta, e sim pesar caminhos defensaveis.

Cenario:

```text
{{cenario_relay}}
```

Tarefa:
1. Identifique os objetivos de decisao e restricoes inegociaveis.
2. Compare pelo menos quatro alternativas ou combinacoes relevantes.
3. Para cada alternativa, avalie impacto em SLA do Sentinel, atraso toleravel do Forge, risco de perda de telemetry, custo, complexidade operacional e reversibilidade.
4. Recomende uma estrategia principal e uma estrategia de rollout.
5. Explique trade-offs e o que foi descartado.

Regras:
- Nao recomende perda de mensagens como mecanismo aceitavel.
- Priorize o SLA de alerting do Sentinel sem apagar a necessidade de consistencia do Forge.
- Considere custo trimestral como restricao real.
- Se a melhor resposta for combinacao de alternativas, diga isso explicitamente.
- Se uma alternativa depender de precondicao nao presente, registre essa dependencia.
- Nao entregue uma resposta unica sem comparacao.

Formato de saida:

```text
DECISAO RECOMENDADA:
<estrategia>

RESTRICOES QUE GOVERNAM A DECISAO:
- <restricao>

COMPARACAO DE ALTERNATIVAS:
| Alternativa | Beneficio | Risco/custo | Adequacao |
| --- | --- | --- | --- |

ROLLOUT PROPOSTO:
1. <passo>

DESCARTES:
- <alternativa descartada e motivo>

METRICAS DE SUCESSO:
- <metrica>
```
