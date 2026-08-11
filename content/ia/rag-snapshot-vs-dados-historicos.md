---
type: arquitetura
description: Análise de falhas em sistemas RAG e agentes de IA causadas por datasets de snapshot estáticos em vez de ingestão temporal/histórica.
tags:
  - notes/ia
  - notes/arquitetura
---

# RAG: Snapshot Estático vs Ingestão Histórica Incremental

## Visão Geral
Em aplicações de IA generativa (RAG e Agentes autônomos), fornecer um dataset baseado em substituição por snapshot estático (`snapshot replace`) impede o agente de responder a perguntas que dependam da evolução temporal de dados ou histórico.

## Lições Aprendidas (PoC Agente CSM)
- **Sintoma:** O agente de IA responde incorretamente a perguntas de usuários mesmo com dados sincronizados.
- **Causa Raiz:** O pipeline de sincronização sobrescrevia o dataset apenas com o snapshot atual dos dados, eliminando o histórico de interações e métricas passadas.
- **Diretriz de Arquitetura:**
  1. Se a intenção do agente envolver análises de evolução, tendências ou comparativos de tempo, a pipeline de dados deve alimentar um banco vetorial ou data store com **registros temporais e versionados**.
  2. Automação de sync deve ser gerenciada por orquestradores/pipelines contínuas (ex: cron jobs, n8n, Airflow) em vez de scripts manuais em máquinas locais.

## Conexões e Referências
- **Documentação de Sistemas Legados via IA**
