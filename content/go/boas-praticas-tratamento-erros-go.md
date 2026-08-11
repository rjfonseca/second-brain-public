---
type: tecnologia
description: Boas práticas e mandamentos para tratamento elegante e idiomático de erros na linguagem Go.
tags:
  - notes/go
  - notes/boas-praticas
---

# Boas Práticas e Tratamento de Erros em Go

## Visão Geral
Em Go, erros não são exceções disparadas na pilha de execução; erros são valores de primeira classe retornados explicitamente pelas funções (`err error`).

## Princípios de Tratamento Idiomático

1. **Evitar Erros Silenciosos:** Nunca ignore o retorno de erro com `_` sem justificativa válida.
2. **Wrapping Contextual:** Adicione contexto descritivo ao erro antes de propagá-lo para cima usando `fmt.Errorf("falha ao processar ordem %s: %w", orderID, err)`.
3. **Inspeção via `errors.Is` e `errors.As`:** Em vez de comparar strings com `err.Error()`, utilize `errors.Is` para verificar tipos de erro sentinel e `errors.As` para extrair tipos de erro customizados.
4. **Tratar uma Única Vez:** Evite logar o erro e também retorná-lo para a função chamadora (o que gera logs duplicados). Logue o erro apenas no topo do manipulador ou retorne-o envolvido em contexto.

## Conexões e Referências
- Referência: [10 Golang Error Handling Commandments](https://preslav.me/2026/05/19/10-golang-error-handling-commandments/)
- [SDK Namespace Facade Pattern](sdk-namespace-facade-pattern.md)
