---
type: design-pattern
description: Padrão de design Namespace Facade em Go para construção de SDKs modulares e sustentáveis.
tags:
  - notes/go
  - notes/design-pattern
---

# Go: SDK Namespace Facade Pattern

## Visão Geral
Ao construir SDKs corporativos em Go consumidos por múltiplos times, expor dezenas de funções soltas no pacote raiz gera confusão na autoconclusão da IDE e acoplamento desnecessário.

## Estrutura do Padrão
O padrão **Namespace Facade** instancia um único cliente principal (`Client`) que expõe sub-namespaces temáticos para operações de domínio:

```go
// Uso fluente, intuitivo e agrupado por domínio
res, err := cli.Invoices().Search(ctx, query)
user, err := cli.Users().FindByID(ctx, userID)
```

## Benefícios
1. **Navegabilidade e Autocompletion:** O desenvolvedor descobre as APIs de cada domínio digitando `cli.<Dominio>().` na IDE.
2. **Manutenibilidade:** Cada sub-namespace vive em um arquivo ou pacote separado sem sobrecarregar a struct do cliente raiz.
3. **Testabilidade & Mocking:** Permite mockar namespaces individuais através de interfaces pequenas, desacopladas e atômicas.

## Exemplo de Implementação Idiomática

```go
type Client struct {
    httpClient *http.Client
    baseURL    string
}

func (c *Client) Invoices() *InvoiceService {
    return &InvoiceService{client: c}
}

func (c *Client) Users() *UserService {
    return &UserService{client: c}
}
```
