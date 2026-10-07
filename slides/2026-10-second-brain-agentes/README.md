# 🚀 Apresentação: O Exocórtex do Engenheiro (Casa de IA)

Material de apresentação em **Reveal.js 5.x** abordando teoria do Second Brain, panorama de mercado, simbiose com agentes de IA e a arquitetura determinística do Daneel.

---

## 💻 Como Rodar Localmente

Para rodar a apresentação localmente, use qualquer servidor HTTP estático (Reveal.js precisa carregar o `slides.md` via fetch/XHR):

```bash
# Opção 1: Python 3 nativo
python3 -m http.server 8000 --directory public/slides/2026-10-second-brain-agentes

# Opção 2: Node.js (npx serve)
npx serve public/slides/2026-10-second-brain-agentes

# Opção 3: Se já estiver dentro da pasta:
cd public/slides/2026-10-second-brain-agentes && python3 -m http.server 8000
```

Abra no navegador em: `http://localhost:8000`

---

## ⌨️ Atalhos de Apresentação

| Tecla | Ação |
| :--- | :--- |
| `Espaço` / `→` | Próximo slide |
| `←` | Slide anterior |
| `S` | **Modo Apresentador** (Speaker Notes com cronômetro e preview do próximo slide) |
| `F` | Alternar tela cheia (Fullscreen) |
| `ESC` / `O` | Visão geral de todos os slides em grade (Overview mode) |
| `?` | Exibir todos os atalhos do Reveal.js |

---

## 🛡️ Firewall de Privacidade
Este material é **100% público** e projetado para ser agnóstico a sistemas internos ou dados proprietários.
Auditado via `daneel guard` sob as Leis da Robótica do Second Brain.
