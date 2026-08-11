---
type: arquitetura
description: Diretrizes de arquitetura para consumo de APIs públicas e filas de eventos em ambientes multi-tenant, evitando o problema de Noisy Neighbor.
tags:
  - notes/arquitetura
  - notes/resiliencia
---

# Consumo de APIs Multi-tenant e Prevenção de Noisy Neighbor

## Visão Geral
Em sistemas que consomem eventos ou fazem polling em APIs públicas em nome de múltiplos clientes (tenants), a arquitetura do worker de consumo deve evitar que uma única conta grande monopolize a capacidade de processamento (problema do *Noisy Neighbor*).

## Requisitos de Arquitetura de Fila e Worker

### 1. Seleção e Inicialização de Contas (Tenant Bootstrapping)
- O worker **nunca** deve assumir um `account_id` estático ou pré-existente sem um plano claro de inicialização.
- A ordenação da fila de sincronização deve garantir que novas contas ou contas sem processamento recente tenham prioridade justa.

### 2. Tratamento de Paralelismo e Isolamento
- **Fair Queueing / Partitioning por Tenant:** Distribuir tarefas em filas divididas por tenant ou usar limites de concorrência por `account_id`.
- **Throttling & Backoff:** Evitar que estouro de rate limit de uma conta em API externa bloqueie o consumo de outras contas na mesma fila.

## Conexões e Referências
- **Experiência Integrada: Jornada do Cliente**
