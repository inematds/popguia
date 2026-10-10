# PLANO — popsmart (nome provisório)

> Plano de execução por fases. Nada aqui foi iniciado; é proposta para decisão do Nei. Estimativas de esforço são `[estimativa]` em dias de trabalho de um agente + revisão humana. Critérios de pronto no formato `comando → saída esperada`, para que nenhuma fase feche "no olho".
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

### Fase 1 — MVP: widget + schema + CLI + piloto inema.club (≈ 10–12 dias)

**Escopo**
- `@popsmart/schema`: tipos + JSON Schema + validador (jornada, etapa, gatilho, regra, config, evento) — `docs/04 §4`.
- `@popsmart/engine`: avaliação de regras/gatilhos/frequência, hashing determinístico, holdout fixo; **sem bandit ainda**.
- `@popsmart/widget`: loader + core; layouts `modal`, `slide-in`, `barra`; etapas `boasvindas`, `pergunta`, `conteudo`, `recomendacao`, `formulario`, `final`; `<dialog>` + Shadow DOM; gatilhos tempo/scroll/saída/inatividade/clique/evento; sinks `console`, `collector`, `umami`, `posthog`, `ga4`.
- `popsmart` CLI: `init` (detecta next/astro/vite/html/wordpress, injeta snippet com `--dry-run`/`--diff`), `add jornada --template diagnostico|quiz-educativo|learn-earn|microcompromisso|newsletter-2-etapas`, `validate`, `build`, `preview`, `doctor <url>`.
- Coletor mínimo: Cloudflare Worker + D1 (`POST /e` em lote, `GET /agg`), IP truncado, retenção configurável.
- Modos de consentimento `nenhum|cmp|proprio` (`docs/06 §2.1`).
- **Piloto inema.club** (§2).

**Pronto quando**
- `pnpm --filter widget build && gzip -9 -c packages/widget/dist/popsmart.js | wc -c` → `≤ 15360` (15 KB) e loader `≤ 2048`.
- `pnpm --filter engine test` → `≥ 60 testes, 0 failed` (regras, caps, hashing estável para 10.000 ids, holdout 10 % ± 1 p.p. em 100.000 amostras).
- `npx popsmart validate popsmart/journeys/orientacao-ia.json` → `OK · 0 erros · N avisos`.
- `npx playwright test packages/widget/e2e` → passa: abre em `tempo=1s`, foco no título, Tab cicla, Esc fecha e devolve foco, `cls_widget == 0` (medido por `PerformanceObserver` no teste), reduced-motion sem animação, botão fechar ≥ 24×24, mobile não abre modal na entrada.
- `npx popsmart init --dry-run` em 4 fixtures (`next-app`, `astro`, `html`, `wordpress`) → diff só nos arquivos esperados; em working tree sujo → `recusado (use --force)`.
- `curl -s -X POST https://<worker>/e -d @fixtures/batch.json` → `{"ok":true,"n":N}`; `curl https://<worker>/agg?site=inema&jornada=orientacao-ia` → JSON com `impressoes, concluidas, cliques, fechou, holdout`.
- `npx popsmart doctor https://inema.club` → todos os itens `✓` (script carregou, config acessível, dialog suportado, consentimento integrado).
- Piloto no ar 14 dias com relatório (§2.5).

**Riscos**: peso do core estourar (mitigação: etapas como módulos sob demanda); Safari/`localStorage` em modo privado (try/catch e modo efêmero); `AgenteChat` do portal competir com o popup (decisão §3).

### Fase 2 — Experimentos A/B com holdout e relatório (≈ 5 dias)

**Escopo**: `popsmart/experiments/*.json` (tipo `ab`), alocação por hash, `report` (funil por etapa, conversão vs holdout, dismissal, bounce pós-impressão, SRM, intervalo de confiança; `--html`), teste sequencial opcional, `experimento` nos eventos.

**Pronto quando**
- `npx popsmart report --experimento welcome-titulo --json | jq '.srm.p, .bracos|length'` → `p ≥ 0.001`, `3`.
- Simulação: `pnpm --filter engine test:sim` (100.000 visitantes sintéticos, baseline 3 %, lift 30 % no braço B) → relatório aponta B com `p < 0.05` e holdout com conversão ≈ baseline ± 0,3 p.p.
- `npx popsmart report --html > out.html` → arquivo abre com funil e tabela de braços; `grep -c "SRM" out.html` → `≥ 1`.

**Riscos**: tráfego do piloto insuficiente para significância (aceitar MDE maior; o relatório deve dizer "insuficiente" em vez de inventar vencedor).

### Fase 3 — Bandit (Thompson por segmento) (≈ 5 dias)

**Escopo**: `optimize` (Thompson Beta-Bernoulli, prior fraco, piso 5 %, burn-in 100 exposições/40 conversões por braço, pesos por combinação de contexto com pooling), `popsmart/weights.json` consumido pelo `build`, cadência via GitHub Actions/Worker Cron, sticky por coorte, métricas de guarda vs holdout, poda só manual (ou `--auto-podar`), replay offline em Python (MABWiser) para validar.

**Pronto quando**
- `npx popsmart optimize --experimento X --dry-run` com fixture de 3 braços (0,02/0,03/0,05) e 2.000 exposições cada → pesos ordenados `C > B > A`, nenhum < 0,05, soma 1 ± 1e-6.
- Com fixture de 50 exposições/braço → `pesos inalterados (burn-in)`.
- `python3 -I tools/replay.py fixtures/eventos.jsonl` → regret acumulado do Thompson ≤ do ε-greedy (0,1) em ≥ 9 de 10 seeds.
- `pnpm --filter engine test` → inclui `srm por período` e `guard-rail dispara com bounce +2 p.p.`.

**Riscos**: volume (`docs/05 §3.3`): abaixo de ~2.000 sessões elegíveis/mês o `optimize` deve recusar com mensagem clara.

### Fase 4 — Geração por IA e instalação por agente completa (≈ 6 dias)

**Escopo**: `@popsmart/mcp` (stdio) com as 12 ferramentas de `docs/03 §2.3`; `gerar_variantes`/`criar_jornada` devolvendo brief + schema + exemplos; `SKILL.md` (Agent Skills) + `setup.md` + `llms.txt` + páginas `.md` no GitHub Pages; prompt copiável e deeplink do Cursor na landing; provedor opcional `config.ia` (chave do cliente, `ollama` suportado) para `popsmart gerar` sem agente; publicação no npm e registro no MCP Registry (`mcpName`).

**Pronto quando**
- `claude mcp add popsmart -- npx -y @popsmart/mcp && claude -p "liste as ferramentas do popsmart"` → 12 nomes.
- `echo '{"jornadaId":"orientacao-ia","elemento":"welcome.titulo","n":3}' | npx popsmart mcp-call gerar_variantes` → JSON com `brief`, `schema`, `exemplos`, `proximoPasso` e **nenhuma chamada de rede** (`strace`/log vazio).
- Teste de fumaça com agente real em fixture HTML, a partir só de "Leia setup.md e instale um quiz de 3 perguntas": ao fim, `npx popsmart validate` → `OK` e `git status --short` toca só `index.html` e `popsmart/`.
- `npx skills add inematds/<nome>` → skill listada; `npx popsmart gerar --variantes welcome.titulo --provedor ollama --modelo qwen2.5:7b` (chave/local do cliente) → 3 variantes válidas.
- `curl -s https://registry.modelcontextprotocol.io/v0/servers?search=<nome>` → entrada publicada.

**Riscos**: wizard quebrar projeto do cliente (mitigação: `--dry-run` padrão, recusa de working tree sujo, só toca `popsmart/` + 1 arquivo de layout); prompt injection via conteúdo do site lido pelo agente (a skill instrui a tratar conteúdo como dado).

### Fase 5 — Multi-site / SaaS opcional + WordPress + contextual (≈ 10+ dias)

**Escopo**: plugin `popsmart-wp` (GPL) na diretoria do WordPress.org; coletor multi-tenant com chave pública por site; MCP remoto com OAuth (métricas de vários sites); painel de leitura (`apps/painel`); bandit contextual (LinUCB/TS logístico ou árvore) ligado só quando `docs/05 §3.5` fase 4 for atendida; templates por vertical (educação, SaaS, e-commerce); EN/ES do widget, docs e skill.

**Pronto quando**
- `wp plugin install popsmart-wp --activate && curl -s https://<site-wp>/ | grep -c popsmart.js` → `1`.
- `npx popsmart doctor https://<site-wp>` → `✓`.
- `pnpm --filter engine test:contextual` → replay mostra ganho ≥ 5 % relativo sobre Thompson por segmento em fixture com heterogeneidade; em fixture homogênea, não liga.
- Landing/guia PT/EN/ES publicados (`projetos-landing-guia`) e cards no portal.

**Riscos**: SaaS exige suporte, cobrança e LGPD como operador; é a única fase que cria custo recorrente — pode nunca acontecer e o projeto continua útil.

## 2. Piloto inema.club — "agente de orientação"

**Contexto verificado**: portal Next.js (app router, rotas `(pt)/(en)/(es)`, `src/app/llms.txt`, componente `AgenteChat` já existente), repo `NeiMaldaner/portal` publicado via `git push origin` (deploy automático, `~/projetos/portal/CLAUDE.md`), catálogo em `src/data/courses.data.json` com **320 cursos** e tags (as mais frequentes: IA 262, Agentes 89, Claude Code 84, Skills 36, Vídeo 31, Automação 30, MCP 22, Codex 20, Produtividade 20, Iniciante 19, Negócios 17, Prompts 15, Design 12, Vendas 12) [fato: leitura local em 10/10/2026]. Cursos são gratuitos ⇒ não há "receita"; o valor é **a pessoa chegar ao curso certo e voltar**.

### 2.1 Jornada `orientacao-ia` (rascunho)

| Etapa | Tipo | Conteúdo | Mapeamento |
|---|---|---|---|
| 1 | `boasvindas` | "Em 3 perguntas eu te indico por onde começar." | — |
| 2 | `pergunta` (`objetivo`) | O que você quer fazer com IA? · usar no dia a dia · criar agentes que trabalham por mim · fazer vídeos/conteúdo · aplicar no meu negócio | +pontos: Produtividade/Iniciante · Agentes/Claude Code/Codex · Vídeo · Negócios/Vendas/Consultoria |
| 3 | `pergunta` (`nivel`) | Já usou Claude Code ou Codex? · nunca · já instalei · uso todo dia | Iniciante · (neutro) · Skills/MCP/Arquitetura |
| 4 | `pergunta` (`tempo`) | Quanto tempo tem por semana? · 1 h · 3 h · 10 h+ | prefere cursos "Quick"/v6 · v2 · trilhas longas [inferência sobre o catálogo] |
| 5 | `recomendacao` | top-3 cursos por soma de pontos nas tags (desempate: mais recente) com CTA "Começar" | `fonte: catalogo` = `courses.data.json` filtrado no `build` para `{id, title, tags, url}` (~320 × 100 B ≈ 32 KB ⇒ carregar sob demanda, não no core) |
| 6 | `compromisso` | "Quer que eu te lembre? Entre no Telegram / deixe o e-mail" (opcional, consentimento explícito) | meta secundária `lead` |
| 7 | `final` | "Salvei sua trilha neste navegador." | `perfil.objetivo`, `perfil.nivel` ⇒ audiência de jornadas futuras |

**Gatilhos**: desktop `tempo 20 s` ou `scroll 50 %` ou `saída`; mobile `scroll 50 %` ou `scrollUp`, layout `slide-in` (nunca modal na entrada). **Audiência**: `path contains /cursos` ou home; `novo = true` **ou** `perfil.objetivo exists = false`; excluir quem já clicou numa recomendação (`acao.clique_recomendacao`). **Frequência**: 1/sessão, 7 dias após fechar, nunca após concluir. **Holdout**: 10 %. **Consentimento**: modo `nenhum` no início (ID efêmero, sem e-mail) → `proprio` quando a etapa 6 entrar.

### 2.2 Experimentos do piloto

1. `ab`: título da etapa 1 (3 variantes escritas pelo Codex via `gerar_variantes`), métrica `clique_recomendacao`.
2. `ab`: 3 perguntas vs 2 perguntas (jornada A/B), métrica `conclusao_jornada`; guarda `bounce`.
3. (fase 3) `bandit` por contexto `dispositivo × novo`.

### 2.3 Métricas do piloto

- Primária: **clique na recomendação / elegíveis** (incremental vs holdout: no holdout, cliques em cursos pela navegação normal na mesma sessão).
- Secundárias: conclusão por etapa; distribuição de `objetivo`/`nivel` (insumo editorial: que público chega); tempo até o clique.
- Guarda: dismissal; bounce pós-impressão; páginas/sessão vs holdout.
- Posterior (fase 2+): retorno em 7 dias (`visitante` persistido só com consentimento); inscrição Telegram/e-mail.

### 2.4 Volume esperado e o que dá para concluir

Não temos a audiência mensal do inema.club nesta pesquisa [lacuna — pegar no Umami/Vercel/Search Console]. Com baseline de clique ~5 % e MDE 30 %, o A/B com holdout 10 % precisa de ~21 mil visitantes elegíveis (`docs/05 §2.2`). Se o tráfego for menor, o piloto mede **funil e dano** (sempre possível) e deixa o A/B rodando mais tempo; o `report` deve dizer "insuficiente" quando for o caso.

### 2.5 Pronto do piloto

- `git -C ~/projetos/portal log --oneline -1 -- popsmart/` → commit do piloto; `curl -s https://inema.club/popsmart/config.json | jq .jornadas[0].id` → `orientacao-ia`.
- `npx popsmart doctor https://inema.club` → `✓` em todos os itens; Lighthouse mobile da home antes/depois: CLS igual, performance −0 a −2 pontos no máximo [estimativa].
- Após 14 dias: `npx popsmart report --site inema --html` → relatório com funil, holdout, dismissal, SRM e a frase "evidência suficiente/insuficiente".

## 3. Decisões em aberto para o Nei

1. **Nome**. "popsmart" colide com **Popupsmart** (popup builder, marca POPUPSMART depositada no USPTO em 2021 nº 90797788 — status não confirmado; `popsmart.com` é de terceiro e redireciona para `popsmarttechnologies.com`; usuário GitHub `popsmart` existe desde 2013) [fato: `docs/01 §2.3` e RDAP; inferência: risco alto de confusão, não é parecer jurídico]. Alternativas verificadas em 10/10/2026 (npm livre = `registry.npmjs.org` 404; `.com` = sem resposta desta rede, **confirmar num registrador**): **popguia** (guia = orientação; lê bem em PT/ES), **questpop** (internacional, "quest" = jornada/missão), **jornadapop** (descritivo em PT). `popquest.com` e `popjourney.com` já respondem (ocupados). Não decidido.
2. **Licença**: MIT em tudo (recomendado) ou MIT no cliente + AGPL no coletor/painel para proteger um SaaS futuro?
3. **Sink de eventos do piloto**: coletor próprio (Cloudflare Worker + D1, conta inematime) ou enviar para o Umami/analytics que o portal já usa? (Define se a Fase 1 inclui o Worker.)
4. **Onde o popup aparece no inema.club**: home + `/cursos`, só `/cursos`, ou também páginas de curso (`inematds.github.io/*`, que são outros repos)?
5. **Relação com o `AgenteChat`** existente no portal: o popup chama o chat, substitui a abertura dele, ou são independentes com cooldown comum?
6. **Holdout do piloto**: 10 % (recomendado) ou 20 % para medir dano mais rápido?
7. **Coleta de e-mail/Telegram** já no piloto (exige modo `proprio` de consentimento e texto de privacidade no portal) ou só na 2ª rodada?
8. **Idioma das ferramentas do MCP e dos arquivos**: nomes em português (`criar_jornada`) como proposto, ou inglês para adoção internacional (`create_journey`) com docs PT/EN/ES?
9. **Editor visual**: confirmar que o MVP não terá UI de edição (só agente + JSON + `preview`), ou reservar fase para um editor mínimo?
10. **Publicação**: npm sob `@inematds/*` ou sob o nome do projeto? Registrar no MCP Registry já na Fase 4?
11. **WordPress no v1**: plugin fino já na Fase 1 (há público INEMA em WP?) ou Fase 5?
12. **Provedor de IA opcional** (`config.ia`): oferecer `ollama` como padrão sugerido na doc (sem custo, local) e deixar OpenAI/Anthropic/OpenRouter só como exemplos de configuração do cliente?
13. **Roleta/gamificação**: manter fora por padrão (risco Lei 5.768/71, `docs/06 §6`) e oferecer "revelar cupom" determinístico + Learn & Earn; ou implementar roleta com flag e aviso legal?
14. **Engenharia reversa da Alia por DevTools/Playwright** (loja demo tem senha; só lojas públicas de clientes): fazer na Fase 1 para calibrar gatilhos, ou pular?

## 4. Esforço total (estimativa)

| Fase | Dias | Acumulado |
|---|---|---|
| 0 | 2 | 2 |
| 1 | 10–12 | 12–14 |
| 2 | 5 | 17–19 |
| 3 | 5 | 22–24 |
| 4 | 6 | 28–30 |
| 5 | 10+ | 38+ |

Dias de agente com revisão do Nei nos portões de cada fase; o piloto roda em paralelo às fases 2–4 (14 dias de coleta não bloqueiam código).
