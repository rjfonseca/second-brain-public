---
type: tecnologia
description: "AI Overview RAG, ou Geração Aumentada de Recuperação, é uma técnica de inteligência artificial que aprimora modelos de linguagem grandes ..."
tags:
  - notes/ia
---

# Retrieval-Augmented Generation (RAG)

## Retrieval-Augmented Generation (RAG)

### O que é

AI Overview

RAG, ou Geração Aumentada de Recuperação, é uma técnica de inteligência artificial que aprimora modelos de linguagem grandes (LLMs) ao combinar suas capacidades de geração com sistemas de recuperação de informações externas. Essencialmente, o RAG permite que um LLM acesse e incorpore dados atualizados e específicos de fontes externas, como bancos de dados ou documentos, antes de gerar uma resposta, melhorando a precisão e o contexto das respostas. 
Como funciona o RAG:

1. Recuperação:
   Uma consulta do usuário é enviada a um sistema de recuperação de informações, que busca dados relevantes em fontes externas.
2. Aumento:
   As informações recuperadas são combinadas com a consulta original, criando um contexto mais amplo.
3. Geração:
   O LLM utiliza esse contexto expandido para gerar uma resposta mais precisa e informada. 

Benefícios do RAG:

    Atualização:
    O RAG permite que os LLMs acessem dados que podem não estar presentes em seus dados de treinamento originais, tornando-os mais atualizados. 

Contexto:
Ao incorporar dados externos, o RAG fornece mais contexto para as respostas, tornando-as mais relevantes e precisas. 
Precisão:
A recuperação de informações relevantes ajuda a reduzir a ocorrência de "alucinações" (respostas falsas ou sem sentido) por parte do LLM. 
Redução de custos:
O RAG pode reduzir a necessidade de retreinar modelos, economizando tempo e recursos computacionais. 
Confiança do usuário:
Com a atribuição de fontes, o RAG aumenta a confiança do usuário, pois as respostas podem ser rastreadas até suas fontes. 

Exemplos de aplicação:

    Chatbots de suporte ao cliente:
    O RAG pode ser usado para que chatbots forneçam respostas precisas a perguntas complexas, recuperando informações de bases de conhecimento extensas. 

Sistemas de resposta a perguntas:
O RAG pode melhorar a precisão e o contexto de respostas a perguntas sobre tópicos específicos, utilizando fontes de dados externas. 
Geradores de conteúdo:
O RAG pode ser utilizado para gerar conteúdo mais rico e preciso, incorporando informações de diversas fontes. 

Em resumo, o RAG é uma técnica poderosa que melhora significativamente o desempenho de LLMs, tornando-os mais precisos, atualizados e informados, além de reduzir a ocorrência de erros. 

### Links externos
- [Retrieval Augmented Generation (en.wikipedia.org)](https://en.wikipedia.org/wiki/Retrieval-augmented_generation)
- [Retrieval Augmented Generation (aws.amazon.com)](https://aws.amazon.com/pt/what-is/retrieval-augmented-generation/)
- [Rag (python.langchain.com)](https://python.langchain.com/docs/tutorials/rag/)
- [Rag Golang Postgresql Langchain (github.com)](https://github.com/build-on-aws/rag-golang-postgresql-langchain)
- [How To Use Rag In Bigquery To Bolster Llms (cloud.google.com)](https://cloud.google.com/blog/products/data-analytics/how-to-use-rag-in-bigquery-to-bolster-llms)
