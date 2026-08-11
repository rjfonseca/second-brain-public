---
type: tecnologia
description: "Anotações técnicas e referências sobre Structured-Prompt-Driven Development (SPDD)."
tags:
  - notes/ia
---

# Structured-Prompt-Driven Development (SPDD)

Link original: https://martinfowler.com/articles/structured-prompt-driven/

## O que é SPDD?
**Structured-Prompt-Driven Development (SPDD)** é uma metodologia de engenharia de software desenvolvida pela Thoughtworks que trata os prompts de IA como artefatos de entrega de primeira classe, mantidos sob controle de versão (Git) ao lado do código-fonte.

Diferente do uso ad-hoc de LLMs (chats informais e descartáveis), o SPDD estabelece um método estruturado e governável para que equipes colaborem usando IA de forma previsível, auditável e reutilizável.

## Componentes Principais

### 1. O Canvas REASONS
Uma estrutura de 7 dimensões que orienta a criação do prompt desde a intenção de negócio até as salvaguardas técnicas:
- *R — Requirements (Requisitos):* O problema de negócio e a Definition of Done (DoD).
- *E — Entities (Entidades):* Entidades do domínio e seus relacionamentos.
- *A — Approach (Abordagem):* A estratégia geral de design (ex: padrões de projeto, decisões de arquitetura).
- *S — Structure (Estrutura):* Onde a mudança se encaixa no sistema (camadas, classes, dependências).
- *O — Operations (Operações):* Detalhamento dos passos de execução (assinaturas de método, algoritmos).
- *N — Norms (Normas):* Padrões de código gerais (estilo, logs, tratamento de erro).
- *S — Safeguards (Salvaguardas):* Limites não-negociáveis (invariantes, limites de performance, segurança).

### 2. O Fluxo de Trabalho (Workflow)
O fluxo do SPDD garante que a especificação e o código evoluam juntos, evitando o descompasso (drift):
- *Correções Lógicas:* Mudou a regra de negócio ou encontrou um bug? *Atualize o prompt primeiro*, depois gere o código (`requirements -> prompt -> code`) usando comandos como `/spdd-prompt-update` e `/spdd-generate`.
- *Refatoração:* Melhorias na estrutura interna ou estilo de código sem alterar comportamento externo? *Refatore o código primeiro*, depois sincronize a especificação (`code -> prompt`) usando o comando `/spdd-sync`.

## Exemplo de Uso
O artigo ilustra a aplicação do SPDD na evolução de um motor de faturamento de tokens de LLM (*Billing Engine*):
1. *Criação da Story:* Quebra da necessidade de negócio em histórias INVEST (`/spdd-story`).
2. *Análise do Código:* Varredura inteligente das dependências do sistema (`/spdd-analysis`).
3. *Geração do Canvas:* Criação da especificação técnica baseada em REASONS (`/spdd-reasons-canvas`).
4. *Geração de Código:* Criação das classes de faturamento (usando o padrão /Strategy/ para separar as regras de planos "Standard" e "Premium" com base no `modelId`) (`/spdd-generate`).
5. *Validação:* Escrita automatizada de testes funcionais cURL (`/spdd-api-test`) e geração de testes unitários baseados em templates.

## Onde pode ser usado (Fitness Assessment)
O SPDD é recomendado dependendo do cenário de desenvolvimento:
- *Altamente Recomendado (5 estrelas):*
  - Desenvolvimento escalável e padronizado (ex: criar APIs semelhantes repetidamente).
  - Sistemas com alta conformidade, regras regulatórias ou restrições de segurança estritas.
  - Ambientes de colaboração multi-pessoas onde a rastreabilidade é crucial.
- *Baixa recomendação / Não recomendado (2 estrelas ou menos):*
  - Hotfixes urgentes de produção (primeiro corrige para parar o sangramento, depois documenta).
  - Spikes exploratórios ou prototipagem rápida.
  - "Buracos negros de contexto" (domínios com regras de negócio obscuras ou indefinidas).

## Habilidades Essenciais
Para ter sucesso com SPDD, o desenvolvedor precisa focar em três habilidades:
1. *Abstração Primeiro (Abstraction First):* Projetar as entidades e interações antes de pedir para a IA gerar código.
2. *Alinhamento (Alignment):* Firmar o escopo ("o que faremos" e "o que não faremos") e restrições antes do teclado rodar.
3. *Revisão Iterativa (Iterative Review):* Gerenciar a IA em um ciclo fechado e controlado de pequenos passos.

## Conexões neste Second Brain
- **SOLID**: O SPDD apoia a aplicação estrita de princípios como SRP e ISP no design de abstrações antes da geração de código.
- **Code Review**: O processo de revisão sob o SPDD foca na intenção declarada no Canvas, reduzindo o tempo gasto em discussões estéticas ou micro-bugs.
- [Prompts (IA)](prompts-ia.md): Aplicação prática e metodológica avançada de engenharia de prompts na engenharia de software diária.
- **Documentation**: Trata os prompts como documentação ativa, viva e versionada da arquitetura.
- **Technical Debt**: Evita o desvio lógico entre a documentação de arquitetura e o código real, mitigando a geração de débito técnico acidental por IAs generativas.
