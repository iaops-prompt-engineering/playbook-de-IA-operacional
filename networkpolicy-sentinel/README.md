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

# networkpolicy-sentinel

## Objetivo

Converter manifestos permissivos em NetworkPolicies endurecidas com default-deny e allowlist explicita.

## Casos de uso

- Revisao de politicas Kubernetes antes de producao.
- Refino iterativo de artefatos de seguranca.
- Padronizacao de politicas por namespace e fluxo legitimo.

## Exemplo

Entrada: manifesto allow-all, padrao da Aegis e mapa de servicos.

Saida: v1, verificacao critica, v2 final e mudancas aplicadas.

## Limitacoes

- Selectores dependem da existencia real de labels nos namespaces.
- Namespace selectors por nome podem exigir label `kubernetes.io/metadata.name` disponivel no cluster.
- Deve ser validado com `kubectl apply --dry-run=server` antes de producao.
