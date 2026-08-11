---
type: arquitetura
description: Padrão EventBus para comunicação desacoplada e troca de estados entre aplicações Micro-Frontend.
tags:
  - notes/arquitetura
  - notes/frontend
---

# EventBus em Arquitetura de Micro-Frontends

## Visão Geral
Em arquiteturas de **Micro-Frontends** (MFE), múltiplos módulos ou aplicações frontend independentes rodam no mesmo navegador. Para evitar acoplamento direto ou dependência de estado global único, a comunicação entre micro-frontends utiliza o padrão **EventBus**.

## Princípios de Funcionamento
- **Publish-Subscribe (Pub/Sub):** Um MFE dispara um evento (ex: `payment:completed`) sem conhecer quem responderá a ele.
- **Desacoplamento de Framework:** Permite que um MFE construído em React se comunique com um MFE em Vue ou Vanilla JS através do barramento de eventos customizado (`CustomEvent` da Web API ou EventBus dedicado).
- **Contrato de Eventos:** Os esquemas de carga (*payload*) dos eventos devem ser fortemente tipados e documentados para evitar regressões na integração.

## Conexões e Referências
- Referência: [Eventos entre Microfrontends (Isaac Gomes Matos)](https://www.linkedin.com/posts/isaac-gomes-matos_eventos-entre-microfrontends-ugcPost-7406796781818388480-vV3a/)
- **Experiência Integrada: Jornada do Cliente**
