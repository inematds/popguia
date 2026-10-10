# popguia

[![popguia](guia/assets/banner-es.jpg)](https://inematds.github.io/popguia/guia/es/)

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

## Qué es

popguia es un proyecto open source para crear popups que conversan con el visitante: en vez de pedir el correo de entrada, hace dos o tres preguntas, enseña algo útil e indica el producto, curso o contenido adecuado. Sirve para quien tiene un sitio (HTML, Next.js, Astro, WordPress) y usa un agente de código como Claude Code, Codex o Cursor: el agente instala, escribe los recorridos y da seguimiento a los resultados. Es gratis, con licencia MIT, sin mensualidad y sin servidor nuestro. Hoy el repositorio tiene la investigación y el plan; el código todavía está por escribirse.

## 📖 Guía de uso

Guía completa (landing + paso a paso): **https://inematds.github.io/popguia/guia/es/**

**Popups y recorridos inteligentes que el agente de código del dueño del sitio instala y configura por sí solo.**

El dueño de un sitio (HTML, Next.js, WordPress…) le dice a su Claude Code / Codex / Cursor: *"instala popguia y crea un quiz que recomiende el curso adecuado"*. El agente instala el snippet, escribe los recorridos como archivos JSON en el repo, genera variantes, valida accesibilidad/LGPD/SEO y publica. Después, el propio sistema mide con grupo de control, corre A/B y evoluciona a bandit — sin backend obligatorio nuestro y sin cobro por pageview.

Inspiración: Alia Popups (recorridos interactivos, zero-party, Prism AI). Diferencia: open source (MIT), multiplataforma, estado en archivos, instalación por agente, guard-rails por defecto.

## Qué entrega

- **Recorridos**: diagnóstico/quiz educativo, Learn & Earn, microcompromisos, newsletter en 2 etapas, recomendación por catálogo.
- **Smart triggering**: tiempo, scroll, exit intent (desktop/móvil), inactividad, clic, adaptativo; caps de frecuencia; cooldown global.
- **Personalización por contexto**: página, UTM, dispositivo, nuevo/recurrente, respuestas anteriores (zero-party en `localStorage`).
- **Optimización**: holdout permanente → A/B (SRM, secuencial) → Thompson por segmento → contextual.
- **Instalación por IA**: `npx popguia init`, MCP local (`npx -y @popguia/mcp`), `SKILL.md`, `setup.md`, `llms.txt`, prompt copiable.
- **Generación por IA sin costo nuestro**: el MCP devuelve brief + schema; el agente del cliente escribe; proveedor opcional configurado por el cliente (Ollama incluido).
- **Cumplimiento integrado**: `<dialog>` accesible, CLS 0, ≤ 20 KB gz, modos de consentimiento, sin dark patterns, sin ruleta aleatoria por defecto.

## Estado

**Investigación y planificación concluidas el 10/10/2026. Ningún código de producto escrito.** Próximo paso: decisiones de Nei (`PLANO.md §3`) y Fase 0.

## Documentos

| Archivo | Contenido |
|---|---|
| [docs/00-referencia-alia.md](docs/00-referencia-alia.md) (en portugués) | Insumo inicial traído por Nei |
| [docs/01-pesquisa-mercado.md](docs/01-pesquisa-mercado.md) (en portugués) | Alia en profundidad (entidades, disparadores, re-exhibición, Prism) + 9 competidores (tabla) + quejas |
| [docs/02-open-source-github.md](docs/02-open-source-github.md) (en portugués) | 37 repos con estrellas/licencia/último push; veredicto: construir desde cero con referencias |
| [docs/03-instalacao-pela-ia.md](docs/03-instalacao-pela-ia.md) (en portugués) | Estado del arte (shadcn, Sentry, PostHog, Supabase, Stripe, Vercel, Clerk…) y diseño CLI + MCP + skill + llms.txt |
| [docs/04-arquitetura.md](docs/04-arquitetura.md) (en portugués) | Componentes, diagrama mermaid, modelo de datos, stack, peso objetivo |
| [docs/05-algoritmos-e-metricas.md](docs/05-algoritmos-e-metricas.md) (en portugués) | Triggering, A/B con holdout, Thompson/LinUCB, volumen mínimo, métricas, guard-rails |
| [docs/06-conformidade.md](docs/06-conformidade.md) (en portugués) | LGPD/GDPR, WCAG 2.2, Google interstitials, CWV, dark patterns, ruleta (Ley 5.768/71); checklist de 25 ítems |
| [PLANO.md](PLANO.md) (en portugués) | Fases 0–5 con criterios `comando → salida`, sitios de demostración, decisiones cerradas y 7 abiertas |
| [CLAUDE.md](CLAUDE.md) (en portugués) | Instrucciones del repo para agentes |

## Demostración

No hay piloto en un sitio real: popguia es para que otras personas lo instalen en sus propios sitios. La validación usa 3 sitios ficticios en `exemplos/` (escuela en HTML, tienda en Next.js, blog en Astro), publicados en GitHub Pages. Detalles en `PLANO.md §2`.

## Licencia

MIT, gratis, sin SaaS: cada usuario aloja su propio colector (o usa el analytics que ya tiene).
