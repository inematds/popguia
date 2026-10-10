# 06 — Conformidade: LGPD/GDPR, acessibilidade, SEO e Core Web Vitals, dark patterns

> Lido em 10/10/2026 nas fontes oficiais linkadas. Não é parecer jurídico. `[fato: URL]` = lido na fonte; `[inferência]` = aplicação nossa ao widget; `[lacuna]` = não confirmado.

## 1. LGPD (Brasil)

- O **Guia de Cookies da ANPD** (2022, atualizado 01/2025) vale "de forma geral" também para tecnologias similares de rastreamento [fato: https://www.gov.br/anpd/pt-br/centrais-de-conteudo/materiais-educativos-e-publicacoes/guia-orientativo-cookies-e-protecao-de-dados-pessoais.pdf/@@download/file]. `localStorage` e IDs próprios entram por analogia [inferência].
- Cookies **não necessários** ⇒ consentimento é a base mais adequada; necessários podem usar legítimo interesse. O guia lista como "o que evitar": botão único "aceito", mais destaque no aceitar que no rejeitar, não necessários ativados por padrão, sem 2º nível, sem revogação direta, política só em idioma estrangeiro [fato: mesmo guia].
- **Medição de audiência** por legítimo interesse (exemplo 4 do guia) exige: coleta limitada à medição, dados agregados/anônimos, sem compartilhar com terceiros, sem cruzar bases, informação no banner com prazo de retenção e oposição [fato: mesmo guia]. Estender isso a "ID de A/B" é analogia [inferência]; se o teste personaliza por pessoa ou alimenta perfil, vale consentimento [inferência].
- **E-mail e respostas de quiz identificáveis** ⇒ consentimento livre, informado, inequívoco e revogável (LGPD art. 8º §5º; término pela revogação art. 15 III; eliminação art. 16) [fato: https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/L13709compilado.htm]. Legítimo interesse para captação de lead é frágil [inferência].
- **Anonimização**: eventos sem ID persistente nem IP completo só ficam fora da lei se não reversíveis com "esforços razoáveis" (art. 12) [fato: lei].
- **Menores** (art. 14): melhor interesse; para crianças, consentimento específico e em destaque de um dos pais [fato: lei]. Enunciado CD/ANPD 1/2023 admite outras bases dos arts. 7º e 11 [fato secundário: https://lefosse.com/noticias/alerta/anpd-publica-enunciado-sobre-o-tratamento-de-dados-pessoais-de-criancas-e-adolescentes/]. Regra nossa: em contexto infantil, bloquear coleta de e-mail e segmentação [inferência].
- **Retenção**: prazo por finalidade, documentado; retenção indeterminada é incompatível (art. 16) [fato: guia ANPD].

## 2. GDPR / ePrivacy (UE) e Reino Unido

- Antes de depositar ou **ler** um rastreador: informar, obter consentimento, oferecer recusa [fato: CNIL ficha 16, https://www.cnil.fr/en/sheet-ndeg16-use-analytics-your-websites-and-applications].
- **EDPB 2/2023** (escopo técnico do art. 5(3)): informação lida por script e **enviada ao servidor** é "acesso"; identificador único coletado client-side idem; leitura puramente local sem sair do dispositivo não é (§44) [fato: https://edpb.europa.eu/system/files/2024-10/edpb_guidelines_202302_technical_scope_art_53_eprivacydirective_v2_en_0.pdf]. Logo: `visitante` em localStorage **enviado ao coletor** exige consentimento ou exceção; flag local "popup fechado" tem risco menor e é defensável como estritamente necessária [inferência].
- **Exceção CNIL para medição** (2020): finalidade restrita a medição de audiência **e A/B testing**, informar e permitir oposição, sem cruzamento, escopo de um editor, truncar último byte do IP, vida do rastreador ≤ 13 meses [fato: ficha 16]. A página de 2025 mantém 13 meses sem prorrogação automática e dados por no máximo 25 meses, mas **não repete a menção a A/B** [fato: https://www.cnil.fr/fr/cookies-et-autres-traceurs/regles/cookies-solutions-pour-les-outils-de-mesure-daudience] [lacuna].
- **ICO** (UK): a exceção "statistical purposes" lista A/B testing expressamente; exige informação e meio simples de oposição; monitorar indivíduos exige consentimento [fato: https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/guidance-on-the-use-of-storage-and-access-technologies/what-are-the-exceptions/].
- **Em tramitação**: Digital Omnibus (COM(2025) 837) revogaria o art. 5(3) e criaria isenção para medição própria agregada; status de 2026 não confirmado [fato secundário: https://www.osborneclarke.com/insights/digital-omnibus-reshapes-eu-cookie-rules-leaves-banner-fatigue-largely-intact].
- **Double opt-in (Alemanha)**: ônus da prova do consentimento é do remetente (§7 UWG); BGH I ZR 164/09; OLG München tratou o e-mail de confirmação como publicidade, OLG Düsseldorf/Celle aceitaram [fato secundário: https://www.it-recht-kanzlei.de/double-opt-in-newsletter.html · https://cms.law/de/deu/legal-updates/double-opt-in-nach-dem-olg-muenchen-beisst-sich-die-katze-in-den-schwanz]. Prática: DOI com log de IP/data/texto e e-mail de confirmação neutro [inferência].

### 2.1 Como o widget implementa (modos de consentimento em `config.consentimento`)

| Modo | Comportamento |
|---|---|
| `nenhum` (padrão para sites que não têm CMP) | **Sem ID enviado ao servidor**: eventos agregados por sessão com `visitante` efêmero (não persistido) ⇒ sem A/B sticky entre sessões, só regras + frequência local. Formulário de e-mail com checkbox de consentimento não pré-marcada e texto de finalidade. |
| `cmp` | Escuta o evento/categoria do CMP (OneTrust, Cookiebot, custom); só persiste `visitante` e liga experimentos quando a categoria (`analytics`/`necessaria` conforme o cliente decidir) estiver aceita. |
| `proprio` | O widget inclui uma etapa de consentimento mínima (aceitar/rejeitar com igual destaque, 2º nível, link de revogação) antes de persistir ID. |
| Sempre | Chaves de `localStorage` documentadas em `docs/privacidade.md` do cliente (gerado pelo CLI); IP truncado no coletor; retenção padrão 13 meses para `visitante`, 25 meses para agregados; endpoint de revogação/oposição (`window.popsmart.revogar()` apaga tudo local). |

## 3. Acessibilidade (WCAG 2.2 AA)

- Padrão **APG "Dialog (Modal)"**: foco inicial dentro do diálogo (título com `tabindex="-1"` em conteúdo longo), Tab/Shift+Tab ciclam, Esc fecha, `role="dialog"` + `aria-modal="true"` + `aria-labelledby`/`aria-label`, retorno do foco ao abridor ao fechar; só marcar `aria-modal` se o fundo for realmente inerte [fato: https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/].
- **`<dialog>` nativo**: Baseline "Widely available" desde 03/2022 [fato: https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dialog]; 97,12 % de uso global [fato: https://caniuse.com/dialog]. `showModal()` dá top layer, `::backdrop`, Esc e inert do fundo; atributo `closedby` existe no MDN mas suporte não verificado [lacuna] ⇒ feature detection.
- **2.5.8 Target Size (AA)**: alvos ≥ 24×24 CSS px, aplica-se a conteúdo que aparece por cima [fato: https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html]. Botão de fechar ≥ 44×44 em mobile [inferência].
- **2.4.11 Focus Not Obscured (AA)**: item focado ao menos parcialmente visível — nunca atrás de sticky bar ou do próprio popup [fato: https://www.w3.org/WAI/WCAG22/Understanding/focus-not-obscured-minimum.html].
- `prefers-reduced-motion` (web.dev CLS manda respeitar) [fato: https://web.dev/articles/cls]; sem timer que fecha/avança sozinho (2.2.1); sem armadilha de teclado (2.1.2); contraste AA em todos os temas; teste com NVDA/VoiceOver [inferência de rotina].
- **Shadow DOM**: CSS vars atravessam; `aria-labelledby` não cruza o shadow root; foco/`inert` do fundo tratados fora do shadow [inferência prática].
- **Leis**: LBI art. 63 obriga acessibilidade em sites de empresas com sede/representação no país, "conforme as melhores práticas e diretrizes internacionais" [fato: https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2015/lei/l13146.htm]; MPF cobra regulamentação (06/2026) [fato: https://www.mpf.mp.br/o-mpf/unidades/pr-sp/noticias/mpf-exige-que-a-uniao-defina-regras-para-garantir-acessibilidade-em-sites-publicos-e-privados]; European Accessibility Act abrange e-commerce [fato: https://commission.europa.eu/strategy-and-policy/policies/justice-and-fundamental-rights/disability/union-equality-strategy-rights-persons-disabilities-2021-2030/european-accessibility-act_en]. Alvo seguro: WCAG 2.2 AA [inferência].

## 4. SEO — Google "intrusive interstitials"

- Doc oficial (atualizada 10/12/2025): interstitials/diálogos que **obstruem a visão do conteúdo**, geralmente promocionais, prejudicam o entendimento da página e podem levar a "poor search performance". Erros: cobrir a página inteira; redirecionar para outra página para pedir consentimento/dados. Exceções: interstitials obrigatórios (idade, cookies legais) — ainda assim sobrepor, não redirecionar. Recomendação: banners que ocupem "só uma pequena fração da tela", reaproveitáveis para newsletter [fato: https://developers.google.com/search/docs/appearance/avoid-intrusive-interstitials]. Page experience pergunta "Do your pages avoid using intrusive interstitials?" [fato: https://developers.google.com/search/docs/appearance/page-experience].
- A doc atual não fala em "só mobile" nem em "penalidade"; isso é histórico de 2016–17 [fato secundário: https://www.smashingmagazine.com/2017/05/intrusive-interstitials-guidelines-avoid-google-penalty/]. Timing (atraso/exit intent) não é tratado: overlay de tela cheia disparado depois continua "obstruindo" [inferência].
- Regras do widget [inferência]: mobile ⇒ `barra`/`slide-in`/`inline` por padrão, overlay só por interação; desktop ⇒ modal ≤ 60 % da viewport por padrão; nunca na entrada vinda de busca (`document.referrer` de buscador + 1ª página da sessão ⇒ adiar); nunca redirecionar.

## 5. Core Web Vitals e peso do script

- **CLS**: web.dev manda reservar espaço ou "garantir que o elemento não faça parte do fluxo do documento, sobrepondo o conteúdo"; animações `transform` compostas não contam [fato: https://web.dev/articles/optimize-cls]. Shifts em até 500 ms após input discreto têm `hadRecentInput` [fato: https://web.dev/articles/cls]. ⇒ overlay `position: fixed` + `transform/opacity` = CLS 0 [inferência].
- **INP**: bom ≤ 200 ms no p75 [fato: https://web.dev/articles/inp] ⇒ handlers leves, yield em tarefas longas.
- **Carregamento**: `defer`, depois do `load`, `requestIdleCallback` com fallback (Safari 13.1–27.2 desabilita; ~79 % global) [fato: https://caniuse.com/requestidlecallback]. Scripts de consent costumam estar no caminho crítico (Web Almanac 2025) [fato secundário: https://almanac.httparchive.org/en/2025/third-parties].
- **Peso medido** (download público + gzip -9 local, 10/10/2026; não é a transferência real nem inclui chunks): OneTrust stub 14,0 KB · Cookiebot 28,3 KB · Typeform embed 14,2 KB · Crisp loader 2,9 KB · Intercom loader 2,0 KB · OptiMonk preload 1,7 KB · Klaviyo loader 0,5 KB (chunks não medidos) · gtag.js 153 KB [fato: medição própria]. Intercom (2019) relatou ~600 KB gz reduzidos em 65 % [fato secundário: https://www.intercom.com/blog/?p=20842]. Sem medições públicas por entidade para OptiMonk/Wisepops [lacuna].
- **Meta nossa**: núcleo ≤ 15 KB gz, total ≤ 20 KB com config, módulos sob demanda (`04 §6`) [estimativa].

## 6. Dark patterns e promoções

- **EDPB 03/2022** (deceptive design; redes sociais, aplicação a popups é analogia): Overloading (continuous prompting, privacy maze), Skipping, Stirring (emotional steering, hidden in plain sight), Obstructing (dead end, misleading action), Fickle, Left in the dark [fato: https://www.edpb.europa.eu/system/files/2023-02/edpb_03-2022_guidelines_on_deceptive_design_patterns_in_social_media_platform_interfaces_v2_en_0.pdf]. Mapeamento: reabrir a cada visita = continuous prompting; fechar quase invisível = hidden in plain sight; "Não, prefiro pagar mais" = emotional steering (confirmshaming, termo de Brignull) [inferência].
- **FTC "Bringing Dark Patterns to Light"** (2022): não aberto (403/404); resumos citam cancelamento difícil, taxas ocultas, falsa escassez, coleta enganosa [fato secundário: https://infobytes.orrick.com/2022-09-23/ftc-reports-use-dark-patterns/] [lacuna].
- **Roleta de desconto no Brasil**: Lei 5.768/71 exige autorização prévia (hoje SPA/MF) para distribuição gratuita de prêmios por sorteio, vale-brinde, concurso ou assemelhada [fato: https://www.gov.br/pt-br/servicos/obter-autorizacao-para-atividades-de-distribuicao-gratuita-de-premios-a-titulo-de-propaganda-ou-de-captacao-de-poupanca-popular]. Portaria SEAE/ME 7.638/2022: na modalidade "assemelhada a vale-brinde" é **vedada a definição do contemplado por meio randômico** (deve ser por contador ou cronograma); art. 12 veda comercializar/ceder dados coletados em promoções [fato: https://www.normaslegais.com.br/legislacao/portaria-seae-me-7638-2022.htm]. "Cupom de desconto" não aparece nomeado [fato: busca no texto]. Regulamento de shopping (2026) sustenta que "compre e ganhe" sem limite de estoque dispensa autorização, citando Nota Informativa SEI 11/2018 [fato: https://palladiumcuritiba.com.br/wp-content/uploads/2026/07/PROMOCAO-ROLETA-DE-PREMIOS-Palladium-Curitiba-Regulamento-08.07.2026.pdf] [lacuna: não validado].
  - Raciocínio: roleta com RNG e prêmios limitados ⇒ risco alto de ser sorteio/assemelhada ⇒ autorização; desconto garantido a todos com roleta "teatral" ⇒ risco de autorização cai, mas simular chance é enganoso [inferência]. **Decisão de produto**: sem roleta aleatória por padrão; mecânica "revelar cupom" determinística ou desconto condicionado a concluir a jornada (Learn & Earn); roleta só com flag explícita e aviso para consultar advogado.

## 7. Checklist do widget (25 itens — vira testes do `validar_jornada` e do `doctor`)

**Dados e privacidade**
1. Sem ID persistente enviado ao servidor antes do consentimento (ou fora das condições da exceção de medição); cada chave de `localStorage` documentada por finalidade.
2. ID de A/B sem consentimento só com: finalidade estrita, sem cruzamento, 1 editor, vida ≤ 13 meses, oposição fácil, informação ao usuário.
3. Eventos agregados; IP truncado; retenção ≤ 25 meses (agregados) / 13 meses (ID).
4. Consentimento granular, "rejeitar" com a mesma força visual de "aceitar", sem pré-marcação, 2º nível quando houver categorias.
5. Revogação/oposição tão fácil quanto o aceite (`popsmart.revogar()` + link).
6. E-mail só com finalidade declarada e checkbox específica não pré-marcada; pedir o mínimo.
7. DOI com log (IP, data/hora, texto) e e-mail de confirmação neutro.
8. Política de retenção por finalidade com expurgo automático no coletor.
9. Contexto infantil: sem e-mail, sem segmentação.
10. Dados da jornada não são vendidos nem cedidos (Portaria 7.638 art. 12 quando houver promoção).

**Acessibilidade**
11. `<dialog>` + `showModal()` ou APG completo (foco inicial, ciclo, Esc, retorno de foco).
12. `role="dialog"`, `aria-modal` só com fundo inerte, `aria-labelledby` com título visível.
13. Alvos ≥ 24×24 px; fechar ≥ 44×44 em mobile.
14. Foco nunca oculto (2.4.11); teste com Tab.
15. `prefers-reduced-motion` respeitado; nada pisca.
16. Sem timer que fecha/avança sozinho.
17. Contraste AA em todos os temas; teste com leitor de tela.

**SEO e performance**
18. Mobile: banner pequeno/inline; sem overlay de tela cheia na entrada.
19. Sem redirecionar para coletar dados/consentimento.
20. `position: fixed` + `transform/opacity`; CLS causado pelo widget = 0.
21. Carregar após `load`/idle com fallback; `defer`; núcleo ≤ 15 KB gz.
22. Handlers ≤ 50 ms; INP ≤ 200 ms p75 medido.
23. Shadow DOM com testes de foco entre root e documento.

**Dark patterns e promoção**
24. Sem confirmshaming, fechar escondido, reabertura a cada visita, falsa escassez ou contador falso.
25. Sem roleta aleatória por padrão; desconto determinístico; sorteio real só com autorização SPA/MF.

## 8. Lacunas a fechar

- Abrir o relatório da FTC e o post do Google de 2016; ler a íntegra da Portaria 7.638/2022 e a Nota SEI 11/2018; confirmar `closedby` e a data do EAA no EUR-Lex; acompanhar Digital Omnibus e atualização do guia da ANPD; perguntar ao CNIL/DPO se a exceção de medição ainda cobre A/B.
