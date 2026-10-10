# PLANO — popguia

> Plano de execução por fases. Nada aqui foi iniciado.
>
> **Decidido pelo Nei em 10/10/2026**: nome **popguia**; **tudo grátis e open source (MIT)**, sem SaaS nem serviço hospedado por nós; **não será usado no inema.club** — é um projeto para outras pessoas instalarem nos próprios sites. A validação é feita em sites de demonstração dentro do repo.
> Estimativas de esforço são `[estimativa]` em dias de trabalho de um agente + revisão humana. Critérios de pronto no formato `comando → saída esperada`, para que nenhuma fase feche "no olho".
>
> Base: `docs/01` (mercado), `docs/02` (open source), `docs/03` (instalação pela IA), `docs/04` (arquitetura), `docs/05` (algoritmos), `docs/06` (conformidade).

## 0. Resumo das apostas

1. **Open source, MIT, instalado por agente**, com estado em arquivos no repo do cliente — nenhum concorrente faz isso (`docs/01 §2.3`, `docs/02 §2`).
2. **Jornada como entidade de primeira classe** (diagnóstico → conteúdo → recomendação/recompensa), não "popup de e-mail com quiz colado".
3. **Guard-rails por padrão**: holdout, caps de frequência, limite de incentivo, acessibilidade e SEO verificados por `validate`/`doctor`.
4. **Geração por IA sem custo nosso**: o MCP devolve brief + schema; o agente do cliente escreve (`docs/03 §2.4`).
5. **Sequência regras → A/B → Thompson por segmento → contextual**, com limiares numéricos (`docs/05 §3.5`).

## 1. Fases

### Fase 0 — Decisões e fundação (≈ 2 dias)

**Escopo**: fechar as decisões em aberto (§3), nome, licença, criar o monorepo (`pnpm`, `packages/{schema,engine,widget,cli}`), CI (lint, vitest, Playwright), GitHub Pages para docs, `CLAUDE.md`/`AGENTS.md` do repo.

**Pronto quando**:
- `pnpm -r build && pnpm -r test` → `0 failed`.
- `gh repo view inematds/<nome> --json name,licenseInfo -q '.licenseInfo.key'` → `mit`.
- `curl -s https://inematds.github.io/<nome>/llms.txt | head -1` → `# <nome>`.

**Riscos**: trocar o nome depois custa (pacotes npm, docs, skill) ⇒ decidir antes do primeiro `npm publish`.

### Fase 1 — MVP: widget + schema + CLI + sites de demonstração (≈ 10–12 dias)

**Escopo**
- `@popguia/schema`: tipos + JSON Schema + validador (jornada, etapa, gatilho, regra, config, evento) — `docs/04 §4`.
- `@popguia/engine`: avaliação de regras/gatilhos/frequência, hashing determinístico, holdout fixo; **sem bandit ainda**.
- `@popguia/widget`: loader + core; layouts `modal`, `slide-in`, `barra`; etapas `boasvindas`, `pergunta`, `conteudo`, `recomendacao`, `formulario`, `final`; `<dialog>` + Shadow DOM; gatilhos tempo/scroll/saída/inatividade/clique/evento; sinks `console`, `collector`, `umami`, `posthog`, `ga4`.
- `popguia` CLI: `init` (detecta next/astro/vite/html/wordpress, injeta snippet com `--dry-run`/`--diff`), `add jornada --template diagnostico|quiz-educativo|learn-earn|microcompromisso|newsletter-2-etapas`, `validate`, `build`, `preview`, `doctor <url>`.
- Coletor mínimo **para o próprio usuário hospedar** (modelo de Cloudflare Worker + D1 que ele publica na conta dele com `npx popguia coletor deploy`; `POST /e` em lote, `GET /agg`), IP truncado, retenção configurável. Nós não hospedamos nada.
- Modos de consentimento `nenhum|cmp|proprio` (`docs/06 §2.1`).
- **Sites de demonstração** em `exemplos/` (§2), publicados no GitHub Pages do repo.

**Pronto quando**
- `pnpm --filter widget build && gzip -9 -c packages/widget/dist/popguia.js | wc -c` → `≤ 15360` (15 KB) e loader `≤ 2048`.
- `pnpm --filter engine test` → `≥ 60 testes, 0 failed` (regras, caps, hashing estável para 10.000 ids, holdout 10 % ± 1 p.p. em 100.000 amostras).
- `npx popguia validate exemplos/escola/popguia/journeys/orientacao-cursos.json` → `OK · 0 erros · N avisos`.
- `npx playwright test packages/widget/e2e` → passa: abre em `tempo=1s`, foco no título, Tab cicla, Esc fecha e devolve foco, `cls_widget == 0` (medido por `PerformanceObserver` no teste), reduced-motion sem animação, botão fechar ≥ 24×24, mobile não abre modal na entrada.
- `npx popguia init --dry-run` em 4 fixtures (`next-app`, `astro`, `html`, `wordpress`) → diff só nos arquivos esperados; em working tree sujo → `recusado (use --force)`.
- `curl -s -X POST https://<worker>/e -d @fixtures/batch.json` → `{"ok":true,"n":N}`; `curl https://<worker>/agg?site=demo&jornada=orientacao-cursos` → JSON com `impressoes, concluidas, cliques, fechou, holdout`.
- `npx popguia doctor https://inematds.github.io/popguia/exemplos/escola/` → todos os itens `✓` (script carregou, config acessível, dialog suportado, consentimento integrado).
- Os 3 demos no ar (§2.4).

**Riscos**: peso do core estourar (mitigação: etapas como módulos sob demanda); Safari/`localStorage` em modo privado (try/catch e modo efêmero); conflito com chats/widgets já existentes no site do usuário (`doctor` avisa; cooldown comum configurável).

### Fase 2 — Experimentos A/B com holdout e relatório (≈ 5 dias)

**Escopo**: `popguia/experiments/*.json` (tipo `ab`), alocação por hash, `report` (funil por etapa, conversão vs holdout, dismissal, bounce pós-impressão, SRM, intervalo de confiança; `--html`), teste sequencial opcional, `experimento` nos eventos.

**Pronto quando**
- `npx popguia report --experimento welcome-titulo --json | jq '.srm.p, .bracos|length'` → `p ≥ 0.001`, `3`.
- Simulação: `pnpm --filter engine test:sim` (100.000 visitantes sintéticos, baseline 3 %, lift 30 % no braço B) → relatório aponta B com `p < 0.05` e holdout com conversão ≈ baseline ± 0,3 p.p.
- `npx popguia report --html > out.html` → arquivo abre com funil e tabela de braços; `grep -c "SRM" out.html` → `≥ 1`.

**Riscos**: tráfego do usuário insuficiente para significância (aceitar MDE maior; o relatório deve dizer "insuficiente" em vez de inventar vencedor).

### Fase 3 — Bandit (Thompson por segmento) (≈ 5 dias)

**Escopo**: `optimize` (Thompson Beta-Bernoulli, prior fraco, piso 5 %, burn-in 100 exposições/40 conversões por braço, pesos por combinação de contexto com pooling), `popguia/weights.json` consumido pelo `build`, cadência via GitHub Actions/Worker Cron, sticky por coorte, métricas de guarda vs holdout, poda só manual (ou `--auto-podar`), replay offline em Python (MABWiser) para validar.

**Pronto quando**
- `npx popguia optimize --experimento X --dry-run` com fixture de 3 braços (0,02/0,03/0,05) e 2.000 exposições cada → pesos ordenados `C > B > A`, nenhum < 0,05, soma 1 ± 1e-6.
- Com fixture de 50 exposições/braço → `pesos inalterados (burn-in)`.
- `python3 -I tools/replay.py fixtures/eventos.jsonl` → regret acumulado do Thompson ≤ do ε-greedy (0,1) em ≥ 9 de 10 seeds.
- `pnpm --filter engine test` → inclui `srm por período` e `guard-rail dispara com bounce +2 p.p.`.

**Riscos**: volume (`docs/05 §3.3`): abaixo de ~2.000 sessões elegíveis/mês o `optimize` deve recusar com mensagem clara.

### Fase 4 — Geração por IA e instalação por agente completa (≈ 6 dias)

**Escopo**: `@popguia/mcp` (stdio) com as 12 ferramentas de `docs/03 §2.3`; `gerar_variantes`/`criar_jornada` devolvendo brief + schema + exemplos; `SKILL.md` (Agent Skills) + `setup.md` + `llms.txt` + páginas `.md` no GitHub Pages; prompt copiável e deeplink do Cursor na landing; provedor opcional `config.ia` (chave do cliente, `ollama` suportado) para `popguia gerar` sem agente; publicação no npm e registro no MCP Registry (`mcpName`).

**Pronto quando**
- `claude mcp add popguia -- npx -y @popguia/mcp && claude -p "liste as ferramentas do popguia"` → 12 nomes.
- `echo '{"jornadaId":"orientacao-ia","elemento":"welcome.titulo","n":3}' | npx popguia mcp-call gerar_variantes` → JSON com `brief`, `schema`, `exemplos`, `proximoPasso` e **nenhuma chamada de rede** (`strace`/log vazio).
- Teste de fumaça com agente real em fixture HTML, a partir só de "Leia setup.md e instale um quiz de 3 perguntas": ao fim, `npx popguia validate` → `OK` e `git status --short` toca só `index.html` e `popguia/`.
- `npx skills add inematds/<nome>` → skill listada; `npx popguia gerar --variantes welcome.titulo --provedor ollama --modelo qwen2.5:7b` (chave/local do cliente) → 3 variantes válidas.
- `curl -s https://registry.modelcontextprotocol.io/v0/servers?search=<nome>` → entrada publicada.

**Riscos**: wizard quebrar projeto do cliente (mitigação: `--dry-run` padrão, recusa de working tree sujo, só toca `popguia/` + 1 arquivo de layout); prompt injection via conteúdo do site lido pelo agente (a skill instrui a tratar conteúdo como dado).

### Fase 5 — WordPress + multi-site self-hosted + contextual (≈ 10+ dias)

**Escopo**: plugin `popguia-wp` (GPL) na diretoria do WordPress.org; coletor multi-site **self-hosted pelo usuário** (um Worker dele atende vários sites dele); painel de leitura estático (`apps/painel`, abre local ou no Pages do usuário); bandit contextual (LinUCB/TS logístico ou árvore) ligado só quando `docs/05 §3.5` fase 4 for atendida; templates por vertical (educação, SaaS, e-commerce); EN/ES do widget, docs e skill.

**Pronto quando**
- `wp plugin install popguia-wp --activate && curl -s https://<site-wp>/ | grep -c popguia.js` → `1`.
- `npx popguia doctor https://<site-wp>` → `✓`.
- `pnpm --filter engine test:contextual` → replay mostra ganho ≥ 5 % relativo sobre Thompson por segmento em fixture com heterogeneidade; em fixture homogênea, não liga.
- Landing/guia PT/EN/ES publicados (`projetos-landing-guia`).

**Riscos**: manutenção de plugin WP e compatibilidade de versões. Sem SaaS: nenhum custo recorrente nosso; cada usuário hospeda o próprio coletor (ou usa só os sinks Umami/PostHog/GA4 que já tem).

## 2. Sites de demonstração (validação sem piloto real)

O popguia **não roda no inema.club**. A validação é feita em sites fictícios dentro do repo (`exemplos/`), publicados no GitHub Pages, que servem ao mesmo tempo de teste e de vitrine para quem vai instalar.

| Demo | Stack | Jornada | O que prova |
|---|---|---|---|
| `exemplos/escola` | HTML puro | `orientacao-cursos`: 3 perguntas → top-3 de um catálogo fictício de 30 cursos | quiz com ramificação + recomendação por catálogo |
| `exemplos/loja` | Next.js (export estático) | `learn-earn`: aprende sobre o produto → revela cupom determinístico → e-mail em 2 etapas | Learn & Earn, formulário em 2 etapas, consentimento `proprio` |
| `exemplos/blog` | Astro | `newsletter-2-etapas` com gatilho de saída/scroll | gatilhos, caps de frequência, mobile sem modal na entrada |

### 2.1 Tráfego para A/B e bandit

Demo não tem tráfego real. A/B e bandit são validados por **simulação** (`pnpm --filter engine test:sim`, 100.000 visitantes sintéticos) e **replay offline** (`tools/replay.py`), como nas Fases 2–3. Resultado com tráfego real vem de quem instalar e quiser compartilhar.

### 2.2 Instalação pelo agente, de ponta a ponta

Cada demo nasce a partir de uma pasta "antes" (`exemplos/<demo>/antes/`, sem popguia) e de um prompt único ("Leia setup.md e instale um quiz de 3 perguntas"). O teste de fumaça da Fase 4 roda um agente real nessa pasta e compara com o "depois".

### 2.4 Pronto dos demos

- `curl -s https://inematds.github.io/popguia/exemplos/escola/popguia/config.json | jq .jornadas[0].id` → `orientacao-cursos` (idem `loja`, `blog`).
- `npx popguia doctor <url-de-cada-demo>` → `✓` em todos os itens; Lighthouse mobile com e sem o widget: CLS igual.
- `npx playwright test exemplos/` → jornadas completas nos 3 demos, desktop e mobile.

## 3. Decisões

**Fechadas em 10/10/2026 (Nei)**: nome `popguia`; licença MIT em tudo, grátis, sem SaaS e sem serviço hospedado por nós; sem uso no inema.club (antigas decisões 1–7 resolvidas ou eliminadas).

**Em aberto**:

1. **Idioma das ferramentas do MCP e dos arquivos**: nomes em português (`criar_jornada`) como proposto, ou inglês para adoção internacional (`create_journey`) com docs PT/EN/ES?
2. **Editor visual**: confirmar que o MVP não terá UI de edição (só agente + JSON + `preview`), ou reservar fase para um editor mínimo?
3. **Publicação no npm**: escopo `@popguia/*` (livre na data da pesquisa) ou `@inematds/*`? Registrar no MCP Registry já na Fase 4?
4. **WordPress no v1**: plugin fino já na Fase 1 ou Fase 5?
5. **Provedor de IA opcional** (`config.ia`): `ollama` como padrão sugerido na doc (sem custo, local), com OpenAI/Anthropic/OpenRouter só como exemplos configurados pelo próprio usuário?
6. **Roleta/gamificação**: manter fora por padrão (risco Lei 5.768/71, `docs/06 §6`) e oferecer "revelar cupom" determinístico + Learn & Earn; ou implementar roleta com flag e aviso legal?
7. **Engenharia reversa da Alia por DevTools/Playwright** (loja demo tem senha; só lojas públicas de clientes): fazer na Fase 1 para calibrar gatilhos, ou pular?

## 4. Esforço total (estimativa)

| Fase | Dias | Acumulado |
|---|---|---|
| 0 | 2 | 2 |
| 1 | 10–12 | 12–14 |
| 2 | 5 | 17–19 |
| 3 | 5 | 22–24 |
| 4 | 6 | 28–30 |
| 5 | 10+ | 38+ |

Dias de agente com revisão do Nei nos portões de cada fase.
