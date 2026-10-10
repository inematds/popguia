# 03 — Instalação pela IA: como o agente do cliente instala e configura tudo

> Estado da arte lido em 10/10/2026 nas docs oficiais (links em cada item). O desenho proposto para o popguia é nosso `[inferência]`, construído em cima dos padrões verificados.

## 1. Estado da arte (o que os outros entregam ao agente)

| Produto | CLI que escreve código | MCP | Skill / plugin | Docs p/ máquina | Verificação | Fonte |
|---|---|---|---|---|---|---|
| **shadcn/ui** | `npx shadcn init` / `add` (com `--dry-run`, `--diff`) | local: `{"command":"npx","args":["shadcn@latest","mcp"]}` em `.mcp.json`; navega registries | página `/docs/skills` | — | `npx clear-npx-cache`, logs | https://ui.shadcn.com/docs/cli · https://ui.shadcn.com/docs/mcp |
| **Sentry** | `npx @sentry/wizard@latest -i nextjs` cria `instrumentation-client.ts`, `sentry.*.config.ts`, `global-error.tsx`; `--uninstall`, `--ignore-git-changes` | remoto OAuth `https://mcp.sentry.dev/mcp` (escopo por org/projeto na URL) | — | — | página `/sentry-example-page` com botão que dispara erro | https://docs.sentry.io/platforms/javascript/guides/nextjs/ · https://docs.sentry.io/product/sentry-mcp/ |
| **PostHog** | `npx -y @posthog/wizard@latest` (~25 frameworks; envolve o Claude Agent SDK; skills vêm do repo `context-mill`) | hospedado, `npx @posthog/wizard mcp add` | `claude plugin install posthog` | `posthog.com/llms.txt` | `wizard audit events\|autocapture\|feature-flags\|…` | https://posthog.com/docs/ai-engineering/ai-wizard · https://posthog.com/docs/model-context-protocol |
| **Supabase** | `supabase init` (só scaffold) | remoto OAuth `https://mcp.supabase.com/mcp?read_only=true&project_ref=…&features=…` | `npx skills add supabase/agent-skills`; plugin | — | "quais tabelas existem?" | https://supabase.com/docs/guides/getting-started/mcp · https://supabase.com/docs/guides/getting-started/ai-skills |
| **Stripe** | `stripe agent setup` detecta agentes instalados e configura plugin (MCP + skills com auto-update) | `https://mcp.stripe.com` (OAuth / agent keys; chaves sem tag Agent recusadas a partir de 31/10/2026) | `claude plugin install stripe@claude-plugins-official`, `npx skills add https://docs.stripe.com` | `docs.stripe.com/llms.txt` (92 KB) + páginas `.md` | botão "Copiar prompt"; `stripe sandbox create` | https://docs.stripe.com/mcp · https://docs.stripe.com/agents/plugin · https://docs.stripe.com/skills |
| **Vercel** | `vercel` CLI | `npx -y add-mcp https://mcp.vercel.com -g` | `npx plugins add vercel/vercel-plugin`; `npx skills add vercel-labs/agent-skills` | `vercel.com/llms.txt`, `llms-full.txt`, **`get-started.md`** = playbook para o agente ("Perform actions yourself… Pause only for user authentication") | `vercel whoami`, `/vercel-plugin:status` | https://vercel.com/docs/mcp/vercel-mcp · https://vercel.com/get-started.md |
| **Clerk** | `npx -y clerk@latest init` detecta framework + gerenciador de pacotes, cria app "claimable" sem conta | `clerk mcp install` (beta) | `clerk.com/SKILL.md` (manda mostrar checklist e esperar o "sim"; proíbe ler `.env`) | `llms.txt`, `/.well-known/agent-skills/index.json` | `clerk doctor` | https://clerk.com/SKILL.md · https://clerk.com/docs/cli.md |
| **Resend** | `npx resend-cli@latest login` | OAuth | `npx skills add resend/resend-skills` | **`resend.com/setup.md`** = prompt-playbook completo ("Aim to do the whole setup from this one prompt"; chave colada no chat ⇒ revogar) | — | https://resend.com/setup.md |
| **Neon** | `npx neon@latest init` configura MCP + skills + projeto numa vez | OAuth ("no static secret, so a project-level config is safe to commit") | — | — | — | https://neon.com/docs/ai/neon-mcp-server |
| **Better Auth** | `npx auth@latest init` (só Next.js/SQLite) | `npx auth@latest mcp --claude-code\|--cursor\|--codex` | — | `llms.txt` | — | https://better-auth.com/docs/concepts/cli.md |
| **Mintlify** | — | MCP de busca gerado | **`skill.md`** gerado por loop agêntico | `llms.txt`, `llms-full.txt`, menu "Copy page / Open in Claude/ChatGPT" | — | https://www.mintlify.com/docs/ai/skillmd.md |

**Padrões recorrentes** [fato: tabela acima]:
1. **Três camadas**: *skill/`.md`* ensina o agente → *CLI* escreve o código no repo → *MCP* opera a conta/estado. Stripe e Supabase empacotam tudo num plugin.
2. **Detecção de framework** é feita pelo CLI inspecionando `package.json`/lockfile/arquivos do projeto (Clerk, wizards com lista fixa); não há padrão aberto.
3. **Auto-verificação**: página de teste (Sentry), `audit` (PostHog), `doctor` (Clerk), `whoami` (Vercel).
4. **Credenciais**: OAuth como padrão; conta provisória "claimable" (Clerk), sandbox sem cadastro (Stripe); nunca ler `.env` (Clerk).
5. **Docs**: `llms.txt` (spec: https://llmstxt.org/ — só um H1 obrigatório; seção "Optional"), páginas `.md`, `skill.md`, `/.well-known/agent-skills`.
6. **Prompt copiável / playbook** (`setup.md`, `get-started.md`), deeplink do Cursor (`cursor://anysphere.cursor-deeplink/mcp/install?name=…&config=<base64>`, limite de 10.000 caracteres) [fato: https://cursor.com/docs/mcp/install-links.md].

**Armadilhas relatadas** [fato: issues `PostHog/wizard` #1325, #1326, #1329, #1330 em 10/10/2026; docs Supabase/Stripe/Codex]:
- Wizards quebram projetos (SDK duplicado, arquivo de auditoria esquecido, script de start que sai sem `.env`). Mitigação: rodar em branch limpa, `--dry-run`/`--diff`, recusar working tree sujo (Sentry).
- Prompt injection via dados (Supabase documenta); escopo mínimo (`read_only`, `project_ref`, `features`).
- MCP com ferramentas demais consome contexto (Codex limita skills a 2% da janela / 8.000 caracteres [fato: https://developers.openai.com/codex/skills]). Poucas ferramentas, nomes claros.
- Skills instaladas à mão não se atualizam (Stripe).

**Popups já instaláveis por agente?** OptiMonk e Popupsmart têm MCP de escrita (SaaS); Formbricks/Tally/Privy/Klaviyo têm MCP de gestão (não instalam widget). Nenhum grava configuração **no repo do cliente** [inferência a partir de `01`].

## 2. Desenho proposto para o popguia

### 2.1 Princípio

**O estado vive no repo do cliente.** Jornadas, experimentos e configuração são arquivos JSON versionados em `popguia/` dentro do projeto do cliente; o build publica `popguia/config.json` estático junto com o site. Resultado: funciona com **zero backend nosso** (só o coletor de eventos, que pode ser do cliente), o agente edita arquivos como edita qualquer código, e `git diff` é a trilha de auditoria.

### 2.2 O que o agente do cliente recebe (camadas)

| Camada | Artefato | Como chega ao agente |
|---|---|---|
| **A. Playbook** | `https://<docs>/setup.md` (prompt-playbook, estilo Resend/Vercel) + `llms.txt` + todas as páginas em `.md` | Usuário cola "Leia https://…/setup.md e instale no meu site"; ou botão "Copiar prompt" no site |
| **B. Skill** | `SKILL.md` (Agent Skills) com gatilhos "popup, jornada, quiz, lead, onboarding de visitante"; ensina a gerar jornadas a partir do conteúdo do site | `npx skills add inematds/popguia` · `claude plugin install …` (Claude Code) · Codex lê `AGENTS.md`/skills · Cursor rules |
| **C. CLI** | `npx popguia@latest init` · `add jornada <template>` · `validate` · `preview` · `doctor` · `report` · `optimize` | O agente roda; tudo com `--dry-run` e `--diff` |
| **D. MCP local** | `npx -y @popguia/mcp` (stdio; sem conta, sem chave) — mesmo código do CLI, exposto como ferramentas | `.mcp.json` no projeto / `claude mcp add popguia -- npx -y @popguia/mcp` / deeplink Cursor / `codex mcp add` |
| **E. MCP remoto (fase SaaS)** | `https://mcp.<dominio>/mcp` com OAuth, para métricas agregadas de vários sites | Só quando houver coletor hospedado |

### 2.3 Ferramentas do MCP (poucas, nomes em português, verbos claros)

| Ferramenta | Entrada | Saída | Observação |
|---|---|---|---|
| `detectar_site` | caminho do projeto | `{framework: next\|astro\|vite\|html\|wordpress\|desconhecido, pontoDeInjecao, buildCmd, publicDir, conteudo: [urls/sitemap/llms.txt encontrados]}` | lê `package.json`, `next.config.*`, `wp-config.php`, `index.html`, `sitemap.xml`, `llms.txt` |
| `instalar_snippet` | `{framework, modo: script\|npm, dryRun}` | diff dos arquivos tocados | Next.js: `<Script strategy="lazyOnload">` em `app/layout.tsx`; HTML: tag `defer` antes de `</body>`; WP: plugin ou `wp_footer`; Astro/Vite: componente |
| `listar_jornadas` | — | jornadas em `popguia/journeys/*.json` com status | |
| `criar_jornada` | JSON da jornada (ou `template` + parâmetros) | arquivo gravado + resultado do `validar_jornada` | templates: `diagnostico`, `quiz-educativo`, `learn-earn`, `microcompromisso`, `newsletter-2-etapas` |
| `validar_jornada` | id ou JSON | erros de schema, de acessibilidade (textos de botão, contraste), de conformidade (consentimento, frequência) | mesmo validador do CLI |
| `gerar_variantes` | `{jornadaId, elemento, n, objetivo, contexto}` | **brief + JSON Schema + exemplos** (não gera texto) | ver 2.4 |
| `salvar_variante` | `{jornadaId, elemento, variantes[]}` | grava + valida + cria/atualiza experimento | |
| `criar_experimento` | `{jornadaId, tipo: ab\|bandit, holdout, metrica, contexto[]}` | arquivo `popguia/experiments/<id>.json` | |
| `preview_jornada` | id, viewport | HTML estático renderizado (e PNG quando houver Playwright) | para o agente "ver" o popup |
| `ler_metricas` | `{jornadaId?, periodo}` | impressões, conclusão por etapa, conversão, dismissal, bounce pós-popup, SRM, posterior por braço | lê o coletor configurado |
| `otimizar` | `{experimentoId}` | novos pesos (Thompson) gravados em `popguia/weights.json` + resumo | roda a cadência do bandit |
| `doctor` | URL do site publicado | checklist: script carregou? config acessível? `<dialog>` suportado? CLS do widget = 0? consentimento integrado? | auto-verificação |

Regras das ferramentas: todas idempotentes; nada escreve fora de `popguia/` e do ponto de injeção; `instalar_snippet` recusa working tree sujo sem `--force` (padrão Sentry); nenhuma lê `.env` (padrão Clerk).

### 2.4 Geração por IA sem custo para nós (o truque central)

`gerar_variantes` **não chama modelo nenhum**. Ela devolve ao agente do cliente:

```json
{
  "brief": "Escreva 3 variantes do título da etapa 'welcome' da jornada 'orientacao-cursos' para visitantes vindos de UTM=youtube, tom direto, ≤ 60 caracteres, sem promessa de ganho financeiro, em pt-BR.",
  "contexto": {"site": "escola-exemplo.com", "pagina": "/cursos", "audiencia": "iniciante 30+", "exemplosDoSite": ["…"]},
  "schema": {"$ref": "popguia://schemas/variant.json"},
  "exemplos": [{"id": "v1", "texto": "…"}],
  "proximoPasso": "chame salvar_variante com o array gerado"
}
```

O **agente do próprio cliente** (Claude Code, Codex, Cursor) escreve as variantes — ele já é um LLM pago pelo cliente — e chama `salvar_variante`, que valida contra o schema e as regras (sem dark pattern, sem desconto acima do teto). O mesmo vale para `criar_jornada` a partir do conteúdo do site: a skill ensina a ler `sitemap.xml`/`llms.txt`/catálogo e montar perguntas → mapeamento → recomendações.

**Modo (b), opcional**: o cliente configura um provedor em `popguia.config.json` (`"ia": {"provedor": "openai|anthropic|ollama|openrouter", "modelo": "…", "chaveEnv": "POPGUIA_IA_KEY"}`); então `popguia gerar --variantes` roda sem agente (CI, cron, painel). A chave é do cliente, lida do ambiente dele; nunca uma chave nossa. Ollama local cobre quem não quer API.

### 2.5 Detecção de framework e injeção (CLI/MCP)

```
detectar:
  package.json deps.next            → next  (app/ ou pages/)
  deps.astro                        → astro
  deps.vite + index.html            → vite/html
  wp-config.php | wp-content/       → wordpress
  index.html na raiz/public/        → html
  senão                             → pergunta ao agente (que pergunta ao usuário)

injetar:
  next   → app/layout.tsx: <Script src="/popguia/popguia.js" strategy="lazyOnload" data-config="/popguia/config.json" />
           + copia widget para public/popguia/ (ou usa CDN opcional) + script "popguia build" no package.json (gera config.json a partir de popguia/)
  html   → <script defer src="/popguia/popguia.js" data-config="/popguia/config.json"></script> antes de </body>
  wp     → plugin popguia-wp (GPL) que enfileira o script em wp_footer e serve config via REST/arquivo
  astro/vite → componente/tag no layout base
```

O widget carrega o `config.json` depois do `load` (idle), avalia regras localmente, decide variante pelos `weights.json` (ou alocação A/B determinística por hash de `visitor_id`) e envia eventos para o sink configurado (coletor próprio, PostHog, Umami, GA4, ou `console` em dev).

### 2.6 Fluxo exemplo 1 — HTML estático, agente Claude Code

```
Usuário: "Instala o popguia no meu site e cria um quiz de 3 perguntas que recomende o produto certo. Leia https://<docs>/setup.md"
Agente:  lê setup.md → npx skills add inematds/popguia → claude mcp add popguia -- npx -y @popguia/mcp
         detectar_site → {framework: html, publicDir: ".", conteudo: ["sitemap.xml", "produtos/*.html"]}
         instalar_snippet --dry-run → mostra diff → usuário aprova → aplica
         lê os produtos do site → criar_jornada(template: diagnostico, perguntas: [...], recomendacoes: {...})
         validar_jornada → OK (2 avisos: contraste do botão secundário; falta texto de consentimento) → corrige
         preview_jornada → HTML/PNG → usuário vê
         criar_experimento(tipo: ab, holdout: 0.1, metrica: clique_recomendacao)
         git commit (o usuário faz o push)
         doctor https://site-do-cliente.com → tudo verde
```

### 2.7 Fluxo exemplo 2 — site de cursos (Next.js no Vercel), agente Codex

```
Nei:    "codex: instala o popguia no portal com um agente de orientação que pergunta o que a pessoa quer aprender e manda para o curso certo"
Codex:  lê AGENTS.md do portal + SKILL.md do popguia
        detectar_site → next (app router), conteudo: src/data/courses.data.json (320 cursos com tags), src/app/llms.txt
        instalar_snippet → app/layout.tsx + public/popguia/ + "postbuild": "popguia build"
        criar_jornada(id: orientacao-cursos) com 3 perguntas → tags → top-3 cursos por pontuação (ver PLANO.md §2, demo escola)
        gerar_variantes(elemento: welcome.titulo, n: 3, contexto: utm_source) → Codex escreve 3 títulos → salvar_variante
        criar_experimento(tipo: ab, holdout: 0.1, metrica: clique_curso)
        validate + preview → commit → push (deploy automático) → doctor https://escola-exemplo.com
```

### 2.8 Ordem de implementação sugerida

1. **CLI** (`init`, `validate`, `build`, `doctor`) — é a base e é testável sem agente.
2. **MCP local** como casca fina sobre o CLI (mesmo pacote, `popguia mcp`), para Claude Code / Codex / Cursor.
3. **SKILL.md + setup.md + llms.txt** no repo e no site de docs (GitHub Pages).
4. **Templates de jornada** e `gerar_variantes` (brief + schema).
5. **Plugin WordPress** fino.
6. **MCP remoto + registry** (`mcpName` no `package.json`, `mcp-publisher`) quando houver coletor hospedado [fato: https://modelcontextprotocol.io/registry/quickstart.md].
