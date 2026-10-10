# CLAUDE.md — popsmart

## O projeto

Popups e jornadas inteligentes (estilo Alia Popups) **open source, instaláveis e configuráveis pelo agente de código do próprio cliente** (Claude Code, Codex, Cursor), para HTML, Next.js e WordPress. Piloto: inema.club (agente de orientação que indica curso/aula). Conteúdo INEMA é aberto e gratuito; nada aqui é vendido.

## Estado (10/10/2026)

Só pesquisa e planejamento. **Não há código de produto.** Leia nesta ordem antes de agir: `README.md` → `PLANO.md` (fases, piloto, decisões em aberto) → `docs/0X-*.md` conforme o tema.

| Tema | Arquivo |
|---|---|
| Alia + concorrentes | `docs/01-pesquisa-mercado.md` |
| Repos open source e licenças | `docs/02-open-source-github.md` |
| Instalação por agente (CLI/MCP/skill/llms.txt) | `docs/03-instalacao-pela-ia.md` |
| Arquitetura, modelo de dados, stack, peso | `docs/04-arquitetura.md` |
| Triggering, A/B, bandit, métricas, guard-rails | `docs/05-algoritmos-e-metricas.md` |
| LGPD/GDPR, acessibilidade, SEO/CWV, dark patterns | `docs/06-conformidade.md` |

## Regras do repo

- **Git**: autor e committer `inematds <inematds@gmail.com>` (conferir `git config user.email` antes de commitar). Commit e push só quando o Nei pedir. Publicar = push; nunca consultar/cutucar Vercel.
- **Nome provisório**: "popsmart" colide com Popupsmart; não publicar pacote npm, domínio ou repo público com esse nome antes da decisão 1 do `PLANO.md §3`.
- **Sem API de modelo** (OpenAI, Anthropic API, OpenRouter, Groq…) e sem carregar chaves de `.env` sem autorização explícita do Nei. A API do GitHub (`gh`) está liberada. Geração de texto para o projeto = pela assinatura (Claude Code / `codex exec -m gpt-6-luna`).
- **Geração por IA no produto**: sempre (a) o agente do cliente via MCP (brief + schema, sem custo nosso) e (b) opcionalmente o provedor que o cliente configurar com a chave dele. Nunca assumir chave nossa.
- **Fatos vs inferência**: em qualquer doc, marcar `[fato: URL]` ou `[inferência]`/`[estimativa]`. Números de vendors são autodeclarados.
- **Licença**: tudo que vai para o site do cliente (widget, schema, engine, CLI, MCP, skill) é MIT; não copiar código de projetos AGPL/FSL (Typebot, intro.js, Shepherd, sentry-wizard) — só comportamento.
- **Conformidade é critério de pronto**: `validate`/`doctor` devem cobrir o checklist de 25 itens de `docs/06 §7` (acessibilidade do `<dialog>`, CLS 0, consentimento, sem dark patterns, sem roleta aleatória por padrão).
- **Peso** (gzip): loader ≤ 2 KB, loader+core (`popsmart.js`) ≤ 15 KB, total típico com config ≤ 20 KB (`docs/04 §6`).
- **Critérios de pronto** de cada fase são os `comando → saída esperada` do `PLANO.md`; não fechar fase "no olho".
- **Piloto inema.club**: o portal é `~/projetos/portal` (repo `NeiMaldaner/portal`, publicar com `git push origin`); mudanças lá seguem o `CLAUDE.md` do portal. Catálogo em `src/data/courses.data.json`.
- Saída de artefatos temporários (fixtures geradas, relatórios de teste): `~/projetos/output/popsmart/`.

## Self-learning

When I correct you, or you catch yourself making a mistake: before continuing, add the lesson as a one-line rule under ## Lessons, so it never happens again.

## Lessons
