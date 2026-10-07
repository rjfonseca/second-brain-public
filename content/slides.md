---
title: 🎤 Apresentações & Slides
type: moc
description: Catálogo público de palestras técnicas, apresentações e decks interativos em Reveal.js sobre Engenharia de Software, Arquitetura e Inteligência Artificial.
---

# 🎤 Palestras & Apresentações Técnicas

Este espaço reúne os decks de apresentação e palestras técnicas ministradas por **Rodrigo Fonseca**. Cada apresentação é construída com [Reveal.js](https://revealjs.com/), oferecendo navegação dinâmica, suporte a slides verticais, visualização de código e modo de anotações do apresentador.

---

## 🧠 O Exocórtex do Engenheiro: Second Brain na Era dos Agentes Autônomos

Apresentação técnica sobre a evolução dos métodos de gestão de conhecimento pessoal (PKM) e a sua transformação em memória viva para agentes autônomos de IA.

* **Fórum:** Casa de IA (Outubro de 2026)
* **Formato:** Apresentação interativa em Reveal.js (19 slides) + Live Demo no terminal
* **Duração:** 30 minutos de apresentação conceitual + debate aberto

👉 **[🚀 Abrir Apresentação em Tela Cheia (Reveal.js) ↗](https://rjfonseca.github.io/second-brain-public/slides/2026-10-second-brain-agentes/)**

> [!TIP]
> **Dica de Navegação:** Dentro do deck, pressione **`S`** para abrir a janela do apresentador com cronômetro e notas de fala, ou **`F`** para tela cheia.

### 📌 Resumo dos Tópicos Abordados

1. **A Crise da Sobrecarga Cognitiva:**
   * Fragmentação do foco de engenharia entre dezenas de ferramentas efêmeras.
   * O cérebro biológico como processador de ideias, não banco de dados de retenção.
2. **Fundamentos Teóricos & O Paradoxo do Mercado:**
   * O método Zettelkasten de Niklas Luhmann (pensamento em rede vs hierarquias em árvore).
   * O framework CODE & PARA de Tiago Forte.
   * A teoria da Mente Estendida (Clark & Chalmers) e o conceito de Exocórtex.
   * Por que ferramentas tradicionais (Notion, Obsidian) frequentemente se tornam "cemitérios de Markdown".
3. **A Virada de Chave dos Agentes de IA:**
   * A transição da nota passiva para a memória de trabalho de agentes autônomos.
   * Por que LLMs sozinhas alucinam sem uma base determinística de *ground truth*.
   * A arquitetura simbiótica: **Músculo Determinístico (Go / SQLite FTS5 < 3ms)** vs **Córtex Cognitivo (LLMs / Agentes)**.
   * O protocolo MCP (Model Context Protocol) como interface universal.
   * O *Dream Cycle* (sono REM digital combatendo a entropia e quebra de links).
4. **O Sistema Daneel no Dia a Dia de Staff Engineer:**
   * Quatro interfaces integradas: CLI, TUI em Charm Bubble Tea, Web 3D (Galáxia de grafos) e servidor MCP.
   * Briefing matinal com separação entre Fronteira e Neblina de Guerra (*Fog of War*).
   * Telemetria quantitativa de impacto (`[A]` a `[E]`) e prevenção de *Staff Drift*.
   * Preparação de reuniões em 2 segundos (`daneel prep`).
   * A skill `second-brain` guiando agentes de IA em outros repositórios corporativos sob a Lei da Busca Prévia.

---

## 🛠️ Como Executar os Slides Offline

Se você clonou o repositório público, pode rodar qualquer apresentação localmente utilizando um servidor HTTP simples:

```bash
# A partir da raiz do repositório:
python3 -m http.server 8000 --directory slides/2026-10-second-brain-agentes
```

Abra no navegador em `http://localhost:8000`.
