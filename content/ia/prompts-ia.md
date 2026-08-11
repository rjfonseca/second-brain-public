---
type: tecnologia
title: Prompts Estruturados para Desenvolvimento com IA
description: "Padrões de prompts eficazes para revisão de documentação, orquestração multi-agente e desenvolvimento autônomo com IA."
tags:
  - notes/ia
  - notes/prompts
---

# 📝 Prompts Estruturados para Engenharia com IA

Coleção de prompts testados e estruturados para acelerar fluxos de trabalho de engenharia de software, revisão de documentação e orquestração de times de agentes autônomos.

---

## 🔍 1. Revisão Sistemática de Documentação vs. Código

Utilize este prompt para garantir que a documentação técnica em `docs/` reflita com precisão o estado real do código após refatorações.

```text
Revise toda a documentação em docs/ contra a implementação atual do código.
Para cada arquivo .md, verifique:
1. Referências a caminhos de diretório/arquivo que não existem mais
2. Nomes de pacotes, serviços ou handlers que mudaram
3. Links quebrados (internos e externos)
4. Texto que não pertence à documentação (preambles de conversa com IA,
   frases como "Aqui está o documento finalizado", etc.)
5. Descrições de comportamento que divergem do código atual

Regras:
- Corrija o que estiver errado em relação ao estado atual do código
- Se algo descreve funcionalidade que ainda não existe mas está planejada,
  deixe como está e me avise
- Não adicione conteúdo novo, apenas corrija o existente
- Liste o que encontrou e o que corrigiu ao final
```

### Variações Úteis

#### Modo Somente Relatório (Sem Alterar Arquivos)
```text
Não altere nenhum arquivo. Produza um relatório detalhado com:
- Arquivo
- Linha
- Problema encontrado
- Sugestão de correção
```

#### Focado no Diff da Branch
```text
Revise a documentação em docs/ que pode ter sido afetada pelas mudanças nesta branch (compare com git diff main..HEAD --name-only para contexto). Corrija inconsistências entre docs e código.
```

---

## 🤖 2. Orquestração Multi-Agente para Desenvolvimento Autônomo

Este blueprint define a estrutura de contexto, regras e papéis para delegar o desenvolvimento ponta-a-ponta de um serviço a um time de subagentes autônomos.

```text
# Contexto e Objetivo
Precisamos construir um componente de roteamento e ingestão de eventos para desacoplar a arquitetura legada:
1. Event Router: consome mensagens de um tópico e as roteia para o tópico de destino.
2. Event Syncer: envia mensagens para a API analítica de eventos para persistência no Data Warehouse.

# Papéis do Time de Agentes
- Lead Agent: Orquestra o ciclo de vida, tira dúvidas e valida critérios de aceite.
- Architect Agent: Analisa especificações, resolve trade-offs e gera o plano de implementação detalhado.
- Developer Agent: Implementa o código em Go seguindo o plano passo a passo e princípios Clean Architecture.
- QA Agent: Cria e executa testes de aceitação e integração ponta a ponta (via Docker Compose).

# Regras Operacionais
- Cada tarefa deve passar obrigatoriamente por: Especificação -> Implementação -> Testes Automatizados.
- Se um agente encontrar impedimentos ou falta de diretrizes, deve solicitar clarificação imediatamente.
- A tarefa só é considerada concluída quando todos os testes de aceitação passarem com sucesso.

# Critérios de Aceite
- Testes de aceitação automatizados cobrindo o fluxo completo.
- Build reproduzível via Dockerfile e Docker Compose.
- Logs estruturados e métricas de telemetria configuradas.
```

---

## 🏗️ 3. Refatoração Segura e Clean Architecture

Prompt para guiar agentes em refatorações de código legado aplicando o padrão Strangler Fig e separação de responsabilidades.

```text
Você é um Staff Software Engineer especialista em refatoração e Clean Architecture.
Analise o módulo selecionado e elabore um plano de refatoração para:
1. Isolar regras de negócio puras (domínio) de bibliotecas de terceiros (infraestrutura).
2. Introduzir interfaces atômicas para facilitar injeção de dependências e testes com mocks.
3. Garantir compatibilidade regressiva de APIs públicas.

Antes de editar qualquer código:
- Apresente um sumário executivo com os trade-offs da mudança.
- Aguarde minha confirmação para prosseguir com a implementação.
```
