# CP05 - migracao-forge-event-driven em cadeia

O prompt foi separado em tres elos independentes, e a saida de cada etapa passa a ser a entrada real da seguinte.

O exercicio nao pede apenas separacao estrutural. Ele pede uma cadeia em que o diagnostico alimenta a estrategia, a estrategia alimenta o plano e cada elo registra sua propria saida de forma reutilizavel.

## Intencao do CP05

- Evitar um prompt monolitico que tenta resolver diagnostico, estrategia e plano ao mesmo tempo.
- Fazer a analise amadurecer em camadas.
- Preservar rastreabilidade entre entrada, saida e proxima etapa.
- Registrar o output de cada elo de forma clara para auditoria e reuso.

## Elo 1 - Diagnostico

Entrada usada: descricao do Forge atual, dependencias e garantias exigidas.

```text
ELO 1 - DIAGNOSTICO:
O Forge hoje roda em batch a cada 60min via job forge-batch-ingest. Cada execucao passa por 14 etapas Spark e leva cerca de 40min, gravando tabelas horarias no warehouse. A fragilidade principal e o acoplamento temporal: se um lote falha ou atrasa, o volume seguinte acumula e amplia a pressao operacional. Sentinel, Cerebro e billing dependem dessas tabelas particionadas por hora, entao a compatibilidade de contrato e um invariante.

LACUNAS:
- Falta medir volume real por janela, taxa de late events e custo de reprocessamento.
- Falta confirmar quais consumidores conseguem ler simultaneamente batch e event-driven.

ENTRADA PARA O ELO 2:
Forge opera hoje em batch horario com 14 etapas Spark, depende de tabelas particionadas por hora e nao pode quebrar Sentinel, Cerebro nem billing durante a transicao. A migracao precisa manter compatibilidade de contrato, tratar o acoplamento temporal como risco principal e reconhecer lacunas de volume, late events e leitura dual.
```

Observacao:
- Este elo precisa capturar o estado atual com precisao suficiente para sustentar decisao posterior.
- O valor dele nao e sugerir solucao, e sim produzir base confiavel para a estrategia.

## Elo 2 - Estrategia

Entrada usada: saida real do Elo 1.

```text
ELO 2 - ESTRATEGIA:
A melhor abordagem e uma migracao por dual-run. O Forge deve consumir o Relay continuamente em microbatches enquanto o batch atual continua sendo a fonte oficial. Em paralelo, a nova trilha event-driven grava saidas equivalentes em tabelas paralelas, mantendo a particionagem horaria. Sentinel e Cerebro podem migrar por feature flag depois da reconciliacao, enquanto billing permanece no batch ate o fechamento de consistencia.

PONTOS DE REVERSAO:
- Desligar o consumidor novo e voltar a ler apenas as tabelas batch.
- Manter o batch como fonte oficial ate que checksums, volume e atraso estejam equivalentes.
- Interromper a migracao de consumidores se houver divergencia de schema ou ordering.

ENTRADA PARA O ELO 3:
A estrategia escolhida e dual-run: consumir o Relay continuamente em microbatches, escrever saida paralela compatível por hora, comparar com batch antes de promover a nova trilha e manter billing no batch ate o fechamento de consistencia. O rollback precisa permitir voltar a leitura batch sem perda de continuidade, e a migracao de consumidores depende de parity de schema, volume, atraso e checksums.
```

Observacao:
- Aqui a saida deve mostrar porque a estrategia incremental e a melhor resposta para o contexto.
- Alternativas nao precisam virar tese longa, mas devem aparecer o suficiente para justificar a escolha.

## Elo 3 - Plano executavel

Entrada usada: saida real do Elo 2.

```text
ELO 3 - PLANO EXECUTAVEL:
1. Definir o contrato de evento e as regras de idempotencia.
2. Criar o consumidor Forge event-driven em shadow mode, sem publicar como fonte oficial.
3. Gravar a saida paralela com a mesma particao horaria do batch atual.
4. Rodar reconciliacao batch vs event-driven por volume, atraso e checksums.
5. Migrar Sentinel por feature flag depois da consistencia da nova trilha.
6. Migrar Cerebro apenas apos estabilidade de schema e ausencia de divergencia.
7. Manter billing por ultimo, com validacao noturna antes do corte final.

ROLLBACK:
Desligar o consumidor novo, cancelar a promocao por feature flag e apontar os dependentes para as tabelas batch ate a reconciliacao voltar a ficar verde.

RISCOS RESIDUAIS:
- Duplicidade de eventos.
- Ordering inconsistente em eventos atrasados.
- Divergencia de agregacao entre batch e microbatch.
- Custo adicional do dual-run ate o corte definitivo.

CRITERIOS DE SUCESSO:
- Paridade de volume entre batch e event-driven.
- Atraso dentro do limite acordado para Sentinel e consumidores dependentes.
- Checksums e reconciliacao sem divergencias materiais por janela.
- Billing validado no ciclo noturno antes de abandonar o batch.
```

Observacao:
- Este elo precisa ser o mais operacional dos tres.
- Se o plano nao puder virar backlog ou runbook, ele ainda esta abstrato demais.

## O que foi reforcado nesta versao

- Elo 1 ganhou inventario de invariantes, lacunas e criterios para alimentar o proximo passo.
- Elo 2 ganhou exigencia de analise de alternativas, pontos de reversao e premissas.
- Elo 3 ganhou estrutura de fases, criterios observaveis e resumo final pronto para execucao.
