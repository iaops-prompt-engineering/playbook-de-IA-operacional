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

## Metodo de criacao

- Prompt final parametrizavel, feito para receber manifesto barrado, padrao de seguranca e mapa de servicos.
- Curadoria feita sobre meta-prompting com loop generate -> critique -> refine.
- Escolha metodologica: registrar iteracoes explicitas v1 -> verificacao -> v2, porque artefato de seguranca nao deve sair na primeira tentativa.

Justificativa:
- O manifesto inicial e um allow-all disfarçado.
- Em NetworkPolicy, um detalhe pequeno muda muito o efeito real.
- Revisao estruturada ajuda a pegar falhas tipicas como DNS incompleto, seletor amplo demais e comentario faltando.

## Casos de uso

- Revisao de politicas Kubernetes antes de producao.
- Refino iterativo de artefatos de seguranca.
- Padronizacao de politicas por namespace e fluxo legitimo.

## Exemplo

Entrada: manifesto allow-all, padrao da Aegis e mapa de servicos.

Saida: v1, verificacao critica, v2 final e mudancas aplicadas.

## Contrato de saida

- `V1`: primeira versao endurecida.
- `VERIFICACAO`: perguntas e achados de revisao de seguranca.
- `V2 FINAL`: YAML corrigido.
- `MUDANCAS DA V2`: como cada achado foi tratado.

## Modelo recomendado

- Modelo principal: GPT-4o mini para custo baixo.
- Segundo provedor opcional: Claude Haiku para comparar clareza do YAML e da auto-critica.

## Limitacoes

- Selectores dependem da existencia real de labels nos namespaces.
- Namespace selectors por nome podem exigir label `kubernetes.io/metadata.name` disponivel no cluster.
- Deve ser validado com `kubectl apply --dry-run=server` antes de producao.
- Default-deny de egress tambem bloqueia DNS; e preciso liberar DNS explicitamente quando o workload depende dele.
