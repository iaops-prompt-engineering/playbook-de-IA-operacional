---
nome: triagem-de-pods
descricao: Triagem de pods Kubernetes a partir de snapshot de get pods, describe e logs.
versao: 1.0.0
tags:
  - devops
  - kubernetes
  - sre
  - incident-response
inputs:
  - nome: snapshot_cluster
    descricao: Snapshot colado do cluster, contendo kubectl get pods, kubectl describe pod e logs relevantes.
  - nome: namespace
    descricao: Namespace avaliado no snapshot.
---

# triagem-de-pods

## Objetivo

Gerar uma triagem rapida e confiavel de pods Kubernetes problematicos a partir de um snapshot colado no prompt.

## Casos de uso

- Plantao SRE avaliando `CrashLoopBackOff`, `ImagePullBackOff` ou `Pending`.
- Passagem de turno com causa provavel e acao imediata.
- Revisao de incidente sem acesso direto do modelo ao cluster.

## Exemplo

Entrada: `snapshot_cluster` com `kubectl get pods`, `kubectl describe pod` e `kubectl logs --previous`.

Saida esperada: resumo, lista de pods problematicos, causa provavel, evidencias e acao do plantao.

## Limitacoes

- O prompt nao acessa o cluster. Ele depende integralmente do snapshot colado.
- Dados sensiveis como nomes de tenants, hosts internos e tokens em logs devem ser sanitizados antes de enviar a provedor externo.
- A causa e provavel, nao uma confirmacao definitiva.
