---
title: Golang Hub
description: "Guia completo, referências de runtime, concorrência, otimização de memória e padrões de design em Go."
tags:
  - golang
  - go
  - engenharia
---

# 🐹 Golang Knowledge Hub

Bem-vindo ao acervo de anotações técnicas e arquiteturais sobre **Go (Golang)**. Este repositório reúne conceitos de runtime, mecânicas de Garbage Collector, profiling de performance, padrões de SDKs e melhores práticas idiomáticas.

---

## 🧠 1. Runtime, Memória & Concorrência
- [Golang Core](golang.md) — Conceitos fundamentais e visão geral do ecossistema Go.
- [Golang Memory Management](golang-memory-management.md) — Estrutura de memória (heap, stack, mcache, spans) e ciclo de vida do Garbage Collector.
- [Golang Memory Arenas](golang-memory-arenas.md) — Alocação de memória em arenas para minimizar a pressão sobre o GC.
- [Golang Struct Padding](golang-struct-padding.md) — Alinhamento de memória e ordenação de structs para otimização de cache lines.
- [Cache in Golang](cache-in-golang.md) — Estratégias de cache multi-backend e in-memory em Go.

---

## ⚡ 2. Performance, Profiling & Benchmarks
- [Profiling in Go](profiling-in-go.md) — Diagnóstico de CPU, heap, goroutines e contenção de mutexes com `pprof`.
- [Benchmarks in Go](benchmarks-in-go.md) — Criação de benchmarks e medição estatística com o pacote `testing.B`.
- [pkgsite CLI Tooling](pkgsite-cli-tooling.md) — Servidor local e documentação de pacotes Go.

---

## 🛠️ 3. Padrões de Design, Idiomas & SDKs
- [Boas Práticas e Tratamento de Erros em Go](boas-praticas-tratamento-erros-go.md) — Padrões modernos com `errors.Is`, `errors.As` e wrapping (`%w`).
- [SDK Namespace Facade Pattern](sdk-namespace-facade-pattern.md) — Estruturação de clientes SDK com namespaces fluentes.
- [Go Design Patterns](go-design-patterns.md) — Padrões de projeto adaptados para a semântica e simplicidade de Go.
- [Functional Options](functional-options.md) — Padrão de opções funcionais para inicialização extensível de structs.
- [Handling Errors in Go](handling-errors-in-go.md) — Evolução histórica e ergonomia de tratamento de erros.
- [Go Doc Comments](go-doc-comments.md) — Diretrizes canônicas para documentação de pacotes e símbolos.

---

## 🧪 4. Testes, Tooling & Ecossistema
- [Golden Files](golden-files.md) — Testes de snapshot e comparação de saídas com flags de update.
- [Delve Debugger](delve.md) — Uso do `dlv` para depuração de processos locais e containers.
- [Go Modules](go-modules.md) — Gerenciamento declarativo de dependências no Go.
- [Golang Modules Versioning](golang-modules-versioning.md) — Versionamento semântico e compatibilidade major.
- [Google Cloud Go Release Format](google-cloud-go-release-format.md) — Formato de releases de SDKs de nuvem.
- [Beam Go SDK](beam-go-sdk.md) — Pipelines de processamento distribuído com Apache Beam em Go.
- [Web Apps in Go](web-apps-in-go.md) — Frameworks e bibliotecas para serviços web.
- [goawk](goawk.md) — Interpretador AWK de alto desempenho implementado em Go.
- [Golang Scripting](golang-scripting.md) — Automação de scripts com execução direta em Go.
