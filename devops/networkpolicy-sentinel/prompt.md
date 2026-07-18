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

Regras obrigatorias:
- Nao use `podSelector: {}` na politica de allow do Sentinel, exceto na politica default-deny do namespace.
- Nao use regra vazia `- {}` em ingress ou egress.
- Ingress permitido: Relay e API gateway para pods `app=sentinel`.
- Egress permitido: Forge porta 5432, Cerebro porta 9200 e DNS interno porta 53 TCP/UDP.
- Inclua `policyTypes` com `Ingress` e `Egress`.
- Lembre que `podSelector: {}` seleciona todos os pods do namespace; use isso apenas no default-deny.
- A saida final deve conter YAML aplicavel e a verificacao resumida.

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
