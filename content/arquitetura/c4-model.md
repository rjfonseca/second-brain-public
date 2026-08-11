---
type: arquitetura
description: "Anotações técnicas e referências sobre C4 Model."
tags:
  - notes/arquitetura
---

# C4 Model

## What is the C4 Model
Modeling using the [C4 Model](https://c4model.com/) should be done in 4 levels:

- Context
- Container
- Components
- Code (actually, this one should be done by tooling or just ignored)

### Context
This is the starting point. This diagram shows the system in scope connecting to other systems around it.
### Container
This is a high-level technical model showing the building blocks of the system in scope.
### Component
This is a high-level breakdown of a single container, showing it's components and how the connect with each other and external agents.
### Code
This is the UML's class diagram of it's equivalent.
## Other references
There is also an [InfoQ article](https://www.infoq.com/articles/C4-architecture-model/) about it.
