# 05 — Algoritmos e métricas: triggering, A/B → bandit, guard-rails

> Fatos com link; limiares sem fonte são `[inferência]` nossa e devem ser recalibrados com dados do piloto. Tamanhos de amostra abaixo foram calculados pela fórmula clássica de duas proporções (α = 5 %, poder 80 %, bilateral) e **não** conferidos no calculador do Evan Miller (ele roda em JS); confira em https://www.evanmiller.org/ab-testing/sample-size.html antes de fixar metas.

## 1. Smart triggering (quando mostrar)

### 1.1 Gatilhos e heurísticas publicadas

| Gatilho | Desktop | Mobile | Fonte |
|---|---|---|---|
| **Exit intent** | `mouseleave` no `document` com `relatedTarget == null` e `clientY ≤ 20 px` (ouibounce usa sensibilidade 20 px, não dispara no 1º segundo, 1× por visitante) | não existe cursor; sinais: **scroll rápido para cima** (PushOwl sugere > 15 px/frame como ponto de partida), botão voltar (OptiMonk: só na página de entrada e após interação), inatividade, troca de aba | [fato: https://github.com/carlsednaoui/ouibounce · https://www.pushowl.com/blog/mobile-exit-intent-popups · https://support.optimonk.com/en/articles/11874630-how-can-i-display-popups-on-mobile-devices] |
| **Tempo na página** | padrão da Alia: 5 s | idem | [fato: HC/11523043] |
| **Scroll depth** | 25/50/75 % | idem | [fato: HC/13513698] |
| **Inatividade** | OptiMonk: parou de rolar e tocar | idem | [fato: OptiMonk mobile triggers] |
| **Clique / evento JS** | seletor ou `push({type:'open'})` | idem | [fato: HC/16977067] |
| **Adaptive timing** | atraso aprendido por grupo dentro de [min, max] | idem | [fato: HC/17455652] |

Benchmark de vendor (Popupsmart, 3.034 popups, ≥ 1.000 exibições cada): exit intent 1,10 % lead/exibição vs 0,78 % sem; **página importa mais que gatilho** (carrinho/checkout 4,57 % vs home 1,07 %); esperar antes de mostrar 0,83 % vs 0,61 % no load, e a duração da espera não mudou nada mensurável — o próprio texto diz que a diferença do exit intent pode ser acaso [fato: https://popupsmart.com/blog/do-exit-intent-popups-work · https://popupsmart.com/blog/popup-timing].

Recomendação para o engine [inferência]:
- Desktop: `saida` (20 px, após 3 s de página) **OU** `scroll ≥ 50 %` **OU** `tempo ≥ 20 s`, o que vier primeiro, respeitando frequência.
- Mobile: **nunca overlay na entrada** (SEO, `06`); `scrollUp` rápido ou `scroll ≥ 50 %` ou `inatividade 15 s`; layout `barra`/`slide-in` por padrão.
- Manipular `history` para capturar "voltar" só com opt-in explícito (Piano recomenda desligar em páginas críticas para SEO [fato: https://docs.piano.io/en/subscriptions/composer-events]).
- `adaptativo`: só na fase bandit, aprendendo o atraso por grupo em 3 buckets (0–5 s, 5–15 s, 15–30 s) como braços de um Thompson.

### 1.2 Frequência e reexibição (caps)

| Regra | Indústria | Padrão proposto |
|---|---|---|
| Por sessão | Wisepops: a cada carregamento / por sessão (acaba com navegador fechado ou 2 h sem página) / a cada X dias ou minutos / limite de exibições [fato: https://support.wisepops.com/en/articles/9902047-popup-frequency-settings] | **1 por sessão** |
| Após fechar | Klaviyo: "show again X days after closing" (0 = cada sessão) [fato: https://community.klaviyo.com/marketing-30/how-to-make-sure-sign-up-forms-are-shown-only-once-7490]; Alia: show-again 10 dias no exemplo [fato: HC/12569519] | **7 dias** (configurável) |
| Após converter | Wisepops/Klaviyo/Alia: parar após CTA/envio; Alia admite que supressão não é 100 % [fato: HC/11492997] | **nunca** (por `perfil.convertido`, hash de e-mail opcional, `acao.*`) |
| Cooldown global entre popups | Alia sugere ~3 min [fato: HC/15244586] | **180 s** |
| 2ª chance na sessão | Alia Smart Trigger: reabre 1× só para quem fechou e tem alta probabilidade [fato: HC/11719824] | fase bandit: braço "reabrir em exit intent" vs "não reabrir" |
| Teaser após fechar | Alia floating button [fato: HC/16499160] | `layout: teaser` opcional |

Tudo gravado em `localStorage` (chaves documentadas) e refletido no evento `fechou`/`conversao` para o relatório.

## 2. A/B com grupo de controle (holdout)

### 2.1 Por que holdout

Sem um grupo que **não vê popup**, só se sabe qual popup é melhor, não se o popup ajuda. O holdout mede o incremental e detecta dano (bounce, menos páginas/sessão) [inferência; princípio padrão]. Padrão proposto: **10 % holdout permanente** em toda jornada ativa, com alocação determinística por hash (`04 §4.2`).

### 2.2 Tamanho de amostra (por braço, 50/50; conversão = taxa primária)

| Baseline | MDE rel. 20 % → alvo | n/braço (80 %) | n/braço (90 %) | MDE rel. 30 % → alvo | n/braço (80 %) | n/braço (90 %) |
|---|---|---|---|---|---|---|
| 2 % | 2,4 % | 21.109 | 28.258 | 2,6 % | 9.798 | 13.116 |
| 3 % | 3,6 % | 13.914 | 18.626 | 3,9 % | 6.455 | 8.640 |
| 5 % | 6,0 % | 8.158 | 10.921 | 6,5 % | 3.780 | 5.060 |

Com holdout de 10 % (10 % controle / 90 % tratamento) o controle precisa de ~56 % do n acima e o total de visitantes é 10× o controle [inferência, cálculo nosso]: baseline 2 %/MDE 30 % ⇒ ~5.400 no controle, ~54 mil visitantes elegíveis; baseline 5 %/MDE 30 % ⇒ ~2.100 / ~21 mil.

### 2.3 Regras de validade

- **Sem peeking** com p-valor fixo: olhar 10× e parar no 1º "significativo" transforma 1 % reportado em ~5 % real [fato: https://www.evanmiller.org/how-not-to-run-an-ab-test.html]. Ou fixar n antes, ou usar teste sequencial (mSPRT/SPRT; Statsig diz que não há penalidade por olhar [fato: https://docs.statsig.com/experiments-plus/sprt]; calculadora sequencial do Evan Miller: https://www.evanmiller.org/ab-testing/sequential.html).
- **SRM** (sample ratio mismatch): qui-quadrado das contagens por braço vs alocação; alertar com p < 0,001 (Harness, Kameleoon) [fato: https://developer.harness.io/docs/feature-management-experimentation/experimentation/experiment-results/analyzing-experiment-results/sample-ratio-check · https://help.kameleoon.com/experiment-analytics/statistical-methods/sample-ratio-mismatch]. O `report` calcula sempre.
- Decidir pelo **incremental sobre o holdout**, não só entre variantes.

## 3. De A/B para bandit

### 3.1 Qual algoritmo

| Algoritmo | Quando | Fonte |
|---|---|---|
| **Thompson Beta-Bernoulli** | padrão para conversão binária rápida; GrowthBook usa com piso de 1 % por variação | [fato: https://docs.growthbook.io/bandits/overview] |
| ε-greedy | baseline de auditoria; VWO mistura ε-greedy + Thompson com ~10 % de exploração | [fato: https://help.vwo.com/hc/en-us/articles/900003845666-Understanding-the-Working-of-Multi-Armed-Bandit-in-VWO] |
| UCB1 | quando se quer determinismo | — |
| **LinUCB / TS logístico** | muitas combinações de contexto; no dataset Yahoo deu +12,5 % de cliques sobre bandit sem contexto, e a vantagem cresce com dados escassos | [fato: https://arxiv.org/abs/1003.0146] |
| Árvore + Thompson por folha (GrowthBook contextual, beta enterprise) | segmentos sem tráfego são agrupados (`maxLeaves`) | [fato: https://docs.growthbook.io/bandits/contextual] |

Tutorial de referência para Thompson: Russo et al. [fato: https://arxiv.org/abs/1707.02038].

### 3.2 Quando bandit vence e quando não

- **Vence**: objetivo é reduzir o custo do teste (não aprender), ≥ 5 variações, uma só métrica, campanhas sempre ativas/sazonais [fato: GrowthBook overview · https://experienceleague.adobe.com/en/docs/journey-optimizer/using/content-management/content-experiment/technotes/mab-vs-ab].
- **Perde**: 2–4 variações com várias métricas e necessidade de lift preciso (A/B pode até selecionar mais rápido com 2 braços) [fato: GrowthBook overview]; Optimizely: MAB não produz significância, serve para promoções/títulos curtos, nunca para hipótese exploratória [fato: https://docs.developers.optimizely.com/feature-experimentation/docs/run-a-mab-optimization].
- **Recompensa atrasada** enviesa: coortes recentes ainda estão na janela; GrowthBook pesa por coorte e recomenda cadência ≥ tempo natural de conversão [fato: GrowthBook overview]. Regra nossa: **recompensa do bandit = métrica rápida** (conclusão da jornada, clique, lead); receita/compra se mede no holdout [inferência].
- **Não-estacionariedade**: Thompson com janela deslizante/desconto (Trovò et al.) [fato: https://arxiv.org/pdf/2409.05181]; com tráfego baixo, janela curta joga dados fora [inferência].

### 3.3 Volume mínimo (única fonte com números: GrowthBook)

- ≥ **100 usuários por variação** antes de atualizar pesos; recomenda **40 conversões por variação** antes de começar; cadência mínima **15 min**; TTL do cache do SDK ≤ 1/5 da cadência, senão SRM [fato: https://docs.growthbook.io/bandits/config].
- Optimizely, VWO, Statsig, Dynamic Yield: sem mínimo numérico publicado [lacuna].
- Regra prática derivada [inferência]: com baseline 3 %, 40 conversões ≈ 1.300 exposições por braço só para sair do burn-in; diferenciar lift de 20 % continua pedindo milhares. **Abaixo de ~2.000 sessões elegíveis/mês por jornada, bandit não compensa** — ficar em regras + A/B longo ou aceitar MDE ≥ 50 %.

### 3.4 Implementação no engine (Thompson Beta-Bernoulli)

```
para cada braço a: α_a = prior_α + conversões_a ; β_a = prior_β + (exposições_a − conversões_a)
amostrar θ_a ~ Beta(α_a, β_a) para cada a (M = 10.000 amostras Monte Carlo no optimize)
peso_a = P(θ_a é o maior) ; aplicar piso (5 % por braço) e renormalizar
holdout fixo fora dos pesos ; gravar weights.json {experimento, periodo, pesos, n, conv, atualizadoEm}
```
- Prior fraco informativo: `Beta(2, 98)` para baseline ~2 % em vez de `Beta(1,1)` [inferência].
- Contexto: um par (α, β) por combinação de `contexto[]`, com prior = posterior global encolhido (pooling); combinação com < 100 exposições usa o global.
- Cadência: diária por padrão; 6 h se ≥ 2.000 exposições/dia; nunca < 15 min.
- Sticky por coorte de período (`04 §4.2`); SRM só dentro de cada período de pesos fixos ou no holdout [inferência].
- Validação offline: replay (Li et al. 2010) com MABWiser/contextualbandits sobre eventos reais antes de ligar em produção.

### 3.5 Sequência recomendada e limiares

| Fase | Condição de entrada | O que roda |
|---|---|---|
| **1. Regras** | sempre | gatilhos + caps + holdout 10 % (só mede) |
| **2. A/B** | ≥ ~20 mil sessões elegíveis/mês (cobre baseline 5 %/MDE 30 % com holdout em ~1 mês) — ou aceitar MDE maior | 2–3 variantes, n fixo ou sequencial, SRM, decisão pelo incremental |
| **3. Thompson por segmento** | ≥ 100 exposições e ≥ 40 conversões por braço por segmento; 2–4 braços; ≤ 4 segmentos grossos | pesos diários, piso 5 %, holdout permanente 5–10 % |
| **4. Contextual** | melhor braço difere entre segmentos **e** ≥ 500 conversões/mês por segmento; > 6 combinações de contexto | LinUCB/TS logístico ou árvore; validado por replay; manter só se ganho ≥ 5 % relativo sobre a fase 3 no holdout |

## 4. Métricas

| Classe | Métrica | Definição | Uso |
|---|---|---|---|
| Primária | **conversão por visitante elegível** (lead, clique na recomendação, conclusão da jornada…) | conversões ÷ elegíveis (inclui holdout no denominador quando se mede incremental) | decisão A/B; recompensa do bandit |
| Co-primária | **receita (ou valor) por visitante** | soma de `meta.valor` ÷ elegíveis | só no holdout / relatório |
| Secundárias | AOV; **taxa de conclusão por etapa** (funil da jornada); tempo até converter | — | diagnóstico da jornada |
| Dano / guarda | **dismissal rate** (fechou sem agir); **bounce pós-impressão** (saiu em < 10 s após ver); páginas/sessão; micro-CSAT opcional ("isso ajudou?") | comparadas com holdout | parar jornada se dismissal > 85 % ou bounce + 2 p.p. vs holdout [inferência] |
| Operacionais | SRM; exposições/braço; conversões/braço; dias de teste; peso do script; erros do widget | — | `report`; alertas |

Popupsmart recomenda medir lead rate, bounce e receita por visitante juntos para o vencedor não custar engajamento [fato: https://popupsmart.com/blog/do-exit-intent-popups-work].

No piloto inema.club (cursos gratuitos) a "receita" é substituída por **clique no curso recomendado**, **inscrição (newsletter/Telegram)** e, depois, **retorno em 7 dias** — ver `PLANO.md`.

## 5. Guard-rails (obrigatórios no engine)

1. **Holdout** permanente (padrão 10 %, mínimo 5 %) por jornada; nunca zero sem `--sem-holdout` explícito.
2. **Frequência**: 1/sessão, 7 dias após fechar, nunca após conversão, cooldown global 180 s (configuráveis, com mínimos).
3. **Limite de incentivo**: `limites.descontoMaxPct` e `limites.incentivoMaxPorLead` (custo = desconto médio × taxa de resgate × ticket ÷ leads) — `validar_jornada` recusa recompensa acima do teto; braço cujo custo por lead passe do teto é pausado pelo `optimize` [inferência].
4. **Piso de exploração** 5 % por braço no bandit; braço só é removido por humano (ou `--auto-podar` com ≥ 95 % de probabilidade de ser pior por 7 dias).
5. **Burn-in**: pesos não mudam antes de 100 exposições e 40 conversões por braço.
6. **Métrica de guarda** sempre calculada vs holdout; alerta para o `report` quando piorar além do limiar.
7. **SRM** p < 0,001 ⇒ experimento marcado inválido até revisão.
8. **Recompensa rápida** no bandit; métricas lentas só em relatório.
9. **Sem dark patterns** (ver `06`): o validador bloqueia confirmshaming, botão de fechar < 24 px, contador falso, roleta aleatória.
