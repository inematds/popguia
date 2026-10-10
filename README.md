# popguia

[![popguia](guia/assets/banner.jpg)](https://inematds.github.io/popguia/guia/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## O que é

O popguia é um projeto open source para criar popups que conversam com o visitante: em vez de pedir o e-mail logo de cara, ele faz duas ou três perguntas, ensina algo útil e indica o produto, curso ou conteúdo certo. Serve para quem tem um site (HTML, Next.js, Astro, WordPress) e usa um agente de código como Claude Code, Codex ou Cursor: o agente instala, escreve as jornadas e acompanha os resultados. É grátis, com licença MIT, sem mensalidade e sem servidor nosso. Hoje o repositório tem a pesquisa e o plano; o código ainda vai ser escrito.

## 📖 Guia de uso

Guia completo (landing + passo a passo): **https://inematds.github.io/popguia/guia/**

**Popups e jornadas inteligentes que o agente de código do dono do site instala e configura sozinho.**

O dono de um site (HTML, Next.js, WordPress…) diz ao seu Claude Code / Codex / Cursor: *"instala o popguia e cria um quiz que recomende o curso certo"*. O agente instala o snippet, escreve as jornadas como arquivos JSON no repo, gera variantes, valida acessibilidade/LGPD/SEO e publica. Depois, o próprio sistema mede com grupo de controle, roda A/B e evolui para bandit — sem backend obrigatório nosso e sem cobrança por pageview.

Inspiração: Alia Popups (jornadas interativas, zero-party, Prism AI). Diferença: open source (MIT), multi-plataforma, estado em arquivos, instalação por agente, guard-rails por padrão.

## O que entrega

- **Jornadas**: diagnóstico/quiz educativo, Learn & Earn, microcompromissos, newsletter em 2 etapas, recomendação por catálogo.
- **Smart triggering**: tempo, scroll, exit intent (desktop/mobile), inatividade, clique, adaptativo; caps de frequência; cooldown global.
- **Personalização por contexto**: página, UTM, dispositivo, novo/recorrente, respostas anteriores (zero-party no `localStorage`).
- **Otimização**: holdout permanente → A/B (SRM, sequencial) → Thompson por segmento → contextual.
- **Instalação pela IA**: `npx popguia init`, MCP local (`npx -y @popguia/mcp`), `SKILL.md`, `setup.md`, `llms.txt`, prompt copiável.
- **Geração por IA sem custo nosso**: o MCP devolve brief + schema; o agente do cliente escreve; provedor opcional configurado pelo cliente (Ollama incluído).
- **Conformidade embutida**: `<dialog>` acessível, CLS 0, ≤ 20 KB gz, modos de consentimento, sem dark patterns, sem roleta aleatória por padrão.

## Estado

**Pesquisa e planejamento concluídos em 10/10/2026. Nenhum código de produto escrito.** Próximo passo: decisões do Nei (`PLANO.md §3`) e Fase 0.

## Documentos

| Arquivo | Conteúdo |
|---|---|
| [docs/00-referencia-alia.md](docs/00-referencia-alia.md) | Insumo inicial trazido pelo Nei |
| [docs/01-pesquisa-mercado.md](docs/01-pesquisa-mercado.md) | Alia em profundidade (entidades, gatilhos, reexibição, Prism) + 9 concorrentes (tabela) + reclamações |
| [docs/02-open-source-github.md](docs/02-open-source-github.md) | 37 repos com estrelas/licença/último push; veredito: construir do zero com referências |
| [docs/03-instalacao-pela-ia.md](docs/03-instalacao-pela-ia.md) | Estado da arte (shadcn, Sentry, PostHog, Supabase, Stripe, Vercel, Clerk…) e desenho CLI + MCP + skill + llms.txt |
| [docs/04-arquitetura.md](docs/04-arquitetura.md) | Componentes, diagrama mermaid, modelo de dados, stack, peso-alvo |
| [docs/05-algoritmos-e-metricas.md](docs/05-algoritmos-e-metricas.md) | Triggering, A/B com holdout, Thompson/LinUCB, volume mínimo, métricas, guard-rails |
| [docs/06-conformidade.md](docs/06-conformidade.md) | LGPD/GDPR, WCAG 2.2, Google interstitials, CWV, dark patterns, roleta (Lei 5.768/71); checklist de 25 itens |
| [PLANO.md](PLANO.md) | Fases 0–5 com critérios `comando → saída`, sites de demonstração, decisões fechadas e 7 em aberto |
| [CLAUDE.md](CLAUDE.md) | Instruções do repo para agentes |

## Demonstração

Não há piloto em site real: o popguia é para outras pessoas instalarem nos próprios sites. A validação usa 3 sites fictícios em `exemplos/` (escola em HTML, loja em Next.js, blog em Astro), publicados no GitHub Pages. Detalhes em `PLANO.md §2`.

## Licença

MIT, grátis, sem SaaS: cada usuário hospeda o próprio coletor (ou usa o analytics que já tem).
