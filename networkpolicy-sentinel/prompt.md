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
5. Depois da v1, faca uma verificacao critica da propria saida e produza v2 corrigida.

Regras obrigatorias:
- Nao use `podSelector: {}` na politica de allow do Sentinel, exceto na politica default-deny do namespace.
- Nao use regra vazia `- {}` em ingress ou egress.
- Ingress permitido: Relay e API gateway para pods `app=sentinel`.
- Egress permitido: Forge porta 5432, Cerebro porta 9200 e DNS interno porta 53 TCP/UDP.
- Inclua `policyTypes` com `Ingress` e `Egress`.
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
