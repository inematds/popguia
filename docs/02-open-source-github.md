# 02 — Open source / GitHub: o que existe e no que se apoiar

> Metadados colhidos em 10/10/2026 com `gh api repos/<owner>/<repo>` e `gh api repos/<owner>/<repo>/license` (estrelas, SPDX, data do último push, linguagem) — são `[fato]` salvo indicação. Tamanhos de bundle: API pública do bundlephobia (min/gzip) quando respondeu; o resto é `[estimativa]`. "O que aproveitar" é avaliação nossa `[inferência]`.
>
> Critério decisivo: **a licença do que roda no site do cliente**. O widget embutido precisa ser MIT/Apache/BSD — qualquer coisa AGPL/FSL/"non-commercial" só é tolerável do lado do servidor, e isso é decisão do Nei (ver `PLANO.md`).

## 1. Tabela de candidatos

### 1.1 Widgets, popups, diálogos, gatilhos (roda no site do cliente — licença permissiva obrigatória)

| Repo | Estrelas | Licença | Último push | Ling. | O que aproveitar |
|---|---|---|---|---|---|
| [preactjs/preact](https://github.com/preactjs/preact) | 38.921 | MIT | 2026-10-09 | JS | Runtime de UI de **5,0 KB gz** (bundlephobia 11.0.1) para o widget, se quisermos JSX; alternativa é vanilla TS |
| [lit/lit](https://github.com/lit/lit) | 21.839 | BSD-3-Clause | 2026-09-24 | TS | Web components com Shadow DOM, 6,0 KB gz (3.3.3); mais pesado que o necessário [inferência] |
| [KittyGiraudel/a11y-dialog](https://github.com/KittyGiraudel/a11y-dialog) | 2.489 | MIT | 2026-10-05 | TS | Diálogo acessível de **1,8 KB gz** (8.1.5): foco, Esc, `aria-modal`; referência direta para o modal |
| [focus-trap/focus-trap](https://github.com/focus-trap/focus-trap) | 1.563 | MIT | 2026-10-07 | JS | 6,5 KB gz (8.2.3); só se `<dialog>` nativo não bastar [inferência] |
| [carlsednaoui/ouibounce](https://github.com/carlsednaoui/ouibounce) | 2.295 | MIT | **2017-05-14** | JS | Exit intent desktop: sensibilidade 20 px do topo, não dispara no 1º segundo, 1× por visitante via cookie. Abandonado; copiar a heurística, não o código |
| [beeker1121/exit-intent-popup](https://github.com/beeker1121/exit-intent-popup) | 320 | MIT | 2026-04-02 | JS | Popup de exit intent simples; referência de API |
| [daltonmenezes/use-exit-intent](https://github.com/daltonmenezes/use-exit-intent) | 101 | MIT | 2026-07-24 | TS | Hook React com estratégias desktop/mobile (inatividade, scroll); referência para a versão React |
| [AMKohn/bounceback](https://github.com/AMKohn/bounceback) | 44 | MIT | 2025-04-28 | JS | Exit intent sem dependências, leve |
| [nilbuild/driver.js](https://github.com/nilbuild/driver.js) | 26.897 | MIT | 2026-10-03 | TS | Tour/spotlight de 7,6 KB gz (1.9.0); modelo de "overlay sem CLS" e de API declarativa |
| [usablica/intro.js](https://github.com/usablica/intro.js) | 23.454 | "other" (AGPL + comercial) | 2026-10-09 | TS | **Evitar** no widget (AGPL) |
| [shipshapecode/shepherd](https://github.com/shipshapecode/shepherd) | 13.818 | AGPL-3.0 + comercial | 2026-10-07 | JS | **Evitar** no widget |
| [NiazMorshed2007/popwola](https://github.com/NiazMorshed2007/popwola) | 82 | MIT | 2024-11-12 | TS | Construtor no-code de popups; pequeno, parado há 1 ano; olhar o editor de templates [inferência] |
| [PopupMaker/Popup-Maker](https://github.com/PopupMaker/Popup-Maker) | 110 | sem SPDX na API (plugin WP ⇒ GPL-2.0+ [inferência]) | 2026-10-05 | PHP | Referência de **integração WordPress**: triggers, cookies de frequência, condições por página; um plugin WP nosso terá de ser GPL-compatível de qualquer forma |

### 1.2 Formulários, surveys, jornadas (dados e fluxo)

| Repo | Estrelas | Licença | Último push | Ling. | O que aproveitar |
|---|---|---|---|---|---|
| [formbricks/formbricks](https://github.com/formbricks/formbricks) | 13.082 | Core AGPL-3.0 + `apps/web/modules/ee` proprietária; **`packages/js` = MIT** (`@formbricks/js-core` 1.0.0, MIT) | 2026-10-10 | TS | Arquitetura do **SDK in-app** (gatilhos por ação, `no-code actions`, atributos de usuário, fila de eventos, surveys renderizadas em overlay). O SDK MIT pode ser lido e copiado; o backend AGPL não entra no cliente. MCP oficial só gerencia surveys (OAuth 2.1) |
| [baptisteArno/typebot.io](https://github.com/baptisteArno/typebot.io) | 10.524 | **FSL-1.1-Apache-2.0** | 2026-10-07 | TS | Melhor modelo de dados de **fluxo com ramificação** (grupos, blocos, edges, variáveis, condições) e embed popup/bubble. A FSL proíbe uso em produto concorrente até virar Apache-2.0 (2 anos por versão) ⇒ só como **referência de desenho** [inferência] |
| [heyform/heyform](https://github.com/heyform/heyform) | 9.008 | AGPL-3.0 | 2026-10-10 | TS | Form builder; referência de UI de perguntas |
| [OpnForm/OpnForm](https://github.com/OpnForm/OpnForm) | 3.798 | AGPL-3.0 + Enterprise | 2026-10-07 | PHP | Idem; não aproveitar |
| [luispdoesai/openFunnel](https://github.com/luispdoesai/openFunnel) | 18 | AGPL-3.0 | 2026-08-10 | JS | Quiz funnel self-hosted para tráfego pago; conceito próximo, imaturo, AGPL |
| [BeefreeSDK/beefree-sdk-skills](https://github.com/BeefreeSDK/beefree-sdk-skills) | 1 | Apache-2.0 | 2026-10-02 | JS | Exemplo de **Agent Skills para integrar um builder de popups** por agente — formato que queremos publicar |

### 1.3 Experimentos, flags, bandits

| Repo | Estrelas | Licença | Último push | Ling. | O que aproveitar |
|---|---|---|---|---|---|
| [growthbook/growthbook](https://github.com/growthbook/growthbook) | 8.486 | MIT, exceto `packages/*/enterprise` (GrowthBook Enterprise License); **`@growthbook/growthbook` SDK 1.8.1 = MIT** | 2026-10-10 | TS | Referência número 1 para **bandit de produto**: Thompson Bayesiano, piso de 1% por variação, mínimo 100 usuários/variação antes de atualizar pesos, 40 conversões/variação recomendadas, cadência ≥ 15 min, TTL de cache ≤ 1/5 da cadência, sticky bucketing, pesagem por coorte [fato: https://docs.growthbook.io/bandits/config]. Bandit contextual = beta enterprise [fato: https://docs.growthbook.io/bandits/contextual]. SDK JS (hash determinístico, namespaces, `attributes`) é copiável; o servidor é pesado demais para "sem backend" [inferência] |
| [PostHog/posthog](https://github.com/PostHog/posthog) | 40.217 | MIT (exceto `ee/`) | 2026-10-10 | Py | Opção de **sink de eventos** do cliente (self-host ou cloud); tem surveys, flags, experiments e MCP + wizard. `posthog-js` 1.438.7 = **101,8 KB gz** (Apache-2.0 AND MIT) ⇒ nunca embutir; só integrar se o cliente já usa |
| [Unleash/unleash](https://github.com/Unleash/unleash) | 13.867 | AGPL-3.0 | 2026-10-09 | TS | Flags; não entra |
| [Flagsmith/flagsmith](https://github.com/Flagsmith/flagsmith) | 6.591 | BSD-3-Clause | 2026-10-09 | Py | Flags/remote config; referência de "config publicada como JSON estático por ambiente" |
| [VowpalWabbit/vowpal_wabbit](https://github.com/VowpalWabbit/vowpal_wabbit) | 8.728 | BSD-3-Clause (texto do LICENSE; API sem SPDX) | 2026-09-28 | C++ | `--cb_explore_adf` para bandit contextual offline; exagero para poucos braços [inferência] |
| [fidelity/mabwiser](https://github.com/fidelity/mabwiser) | 295 | Apache-2.0 | 2024-09-05 | Py | LinUCB, LinTS, TS, UCB1, políticas de vizinhança; bom para **avaliação offline (replay)** e para validar nossa implementação TS |
| [david-cortes/contextualbandits](https://github.com/david-cortes/contextualbandits) | 840 | BSD-2-Clause | 2026-06-28 | Py | Off-policy (doubly robust, offset tree); idem |
| [thoughtworks/simplebandit](https://github.com/thoughtworks/simplebandit) | 19 | Apache-2.0 (texto do LICENSE) | 2023-12-11 | TS | Bandit contextual leve em TS; parado; pode servir de ponto de partida para o `engine` contextual na fase 5 |
| [instadeepai/catx](https://github.com/instadeepai/catx) | 72 | MIT | 2026-07-28 | Py | Pesquisa; não aproveitar |
| npm `@mab-kit/core` 0.1.0 (MIT), `@cognitive-engine/bandit` 0.2.1 (Apache-2.0), `ucb` 3.2.0 (2021) | — | — | — | TS/JS | Imaturos; Thompson Beta-Bernoulli são ~30 linhas — **implementar nós mesmos** [inferência] |

### 1.4 Eventos / analytics (sink opcional)

| Repo | Estrelas | Licença | Último push | Ling. | O que aproveitar |
|---|---|---|---|---|---|
| [umami-software/umami](https://github.com/umami-software/umami) | 39.296 | MIT | 2026-10-10 | TS | Sink leve, self-host, sem cookie; adaptador `umami.track()` |
| [plausible/analytics](https://github.com/plausible/analytics) | 29.359 | AGPL-3.0 | 2026-10-09 | Elixir | Idem (AGPL só no servidor do cliente; adaptador é trivial) |
| [openreplay/openreplay](https://github.com/openreplay/openreplay) | 12.968 | ELv2 + `ee/` | 2026-10-09 | TS | Replay de sessão; fora do escopo |
| [rrweb-io/rrweb](https://github.com/rrweb-io/rrweb) | 20.253 | MIT | 2026-10-08 | TS | Só se um dia quisermos gravar a jornada |

### 1.5 Instalação por agente (referências de desenho)

| Repo | Estrelas | Licença | Último push | Ling. | O que aproveitar |
|---|---|---|---|---|---|
| [shadcn-ui/ui](https://github.com/shadcn-ui/ui) | 125.273 | MIT | 2026-10-10 | TS | Modelo **registry + CLI `init/add` + MCP local via `npx shadcn@latest mcp`** ("o código vai para o repo do cliente") — é o padrão mais próximo do que queremos |
| [getsentry/sentry-wizard](https://github.com/getsentry/sentry-wizard) | 295 | **FSL-1.1-MIT** | 2026-10-08 | TS | Detecção de framework, `--ignore-git-changes`, página de teste; ler como referência, não copiar código (FSL) |
| [PostHog/wizard](https://github.com/PostHog/wizard) | 197 | MIT | 2026-10-09 | TS | Wizard que envolve o Claude Agent SDK + "skills packages"; `wizard audit …` como auto-verificação [fato: https://posthog.com/docs/ai-engineering/ai-wizard] |
| [GrapesJS/grapesjs](https://github.com/GrapesJS/grapesjs) | 26.294 | BSD-3-Clause (texto do LICENSE; API sem SPDX) | 2026-10-06 | TS | Só se um dia houver editor visual; não no MVP |

## 2. O que a busca NÃO encontrou

- Nenhum repo open source que junte popup + jornada/quiz + A/B/bandit + instalação por agente. As buscas `popup exit intent`, `popup builder`, `quiz funnel builder`, `mcp server popup`, `in-app survey widget` retornaram projetos pequenos (< 100 estrelas) ou plugins WP/Elementor [fato: `gh search repos`, 10/10/2026]. A combinação é o espaço vazio.
- Nenhum servidor MCP de popups open source. Os existentes (Formbricks, Tally, OptiMonk, Popupsmart, Privy) são SaaS.

## 3. Recomendação: construir do zero, apoiado em referências

**Veredito:** construir o núcleo do zero em TypeScript (monorepo pequeno), **MIT**, apoiando-se em:

1. **Widget**: vanilla TS + `<dialog>` nativo (97% de suporte [fato: https://caniuse.com/dialog]) + Shadow DOM; sem framework no v1. Se o JSX fizer falta, Preact (5 KB gz, MIT). `a11y-dialog` como referência de foco/Esc. Heurísticas de exit intent de ouibounce/use-exit-intent/OptiMonk (ver `05`).
2. **Modelo de jornada**: desenho inspirado no Typebot (grupos/blocos/edges/variáveis) e nos Steps/Elements da Alia, mas escrito por nós em JSON Schema — Typebot é FSL e não pode ser base.
3. **Motor de decisão**: Thompson Beta-Bernoulli próprio, com as regras operacionais do GrowthBook (100 usuários/variação, 40 conversões, piso de exploração, sticky bucketing). Validação offline em Python com MABWiser/contextualbandits (replay). GrowthBook SDK (MIT) como referência de hashing determinístico para alocação.
4. **Eventos**: coletor próprio minúsculo (Cloudflare Worker + D1, ou Supabase) **e** adaptadores para PostHog/Umami/GA4 quando o cliente já tem — o widget nunca embute o `posthog-js`.
5. **Instalação por agente**: padrão shadcn (CLI `init` que grava arquivos no repo do cliente + MCP local `npx -y <pacote> mcp`) + SKILL.md no formato Agent Skills + `llms.txt` (ver `03`).
6. **WordPress**: plugin fino GPL que só injeta o snippet e expõe a config; Popup Maker como referência de condições.

**Por que não "apoiar-se em X"**:
- GrowthBook inteiro: servidor Node + Mongo, bandit só no plano Pro/enterprise e contextual só enterprise — contraria "sem backend nosso" e "instala pelo agente em 5 minutos" [inferência].
- Formbricks: AGPL no core; o SDK é MIT mas é survey, não jornada com ramificação/recompensa; o backend exigiria self-host pelo cliente [inferência].
- Typebot: FSL impede produto concorrente [fato: texto da licença].
- PostHog: o SDK tem 100 KB gz; como sink opcional é ótimo, como base não.

**Licença sugerida para o projeto**: MIT em tudo que vai para o cliente (widget, schema, engine, CLI, MCP, skill); o coletor hospedado (se virar SaaS) pode ficar MIT ou AGPL — decisão em aberto no `PLANO.md`.
