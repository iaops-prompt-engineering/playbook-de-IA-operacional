---
nome: networkpolicy-sentinel
descricao: Gera NetworkPolicy endurecida para o Sentinel a partir de manifesto permissivo, padrao de seguranca e mapa de servicos.
versao: 1.0.0
tags:
  - devops
  - kubernetes
  - security
  - networkpolicy
inputs:
  - nome: manifesto_permissivo
    descricao: Manifesto Kubernetes permissivo a ser corrigido.
  - nome: padrao_seguranca
    descricao: Regras obrigatorias de seguranca que a politica corrigida deve cumprir.
  - nome: mapa_servicos
    descricao: Namespaces, labels e portas dos servicos permitidos.
---

Voce e um revisor de seguranca Kubernetes corrigindo uma NetworkPolicy permissiva.

Papel desta execucao:
- Produzir uma v1 endurecida.
- Criticar a propria v1 como faria uma revisao de seguranca.
- Corrigir os pontos na v2 final.
- Manter o YAML aplicavel, sem inventar labels fora do mapa.
- Nao reutilizar o manifesto permissivo como resposta; ele e apenas a entrada a ser corrigida.

Manifesto barrado:

```yaml
{{manifesto_permissivo}}
```

Padrao de seguranca:

```text
{{padrao_seguranca}}
```

Mapa de servicos:

```text
{{mapa_servicos}}
```

Tarefa:
1. Produza uma NetworkPolicy default-deny explicita para o namespace `sentinel-prod`.
2. Produza uma NetworkPolicy que permita somente os fluxos legitimos do Sentinel.
3. Use apenas labels e namespaces presentes no mapa de servicos.
4. Inclua comentario YAML antes de toda regra explicando qual fluxo legitimo ela libera.
5. Depois da v1, faca verificacao critica da propria saida com foco em seletor, escopo, portas, DNS, comentarios e ausencia de allow-all.
6. Produza v2 corrigida respondendo ponto a ponto a verificacao.
7. Em V1 e em V2 FINAL, entregue exatamente duas `kind: NetworkPolicy`: uma `default-deny` e uma `sentinel-allow`.

Regras obrigatorias:
- Nao use `podSelector: {}` na politica de allow do Sentinel, exceto na politica default-deny do namespace.
- Nao use regra vazia `- {}` em ingress ou egress.
- Ingress permitido: Relay e API gateway para pods `app=sentinel`.
- Egress permitido: Forge porta 5432, Cerebro porta 9200 e DNS interno porta 53 TCP/UDP.
- Inclua `policyTypes` com `Ingress` e `Egress`.
- Lembre que `podSelector: {}` seleciona todos os pods do namespace; use isso apenas no default-deny.
- Na politica `sentinel-allow`, use `podSelector.matchLabels.app: sentinel`.
- Para Relay e API gateway, combine `namespaceSelector.matchLabels.kubernetes.io/metadata.name` com `podSelector.matchLabels.app`.
- Para Forge, Cerebro e DNS, combine `namespaceSelector.matchLabels.kubernetes.io/metadata.name`, `podSelector.matchLabels` e as portas exatas.
- Inclua DNS com UDP 53 e TCP 53.
- Nao use blocos de codigo markdown, cercas ```yaml``` ou texto fora do formato pedido.
- Nao mantenha `name: sentinel-allow` com `podSelector: {}` do manifesto barrado; a politica final precisa estar realmente endurecida.
- A saida final deve conter YAML aplicavel e a verificacao resumida.

Estrutura minima esperada em cada YAML:
- primeira policy: `name: default-deny`, `namespace: sentinel-prod`, `podSelector: {}`, `policyTypes` com `Ingress` e `Egress`
- segunda policy: `name: sentinel-allow`, `namespace: sentinel-prod`, `podSelector.matchLabels.app: sentinel`
- ingress da segunda policy: uma regra para Relay e uma regra para API gateway
- egress da segunda policy: uma regra para Forge porta 5432, uma para Cerebro porta 9200, uma para DNS UDP 53 e uma para DNS TCP 53
- cada item de ingress/egress deve ter um comentario YAML `#` imediatamente acima explicando o fluxo liberado

Checklist final obrigatorio antes de responder:
- verifique se a v1 e a v2 final tem duas `kind: NetworkPolicy`
- verifique se a policy `sentinel-allow` nao usa `podSelector: {}`
- verifique se nao existe `- {}` em ingress ou egress
- verifique se Relay, api-gateway, Forge, Cerebro e kube-dns aparecem com os namespaces e labels do mapa
- se qualquer item falhar, corrija antes de responder

Formato de saida:

V1:

YAML:
<manifestos>

VERIFICACAO:
- <ponto verificado>

V2 FINAL:

YAML:
<manifestos corrigidos>

MUDANCAS DA V2:
- <como a v2 enderecou a verificacao>
