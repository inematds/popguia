# 01 — Pesquisa de mercado: Alia em profundidade + concorrentes

> Pesquisa feita em 10/10/2026 só por leitura pública (WebSearch/fetch HTTP, `gh`), sem login, sem contornar proteção, sem API de modelo. Convenção: `[fato: URL]` = lido na fonte hoje; `[inferência]` = leitura nossa. Números de vendors são autodeclarados.
>
> Abreviações: `HC/<id>` = `https://intercom.help/alia-help-center/en/articles/<id>`; `SHOP` = `https://apps.shopify.com/alia`.

## 1. Alia Popups — o que ela é de verdade

### 1.1 Posicionamento

- App de popups de e-mail/SMS **só para Shopify** ("we do not integrate directly with any other platform besides Shopify") [fato: https://www.aliapopups.com/pricing]. App só em inglês [fato: SHOP].
- A IA aparece em quatro camadas: **Prism AI** (personalização por contexto), **Smart Testing** (A/B em sequência), **Smart Trigger** (reabre na hora certa) e **Smart Offers** (escolhe a faixa de desconto). Todas **escolhem, cronometram ou testam o que o lojista criou**; nenhuma gera o criativo sozinha, fora do assistente "Lia" do editor [fato: HC/14771905, HC/15158651].
- Preço na Shopify: "From $100/month" — US$ 100 (20 mil visitantes/mês), 200 (50 mil), 300 (100 mil), 400 (300 mil); 14 dias grátis [fato: SHOP]. No site, o plano Basic aparece como US$ 200/mês até 20 mil visitantes e "Fully Managed" US$ 400 [fato: https://www.aliapopups.com/pricing]. **Os dois canais divergem** e não achamos explicação documentada [inferência]. Cobrança por **visitantes únicos rastreados pela Alia**, que diferem da contagem da Shopify [fato: HC/16102228, HC/15604314].

### 1.2 Modelo de dados (entidades extraídas da central de ajuda)

| Entidade | O que é | Fonte |
|---|---|---|
| **Campaign** | Contêiner. Tipos fixos: Standard, Embedded Form, Subscription Upsell (tipo não muda depois) | HC/17039841 |
| **Popup** | Unidade de design dentro da campanha; também é a variante de A/B ("Add test variant", "Duplicate as draft") | HC/11062795, HC/15392050 |
| **Step** | Telas: Welcome, Form (e-mail/SMS), Thank You, mini-quiz, reveal de recompensa; abrir direto numa etapa via "Open step" | HC/12569406, HC/17507095 |
| **Element / Component** | Texto, imagem, botão, form, Question (mini-quiz), compliance SMS; elemento vira Component reutilizável | HC/15217141 |
| **Prism element** | Dois tipos: *Static* (variantes filhas com condição de visibilidade, por regra) e *Prism AI* (2–5 variantes filhas escolhidas pelo modelo) | HC/14596944 |
| **Targeting rule** | Regras globais reutilizáveis (Hide Pages, New/Returning Visitors) + condições por campanha | HC/15244586, HC/11719788 |
| **Trigger** | Por popup; vários coexistem | HC/13513698 |
| **Reward** | Static Code, Unique Discount Code (prefixo `AL`, gerado na Shopify), Free Shipping, Smart Reward (faixas de desconto por modelo de intenção) | HC/15408225, HC/15158651 |
| **Profile / `$alia.user`** | `createdAt`, `shopifyCustomerID`, `country`, `market`, `language`, `email`, `phone`; `profile.properties` guarda respostas do quiz (chave = texto da pergunta) | HC/11386897 |
| **Saídas de perfil** | `alia_offer`, `alia_offer_text`, nome do popup → Klaviyo/Postscript; respostas do quiz viram *customer tags* na Shopify (só o texto da resposta, só a 1ª em multi-seleção) | HC/15158651, HC/16379968 |
| **Floating button** | Teaser que fica após fechar o popup | HC/16499160 |
| **Smart Test / Test Packs** | "Optimization Essentials": até 9 testes e 40+ variantes em sequência | HC/15257852 |

- **Não existe entidade "journey"** na documentação; o equivalente é a sequência de Steps + ramificação por Prism estático + propriedades do quiz [inferência]. "Learn & Earn" também não é um recurso técnico nomeado, é uma jornada em etapas com desconto condicionado [inferência a partir do case MASA].
- **Acesso externo**: API pública **só de leitura** (`read:events`, `read:segments`); sem API de escrita, sem webhooks próprios, sem feed de pedidos atribuídos [fato: HC/14648362]. SDK JS com eventos `alia:campaignMatched`, `alia:popupView`, `alia:popupClose` e abertura programática `window.alia.push({type:'open', campaignID, ignoreTargeting})` [fato: HC/14648362, HC/16977067]. **MCP só de leitura** para Claude/ChatGPT, para consultar analytics [fato: HC/16836583].

### 1.3 Gatilhos e segmentação

**Gatilhos** [fato: HC/11523043, HC/13513698, HC/17507095, HC/17455652, HC/11719824]:
- Tempo na página (padrão 5 s); scroll depth em % (25/50/75); exit intent (desktop = mouse sai da página; mobile = scroll rápido para cima; cada dispositivo com seu "Applies to" e "Open step").
- **Adaptive Timing**: atraso aprendido por grupo dentro de um mín/máx (ex.: 0–20 s); grupos por página, UTM, dispositivo/navegador/SO, país/região/cidade, Shopify market. A divisão não pode ser trocada depois de publicar.
- Produto no carrinho (Subscription Upsell); programático via SDK; **Smart Trigger** reabre uma 2ª vez na sessão só para quem fechou e tem alta probabilidade de converter.
- Gatilho de **inatividade**: não encontrado em nenhum artigo [inferência: não existe ou não documentado].

**Segmentação** [fato: HC/11719788, HC/11492997, HC/15244586, HC/15729067, HC/15392921, HC/15704599, HC/16607541, HC/14157937]:
- Novo vs recorrente (identificador próprio em localStorage + cookies de ESP + login Shopify + UTMs); página (path, "is not on", "contains"); UTM source/medium/campaign; país/Markets/geo-IP; segmentos e listas do Klaviyo; propriedades do perfil (resposta do quiz); ação já concluída ("Has completed action: Email signup"); consentimento de cookies ("Cookie consent allowed"); código custom via `$alia`.
- Gerador de regras de segmentação por IA em linguagem natural, com prévia [fato: HC/15149390].

### 1.4 Reexibição e frequência

- "Show-again delays" configuráveis (exemplo da central: 10 dias) [fato: HC/12569519].
- Cooldown entre popups: "Don't trigger if another popup was viewed [in the last X]" (sugestão: ~3 min) [fato: HC/15244586].
- Supressão após cadastro combina identificador próprio + ESP + login + UTM; no Safari o identificador dura 7 dias se o visitante não voltar; a própria Alia admite que **não há garantia de 100%** e que "normaliza após algumas semanas" [fato: HC/11492997, HC/11719788].
- Smart Trigger reabre **uma única vez por sessão**, pode disparar antes do atraso configurado, e **não personaliza o conteúdo** da reaparição [fato: HC/11719824].
- Lição para nós: a dor real de reexibição é *identidade entre dispositivos e storage volátil*; Alia resolve "mais ou menos". Projetar desde o início com `visitor_id` + `profile.converted` + supressão por hash de e-mail opcional [inferência].

### 1.5 O que o Prism otimiza e como distribui

- Métricas-alvo documentadas: **taxa de envio de e-mail, de SMS e de pedidos atribuídos** (janela de 30 dias) [fato: HC/14775378]. AOV/receita/LTV aparecem só no marketing ("sitewide KPIs") [fato: home e SHOP] — o painel documentado mostra só as três taxas [inferência].
- Fases: **Learning** (divisão uniforme e aleatória) → **Optimizing** (tráfego vai para as melhores variantes **por dimensão**, começando pelos segmentos de maior tráfego) [fato: HC/14771905, HC/14775378, HC/14596944]. Dimensões: 1–3 (recomendadas Path, UTM Source, UTM Campaign) [fato: HC/14596944]. "Lift" = otimizado vs baseline de divisão igual, ajustado pela confiança; pede "algumas centenas de views por variante" [fato: HC/14775378].
- Não é chamado de *bandit* em lugar nenhum; o comportamento descrito é exploração uniforme seguida de exploração dirigida por contexto — compatível com **contextual bandit em lote** [inferência].
- Smart Testing: vencedores por significância estatística; esconde cedo variantes muito ruins; reinicia se o controle for editado; "treinado em 500 milhões de views de 2.000+ lojistas" [fato: HC/12131764]. Smart Trigger: "20–40% mais cadastros em média" (número próprio) [fato: HC/11719824].

### 1.6 O que o Prism NÃO faz (limites úteis para nós)

- Não cria design nem copy; a equipe define as variantes [fato: HC/14771905].
- Não aceita "variante X na página Y" fixa (isso é o Prism estático, sem otimização) [fato: HC/14771905].
- Não usa histórico de compras nem dados de cliente conhecido; só desempenho anônimo + dimensões escolhidas [fato: HC/14596944, HC/14771905].
- Sem API de escrita, sem webhooks, sem feed de pedidos; MCP só leitura [fato: HC/14648362, HC/16836583].
- Copiar popup com componentes Prism para outra loja quebra ("component not found") [fato: HC/14596944].

### 1.7 Reviews, case e sinais de mercado

- Shopify: **4,6 com 114 avaliações** (confirmado hoje); distribuição 110×5★, 1×2★, 3×1★ — a média simples seria ~4,87, a nota exibida é outra fórmula [fato: SHOP/reviews; inferência sobre a fórmula].
- Reclamações (4 negativas, amostra pequena) [fato: SHOP/reviews]: preço ("not worth $400", "overpriced at $100/month"); dificuldade de configurar e editor sem drag-and-drop; tradução limitada; regressão após atualização; "too many drop off due to this app"; cobrança após desinstalar.
- Case MASA: +127% taxa de cadastro, 3,2× cadastros, +46% taxa de compra, "64x ROI" (cálculo não detalhado) — jornada educativa com 10% de entrada e até 20% ao concluir [fato: https://www.aliapopups.com/success-stories/masa]. Onboarding "white-glove" com A/B gerenciado pela equipe [inferência: parte do resultado é serviço, não só software].
- Reddit (post de 28/09/2026): bloqueado para leitura anônima (403 no `.json`); não verificado.
- `alia-demo.myshopify.com`: protegida por senha; não acessada.

## 2. Concorrentes

### 2.1 Tabela comparativa (preços lidos em 10/10/2026, USD/mês)

| Ferramenta | Diferencial | Cobrança e faixa | Fora do Shopify | IA | API / MCP |
|---|---|---|---|---|---|
| **OptiMonk** | Agente de IA monta popup por chat; tudo em todos os planos | Pageviews + créditos de IA. Free 10k PV; $19–29 (20–50k); $69–99 (100–250k); $179–249 (500k–1M) [fato: https://www.optimonk.com/pricing/] | Plugin WP, snippet/GTM | Agente + créditos | **MCP com escrita** (`agentic.optimonk.com/mcp`, OAuth, rascunhos) [fato: changelog OptiMonk] |
| **Wisepops** | Plataforma + estrategista (híbrido); push, feed, embeds | Pageviews; sem free; valores divergem entre fontes ($179–299 por 250–500k) | Woo, Magento, Presta, BigCommerce; tracker/GTM | Segmentação comportamental, AI Insight; testes com controle | Nenhum encontrado |
| **Justuno** | Smart Timing, recomendações, ID de visitante; "done-for-you" | Visitantes; Lite $59 (10k); DFY $299 (20k) [fato: https://www.justuno.com/pricing/] | Apps Magento/WP/BigCommerce; embed genérico | Timing/recomendação, sem geração | Nenhum público |
| **ConvertFlow** | Funis, quizzes, landings (popup é uma superfície) | Views de funil; Free; Pro $299 (30k); Plus $999+ [fato: https://www.convertflow.com/pricing] | Script no head, GTM, WP (plugin beta); SPA: `convertflow.start()` | Remix AI gera funis por chat | JS API + webhooks |
| **Popupsmart** | Builder no-code; agente + MCP + CLI (alegação própria) | Pageviews; Free 5k; $39 (50k); $69 (100k, A/B); $99 (250k); $159 (500k) [fato: https://popupsmart.com/facts-for-ai] | Snippet, GTM, plugin WP, Zapier | Builder por prompt, Smart Mode | MCP + CLI "fazem tudo que o app faz" [fato: https://popupsmart.com/llms.txt]; docs de dev não resolvem DNS |
| **Privy** | E-mail + SMS + popup p/ Shopify | Contatos mailable; e-mail a partir de $30; sem free [fato: https://www.privy.com/pricing] | Plugin WP, snippet manual | Sem builder de IA (fonte: concorrente) | MCP **só leitura** (pagos) [fato: https://help.privy.com/docs/learn/mcp.md] |
| **Klaviyo forms** | Forms ligados ao CRM/fluxos | Perfis ativos; Free 250 perfis; pago a partir de ~$20 [fato: https://www.klaviyo.com/pricing] | `klaviyo.js` com chave pública; plugin WP | IA p/ assuntos | REST + SDKs + MCP remoto (`mcp.klaviyo.com`) |
| **Octane AI** | Quiz de produto com recomendação por IA | Créditos (1 = quiz concluído); $50/$200/$500+ [fato: https://octaneai.com/pricing] | Shopify (inclusive headless) | Smart Products/Copy | REST + webhooks + JS storefront |
| **Typeform** | Formulário conversacional; embed popup/slider | Respostas; $39/$79/$129 [fato: https://www.typeform.com/pricing] | `@typeform/embed` em qualquer site | Criação por IA | Create/Responses/Webhooks/Embed + MCP |
| **Tally** | Grátis e ilimitado (uso justo) | Free; Pro $24; Business $74 [fato: https://tally.so/pricing] | `embed.js` + `Tally.openPopup()` | Não verificado | API grátis + webhooks + MCP (`api.tally.so/mcp`) |

### 2.2 Do que os usuários reclamam (padrões repetidos)

1. **Cobrança**: por pageview/visitante/perfil que infla sem controle (Wisepops cobra pageviews mesmo sem popup; Klaviyo cobra perfis falsos; Privy base + uso) e cobranças após cancelar/desinstalar (Alia, Klaviyo, Justuno, OptiMonk) [fato: reviews Shopify de cada app; links em `docs/00` e nas fontes abaixo].
2. **Editor/UX**: difícil de configurar, sem drag-and-drop (Alia), bugs no editor (OptiMonk, ConvertFlow), customização avançada limitada (Popupsmart, Octane).
3. **Suporte lento** (Privy, Wisepops no trial, Klaviyo 2–3 dias, Popupsmart).
4. **Impacto no site**: "site mais lento" (Justuno), "código com impacto em velocidade e SEO" (Popupsmart), "too many drop off" (Alia), popup atrapalhando a edição do tema (Privy).
5. **Localização**: tradução limitada (Alia), app só em inglês.
6. **Lock-in de plataforma**: Alia/Octane só Shopify; Privy/Klaviyo centrados em e-commerce.

### 2.3 Leitura para o nosso projeto

- **Espaço aberto**: nenhum dos concorrentes é (a) open source, (b) instalável por agente de código de ponta a ponta com configuração **em arquivo no repo do cliente**, (c) sem cobrança por pageview. OptiMonk e Popupsmart têm MCP de escrita, mas ambos presumem conta SaaS e builder visual [inferência a partir da tabela].
- **O que copiar de comportamento** (não de código): entidades Step/Element/Variant, regras globais reutilizáveis, "has completed action", "cookie consent allowed" como condição, Adaptive Timing por grupo, cooldown global entre popups, teaser pós-fechamento, "show again after N days", supressão após conversão com múltiplos sinais.
- **O que fazer diferente**: jornada como entidade de primeira classe (quiz → diagnóstico → recomendação), objetivos não-comerciais (aula certa, inscrição, engajamento) além de lead/pedido, holdout sem popup como padrão, métricas de dano (bounce/dismissal) sempre visíveis, suporte a HTML/Next.js/WordPress desde o v1.
- **Nome**: o nome provisório "popsmart" tinha risco alto de confusão com Popupsmart [inferência, não é parecer jurídico]; em 10/10/2026 o Nei escolheu **popguia** (livre no npm na data da pesquisa).

## 3. Fontes principais

- Alia: HC/14771905, HC/14596944, HC/14775378, HC/14771951, HC/15217141, HC/12131764, HC/15257852, HC/11719824, HC/17455652, HC/17507095, HC/13513698, HC/11523043, HC/15244586, HC/12569519, HC/11492997, HC/11719788, HC/14157937, HC/15729067, HC/15392921, HC/15149390, HC/16607541, HC/11386897, HC/15158651, HC/15408225, HC/16379968, HC/14648362, HC/16977067, HC/16836583, HC/17039841, HC/17040159, HC/12569406, HC/12402041, HC/15604314, HC/16102228, HC/15171575; https://www.aliapopups.com ( /pricing, /faqs, /success-stories/masa ); https://apps.shopify.com/alia e /reviews.
- Concorrentes: https://www.optimonk.com/pricing/ · https://www.optimonk.com/changelog/mcp-integration-build-popups-from-your-favorite-ai-tool-ca22e45a · https://apps.shopify.com/optimonk/reviews · https://wisepops.com/ · https://support.wisepops.com/en/articles/395233-add-wisepops-with-google-tag-manager · https://www.justuno.com/pricing/ · https://www.capterra.com/p/224090/Justuno/reviews · https://www.convertflow.com/pricing · https://help.convertflow.com/article/180-google-tag-manager-how-to-install-convertflow · https://popupsmart.com/facts-for-ai · https://popupsmart.com/llms.txt · https://popupsmart.com/pricing · https://www.privy.com/pricing · https://help.privy.com/docs/learn/mcp.md · https://www.klaviyo.com/pricing · https://www.klaviyo.com/solutions/ai/klaviyo-mcp-server · https://octaneai.com/pricing · https://www.typeform.com/pricing · https://developer.typeform.com/ · https://tally.so/pricing · https://developers.tally.so/widgets/popups · https://tally.so/help/mcp.
