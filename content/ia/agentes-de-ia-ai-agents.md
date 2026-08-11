---
type: tecnologia
description: "Sistema projetado para perceber o ambientes e ajustar o seu comportamento de acordo. Agentes podem ser prõ-ativos na tomada de decisão."
tags:
  - notes/ia
---

# Agentes de IA (AI Agents)

## O que são?
Sistema projetado para perceber o ambientes e ajustar o seu comportamento de acordo.
Agentes podem ser prõ-ativos na tomada de decisão.

- [Agentes De Ia Conceito Arquitetura E Aplicacoes Parte 1 (cienciaedados.com)](https://www.cienciaedados.com/agentes-de-ia-conceito-arquitetura-e-aplicacoes-parte-1/)

## Tipos
### [Large Language Model (LLM)](large-language-model-llm.md)
Treinado com bilhões de dados. Capaz de realizar tarefas sofisticadas, mas mas lento e caro que uma SLM.
### Small Language Model (SLM)
Usado para contextos específicos. Mais limitado, mas mais rápido e barato.

## Vantagens sobre automação tradicional
Você da uma meta e o agente consegue se virar com os detalhes.
- Importante o agente ter autonomia, mas sempre com supervisão;
- Agentes de IA usam suas próprias respostas intermediárias para melhorar a resposta final;
- Agentes de IA podem fazer consultas a bancos de dados e web pra ter fontes de dados confiáveis;
- Não são realmente inteligentes, mas dão a impressão de ser.
## Benefícios
- Redução de custos;
  Realização de tarefas complexas com mais confiança. Redução dos erros manuais em processos.
- Aumento de produtividade;
  Automação de processos.
- Tomada de decisão mais assertiva;
  Ajuda a analisar melhor as demandas
- Melhor experiência do Cliente
  Personalização do produto para o cliente. Ponto polêmico pois se todas as empresas usa IA, o uso de IA deixa de ser um elemento competitivo.
## Desafios
- Problemas de privacidade de dados
- Preocupações éticas
- Complexidade técnica
## Arquitetura / Componentes
### Arquitetura
Como está organizado o seu código, de onde serão obtidos os dados. Onde serão salvos os dados produzidos.
### Função
Como os dados coletados são traduzidos. O que o agente vai fazer. É com que seja específico.
### Programa
É o programa do Agente em si.

## Componentes
### Sensores
Coleta dados do ambiente.
Conectores de bancos de dados, scrappers de web, etc
### Atuadores
Meios pelos quais o agente afeta o ambientes.
Ferramentas, que permitem coisas como criação de arquivos, escrita em banco ou acesso a APIs.
### Processadores
Analisa os dados e toma decisões.
Por exemplo, LLM.
## Como funciona um agente de IA
### Definir meta
Qual o objetivo do agente?
Agente cria tarefas para atingir a meta.
### Adquirir informações
Acesso a internet, interagir com outros agentes, acesso a banco.
### Implementar tarefas
Agente implementa metodicamente as tarefas. Ele pode criar mais tarefas se achar necessário. Ele avalia o resultado das tarefas a vai ajustando até atender a meta.
## Prompt-caching
Aumenta a eficiência e reduz custo dos agentes. Armazena prompt e respostas passadas. Se baseia em similaridade semântica.
## Reasoning
Dedução lógica usada pelo agente de IA
- Dedutivo: Se X->Y e Y->Z então X->Z. Ex: se todo pássaro bota ovo e um urubu é um pássaro, então urubu bota ovo.
- Indutivo: Se X1, X2, X3, então X4. Ex: Vi 100 cisnes e todos eram brancos, então todo cisne é branco.
- Abdução: Hipótese plausível: Ex Se vejo fumaça, presumo que pode haver fogo.
- Cadeia de pensamento (Chain-of-Thought - CoT): Explicita a linha de raciocínio enquanto responde.
- Multi-Agente: Vários agentes com diferentes especilizações colaboram no raciocínio.
