---
type: tecnologia
description: Uso da API e CLI oficial pkgsite para consulta de documentação e metadados de pacotes Go na criação de ferramentas técnicas.
tags:
  - notes/go
  - notes/ferramentas
---

# Go pkgsite CLI e API

## Visão Geral
Os mantenedores da linguagem Go disponibilizam a API e CLI do `pkgsite`, permitindo inspecionar metadados, documentação de pacotes, dependências e versões de módulos diretamente via linha de comando ou chamadas HTTP.

## Aplicações Práticas em Tooling
- **Auditoria de Dependências:** Verificar licenças e versões de pacotes em pipelines de CI/CD.
- **Geradores de Código & Linters:** Consultar símbolos e assinaturas de pacotes externos na construção de linters internos ou geradores de SDK (ex: `qv` CLI).

## Conexões e Referências
- Documentação oficial: [go.dev/blog/pkgsite-api](https://go.dev/blog/pkgsite-api)
- [SDK Namespace Facade Pattern](sdk-namespace-facade-pattern.md)
