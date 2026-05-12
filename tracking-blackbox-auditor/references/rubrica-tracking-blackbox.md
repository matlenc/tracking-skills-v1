# Rubrica Tracking Black-Box — 10 dimensões canônicas estado-da-arte 2026

> **Reference CORE da skill `tracking-blackbox-auditor`** — rubrica universal canônica testada empiricamente em piloto Aquatro Suprimentos 2026-05-11.
> **Cutoff temporal:** estado-da-arte 2026. **Atualização anual obrigatória** (D-AUDITORES-6 — política `arquitetura/politicas/calibracao-rubricas-auditoras.md` pendência B7).

---

## 1. Síntese — pesos canônicos por modo

| # | Dimensão | LP B2B | LP B2C | Site | Ecommerce |
|---:|---|---:|---:|---:|---:|
| D1 | Stack instalada vs benchmark | 12% | 12% | 10% | 10% |
| D2 | DataLayer schema quality | 12% | 10% | 12% | 12% |
| D3 | Cobertura de jornada (funnel) | 8% | 10% | 18% | 8% |
| D4 | Identity & Hash Policy (LGPD-aware) | 12% | 10% | 8% | 12% |
| D5 | EMQ Meta estimado | 15% | 15% | 8% | 12% |
| D6 | Dedup CAPI prontidão | 12% | 10% | 10% | 12% |
| D7 | Click IDs capture & persistence | 12% | 12% | 12% | 10% |
| D8 | Consent Mode v2 + LGPD | 8% | 12% | 12% | 10% |
| D9 | Enhanced Ecommerce GA4 | — | — | — | 8% |
| D10 | Tracking health & performance | 9% | 9% | 10% | 6% |
| D11 | Pipeline pós-LP (cond. informativa) | opcional | opcional | opcional | opcional |

**Total = 100% por modo.** D11 é sub-seção informativa (não pondera score).

**Calibração modelo de negócio dentro de modo `lp`:**
- **LP B2B leadgen (default)** — D5 EMQ 15% + D4 Identity 12% + D7 Click IDs 12% (identidade do contato é o que vale)
- **LP B2C ticket alto** (clínicas premium, escolas privadas, plano saúde) — D8 Consent + LGPD 12% (regulação mais escrutinada CONAR/PROCON) + D3 jornada 10%
- **LP B2C ticket baixo** (consumer) — próximo de B2B default mas com D8 12%

---

## 2. Categorias canônicas da nota final

Nota ponderada 0-10 → Categoria canônica:

| Faixa | Categoria | Interpretação |
|---|---|---|
| 9.0-10 | **Estado-da-arte** | Setup maduro N3-N4; refinamentos finos restantes; pode ser benchmark referência |
| 7.0-8.9 | **Maduro** | Setup robusto N2-N3; ≤2 gaps técnicos relevantes; sem blockers legais |
| 5.0-6.9 | **Funcional** | Setup mínimo N1-N2; ≥3 gaps técnicos OR ≥1 blocker LGPD; recupera valor com remediação P0-P1 |
| 3.0-4.9 | **Subdesenvolvido** | Setup deficiente N0-N1; múltiplos blockers; remediation pesada necessária |
| 0-2.9 | **Inexistente** | Setup quase ausente N0; greenfield prático mesmo com algo instalado |

---

## 3. D1 — Stack instalada vs benchmark do modelo de negócio

### 3.1 Como capturar (procedure)

```python
# Playwright Python
page.goto(target_url, wait_until="networkidle")
html = page.content()
globals_ = page.evaluate("() => Object.keys(window).filter(k => /gtag|gtm|fbq|_fbq|ttq|uetq|clarity|_linkedin|hsq|RdIntegration/.test(k))")
network_tracking = [r.url for r in page_requests if any(domain in r.url for domain in TRACKING_DOMAINS)]
```

### 3.2 Plataformas canônicas detectadas

| Plataforma | Como detectar (regex/global/network) |
|---|---|
| GA4 | `<script src="*googletagmanager.com/gtag/js?id=G-*">` OR `window.gtag` |
| GTM | `<script src="*googletagmanager.com/gtm.js?id=GTM-*">` OR `window.google_tag_manager` |
| GTM Server-side | subdomínio próprio servindo `gtm.js` (não `googletagmanager.com`) — Stape signature OR Cloud Run `*.run.app` |
| Meta Pixel | `fbq` function + `<script src="*connect.facebook.net/*fbevents.js">` |
| Google Ads tag | `<script src="*googletagmanager.com/gtag/js?id=AW-*">` |
| LinkedIn Insight Tag | `_linkedin_partner_id` global OR network request a `linkedin.com/li.lms-analytics` |
| TikTok Pixel | `window.ttq` OR network request a `analytics.tiktok.com` |
| Microsoft Ads UET | `window.uetq` |
| Microsoft Clarity | `window.clarity` |
| Hotjar / FullStory / Mouseflow | globals respectivos (`window.hj`, `window.FS`, `window._mfq`) |
| RD Station tracking | `<script src="*d335luupugsy2.cloudfront.net/*">` OR `window.RdIntegration` |
| HubSpot tracking | `<script src="*js.hs-scripts.com*">` OR `window._hsq` |
| Pinterest Pixel | `window.pintrk` |
| Snapchat Pixel | `window.snaptr` |
| Twitter Pixel | `window.twq` |
| **Universal Analytics (DEPRECATED jul/2023)** | `<script src="*google-analytics.com/analytics.js">` → **BLOCKER técnico** |

### 3.3 Scoring D1 vs benchmark por modelo

| Modelo | Stack esperado canônico | Score 10 |
|---|---|---|
| **B2B leadgen** | GA4 + GTM + Meta + LinkedIn + (Google Ads se paid) + GTM Server-side | Stack completo + SGTM |
| **B2C ticket alto** | GA4 + GTM + Meta + Google Ads + Clarity + (CRM RD/HS bonus) | Stack completo + Clarity heatmaps |
| **Ecommerce** | GA4 + GTM + Meta + Google Ads + (TikTok/Pinterest cond. vertical) | Stack completo + Enhanced Ecommerce GA4 |
| **PLG SaaS** | Product analytics (Amplitude/Mixpanel/PostHog) + Meta + Google Ads | Product analytics maduro + paid stack |

Score 5 com setup parcial (só GA4 + Meta sem GTM). Score 0 sem nada relevante detectado. Universal Analytics ativo = penalidade -2 + BLOCKER técnico.

---

## 4. D2 — DataLayer schema quality

### 4.1 Como capturar

Monkey-patch dataLayer.push interceptando todos pushes durante jornada:

```javascript
page.evaluate(`() => {
  window._auditPushes = [];
  const orig = window.dataLayer.push.bind(window.dataLayer);
  window.dataLayer.push = (...args) => {
    window._auditPushes.push({
      time: Date.now(),
      args: JSON.parse(JSON.stringify(args))
    });
    return orig(...args);
  };
}`);
```

### 4.2 Sub-critérios

| Sub-critério | Threshold |
|---|---|
| dataLayer existe (Boolean) | Score 0 se ausente |
| Schema versioning Snowplow-style (`schema_version` field SemVer OR SchemaVer MODEL-REVISION-ADDITION) | +2 se presente |
| Naming convention canônica (PascalCase events + snake_case params consistente) | +2 se consistente |
| Cobertura eventos canônicos vs 13 eventos core taxonomy (`measurement-architect` reference: PageView/CTAInteract/VideoProgress/FormStart/Lead/MQL/NOICP/SAL/SQL/ProposalSent/Negotiation/DealWon/DealLost) | +2 por cobertura ≥80% |
| Parâmetros enriquecidos (`user_id`/`external_id`/`content_group`/`traffic_source`/`form_name`/`cta_position`/`value`/`currency`/`items`) | +2 se ≥5 parâmetros enriquecidos por evento |
| Dimensões custom presentes (`page_type` / `plan_tier` / `customer_segment` / `funnel_stage`) | +1 se presente |
| **Duplicação Custom GTM + Enhanced Measurement** ⭐ (refinamento Aquatro) | -2 se detectado (`form_start` 2× = anti-pattern) |
| Race conditions detectadas (eventos antes do gtm.js carregar) | -2 se detectado |

### 4.3 Refinamento piloto Aquatro

Em Aquatro detectou-se `form_start` disparando 2×:
- 1× custom GTM (`form_id=aq_lead_popup` + `form_name=Lead Popup Aquatro` + `form_type=builder`)
- 1× Enhanced Measurement automático GA4 (com `form_length=22` + `first_field_id=input_1777304893`)

Métrica inflada 2× em reports + custo extra de hit. Sub-critério **"Duplicação Custom + Enhanced Measurement"** detecta esse anti-pattern. Severidade ALTA (não BLOCKER).

---

## 5. D3 — Cobertura de jornada (funnel coverage)

### 5.1 Como capturar

Playwright simula user real — clicks em CTAs + scroll + form fill + outbound. Detalhe procedural em `procedures-playwright-tracking.md §2-§3`.

### 5.2 Sub-critérios

| Sub-critério | Procedure |
|---|---|
| PageView automático todas páginas | Navegar 3-5 páginas + verificar `page_view` GA4 em cada |
| CTA tracking (hero + body + footer) | Click cada CTA principal + verificar event com `cta_position`/`cta_label` |
| Form start | Focus em primeiro campo do form → `form_start` event |
| Form submit (Lead) | Preencher form com dados sintéticos + submit → `generate_lead` GA4 + `Lead` Meta capturados |
| Form field errors | Submit com erro → `form_error` event |
| Scroll milestones 25/50/75/100% | Scroll programático + verificar events |
| Video tracking | YouTube embed / Vimeo / native player detectado → play + verificar VideoStart/Progress/Complete |
| Outbound link tracking | Click em link externo → `outbound_click` event |
| File download tracking | Click em PDF/doc → `file_download` (GA4 Enhanced Measurement) |
| Search interno | Busca interna → `search` event com `search_term` |
| Phone click (`tel:`) | Click em `<a href="tel:...">` → event capturado |
| WhatsApp click (`wa.me/`) | Relevante BR — click capturado? |

### 5.3 Crítico modo `site`

- Cross-domain handling (linker `_gl` parameter preservado em outbound interno)
- client_id persistente cross-page (cookie `_ga` mesmo across navigations)
- content_group dimension (categoria do conteúdo presente)
- search interno do site funcional pra reports

---

## 6. D4 — Identity & Hash Policy (LGPD-aware)

### 6.1 Como capturar

Intercepta payload Lead/Purchase pra Meta `/tr` via `page.on('request')` + inspeciona parâmetros `ud[em]`, `ud[ph]`, etc.

### 6.2 Sub-critérios

| Sub-critério | Procedure |
|---|---|
| Cobertura campos AM Meta | Contagem de `em`/`ph`/`fn`/`ln`/`ct`/`st`/`zp`/`country`/`ge`/`db`/`external_id`/`client_ip_address`/`client_user_agent` no payload |
| Hash SHA-256 confirmado | Cada campo PII tem exatamente 64 chars hex lowercase |
| Normalização ANTES do hash | Test: passa `test@example.com` → SHA-256 esperado = `973dfe463ec85785f5f95af5ba3906eedb2d931c24e69824a89ea65dba4e813b`. Se bate = normalização correta; se não = sem trim/lowercase |
| Identity resolution cross-platform | `user_id` GA4 + `external_id` Meta + `client_id` (cookie `_ga`) consistentes |
| Same identifier reuse | Mesmo `user_id` em GA4, Meta, LinkedIn |
| **Hash existe mas não trafega** ⭐ (refinamento Aquatro) | Sub-score intermediário 3-5/10 — cookie tem `ph` SHA-256 perfeito mas `fbq('track', 'Lead', cd)` NÃO inclui `ud[*]`. Situação extremamente comum BR (Klickpages, RD Marketing, page builders proprietários) |
| **PII em claro** | **BLOCKER LGPD Art. 5º/7º** — algum payload network tem email/phone/CPF não-hashado? |

### 6.3 Scoring D4

- Score 10: hash trafega corretamente + ≥6 fields AM + normalização correta + identity resolution cross-platform
- Score 7-9: hash trafega + ≥4 fields AM + normalização correta
- **Score 3-5: hash existe client-side mas não trafega** (caso Aquatro)
- Score 1-2: hash ausente OR PII em claro detectado → **BLOCKER LGPD**

---

## 7. D5 — EMQ Meta estimado

### 7.1 Fórmula canônica Meta 2024-2026

Detalhe em `emq-formula-meta.md`. Resumo:

```
EMQ_score = base(2.0)
          + email_match: 0.0-2.0
          + phone_match: 0.0-2.0
          + name_match (fn+ln): 0.0-1.5
          + address_match (ct+st+zp+country): 0.0-1.0
          + external_id: 0.0-1.5
          + client_ip_address: 0.0-0.5
          + client_user_agent: 0.0-0.5
          + fbc (fbclid persistido): 0.0-1.0
          + fbp (Meta first-party cookie): 0.0-0.5
```

### 7.2 Targets canônicos Meta

| Faixa EMQ | Categoria | Action |
|---|---|---|
| ≥7.0 | **Good** ideal | Manter |
| 5.0-6.9 | Fair | Melhorar |
| <5.0 | **Poor** | Crítico — remediation P1 |

**Benchmarks por evento (2026):**
- Purchase: 8.8-9.3
- AddToCart: 8.0+
- Lead: 7.0+
- PageView: 6.5-7.5 (normal mais baixo — menos dados disponíveis)

### 7.3 Confidence

- **Confidence `media`** Nível 1 (estimado client-side via fórmula)
- **Confidence `alta`** Nível 4 (Meta Business Manager confirma EMQ real — margem ±1 ponto vs estimativa)

---

## 8. D6 — Dedup CAPI prontidão

### 8.1 Sub-critérios

| Sub-critério | Procedure |
|---|---|
| `event_id` presente em eventos críticos | Lead/Purchase/CompleteRegistration têm `event_id` no payload `/tr` |
| Formato UUID v4 canônico | `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx` regex |
| Server-side endpoint detectado | Network request duplicada pra `facebook.com/tr` (browser) + subdomínio próprio (Stape signature `events.cliente.com.br`, Cloud Run `*.run.app`, N8N webhook `/webhook/...`) |
| CAPI gateway identificável | Stape pattern, Cloud Run, N8N webhook, custom |
| Browser-server event matching inferível | `event_id` igual entre browser e server-side request |

### 8.2 Heurísticas inferência server-side sem Nível 4 ⭐ (refinamento Aquatro)

Sinaliza CAPI server-side mesmo sem credenciais Meta Business:

| Pattern observável | Inferência |
|---|---|
| Subdomínios suspeitos: `events.*` / `tracking.*` / `tagm.*` / `gtm-server.*` | Stape signature |
| Cloud Run subdomain (`*.run.app`) | Cloud Run hosting próprio |
| Requests pra hosts não-`googletagmanager`/`facebook.com` em payload tracking | CAPI gateway próprio |
| Padrões N8N webhook em network (`/webhook/...`) | N8N orquestração |
| CNAME cloaking patterns | 1st-party cookie strategy + ITP bypass |

### 8.3 BLOCKER técnico D6

- `event_id` ausente com CAPI server-side ativo = **BLOCKER técnico** (Meta CAPI spec — dedup impossível, events duplicados ou subreportados)

---

## 9. D7 — Click IDs capture & persistence

### 9.1 Procedure

Playwright navega URL com query param sintético:
```
?gclid=AUDIT_TEST_GCLID_123&fbclid=AUDIT_TEST_FBCLID_456&wbraid=AUDIT_WBRAID&gbraid=AUDIT_GBRAID&msclkid=AUDIT_MSCLKID&ttclid=AUDIT_TTCLID&li_fat_id=AUDIT_LI_FAT_ID
```

Após navegação inspeciona cookies/localStorage/sessionStorage. Navega `/thank-you` ou form submit → verifica se ID é incluído em payloads outbound.

### 9.2 Click IDs canônicos auditados

| Click ID | Plataforma | Crítico pra |
|---|---|---|
| `gclid` | Google Ads | Conversion attribution |
| `fbclid` | Meta | Attribution (deprecated-ish pós-iOS 14 mas valioso) |
| `wbraid` | Google Ads iOS app-to-web | iOS ATT post-IDFA |
| `gbraid` | Google Ads iOS web | iOS ATT post-IDFA |
| `msclkid` | Microsoft Ads | Attribution |
| `ttclid` | TikTok | Attribution |
| `li_fat_id` | LinkedIn | Attribution |
| `dclid` | Display & Video 360 | Display attribution |
| `epik` | Pinterest | Attribution |

### 9.3 Sub-critérios

- Captura via URL parsing → cookie/localStorage
- Persistência ≥90 dias
- First-touch preserved
- Incluído em CRM submission (`gclid` em RD Station custom field, etc.)
- `_gcl_aw` cookie GA4 detectado
- `_fbp` + `_fbc` cookies Meta detectados
- **Safari ITP / Link Tracking Protection 2024-2026 resilience** — gclid/fbclid stripped das URLs em Safari 17+. Mitigation: SGTM 1st-party (cookies setados via HTTP server-side bypassam ITP). Sub-score se cliente tem Safari traffic ≥20%

---

## 10. D8 — Consent Mode v2 + LGPD compliance

### 10.1 Sub-critérios canônicos 2026

| Sub-critério | Critério |
|---|---|
| Banner de consent presente | OneTrust / Cookiebot / Iubenda / Termly / CookieYes / custom OR ausente (**BLOCKER LGPD**) |
| Default consent state | `gtag('consent', 'default', {...})` antes de qualquer tag, estado = `denied` (LGPD compliant) OR `granted` (**BLOCKER LGPD Art. 7º**) |
| Consent Mode v2 fields | `ad_user_data` + `ad_personalization` presentes (obrigatórios v2 desde mar/2024) |
| Tags pós-aceite only | **Validação empírica:** navegar sem aceitar → tags não disparam. Aceitar → `gtag('consent', 'update', {...granted})` + tags disparam |
| Granularidade banner | Categorização: estritamente necessários / funcionais / analytics / marketing |
| IP anonymization GA4 | `anonymize_ip: true` OR `_anonymizeIp` config |
| Cookies pré-aceite categorizados | Quantos cookies setados ANTES do aceite? Esperado: só estritamente necessários |
| Política de privacidade linkada | Footer com link `/privacidade` + direitos LGPD Art. 18 explícitos |
| Botão "Rejeitar" igual destaque "Aceitar" | LGPD/CONAR/ANPD dark pattern avoidance |
| **Banner cosmético** ⭐ (refinamento Aquatro) | Banner presente mas clicar "aceito" NÃO dispara `consent update` — separar de "banner ausente". **BLOCKER LGPD Art. 8º + ANPD Guia Dark Patterns 2024** |

### 10.2 Mudança upcoming junho/2026

Google Consent Mode v2 vai mudar em **15 jun/2026**: `ad_storage` controla GA4 separado de Ads. `ad_personalization` em Consent Mode v2 vira parâmetro governante pra Analytics→Ads personalization. Cliente sem `ad_personalization: granted` perde linkagem GA4→Ads pra personalização. **Sub-critério forward-looking:** auditoria 2026 sinaliza se cliente está preparado pra a mudança.

---

## 11. D9 — Enhanced Ecommerce GA4 (modo `ecommerce` exclusivo)

Detalhe em `enhanced-ecommerce-ga4-canonicas.md`. Resumo:

### 11.1 11 eventos canônicos GA4 verificados

| Evento | Quando dispara | Fields required |
|---|---|---|
| `view_item_list` | Listing/categoria | `items` array + `item_list_id` + `item_list_name` |
| `select_item` | Click em item do listing | `items` + `item_list_name` |
| `view_item` | PDP carregado | `items` + `currency` + `value` |
| `add_to_cart` | Click add to cart | `items` + `currency` + `value` |
| `remove_from_cart` | Click remove | `items` + `currency` + `value` |
| `view_cart` | Cart aberto | `items` + `currency` + `value` |
| `begin_checkout` | Entrada checkout | `items` + `currency` + `value` |
| `add_shipping_info` | Etapa shipping | `items` + `shipping_tier` |
| `add_payment_info` | Etapa payment | `items` + `payment_type` |
| `purchase` | Confirmation page | `transaction_id` único + `items` + `currency` + `value` + `tax` + `shipping` |
| `refund` | Refund processado | `transaction_id` + `items` cond. |

### 11.2 Sub-critérios

- Currency consistente cross-events (sempre BRL pra BR)
- `transaction_id` único (não-duplicado em refresh)
- Items array schema (item_id+item_name+price+quantity required)
- value = soma items (consistência matemática)
- Meta `Purchase` paralelo
- Google Ads `purchase` value-based bidding setup

---

## 12. D10 — Tracking health & performance

### 12.1 Procedure LCP canônico (refinamento Aquatro)

**NÃO usar** `performance.getEntriesByType('largest-contentful-paint')` direto — retorna `[]` se observer não registrado antes do load.

**Procedure canônico:**

```javascript
page.evaluate(`() => new Promise(resolve => {
  let lcp = null;
  new PerformanceObserver(list => {
    const entries = list.getEntries();
    lcp = entries[entries.length - 1].startTime;
  }).observe({ type: 'largest-contentful-paint', buffered: true });
  setTimeout(() => resolve(lcp), 3000);
})`);
```

### 12.2 Targets canônicos Core Web Vitals 2026

| Métrica | Good | Needs Improvement | Poor |
|---|---|---|---|
| LCP | <2.5s | 2.5-4.0s | >4.0s |
| INP | <200ms | 200-500ms | >500ms |
| CLS | <0.1 | 0.1-0.25 | >0.25 |

**Nota INP:** INP é métrica de campo (real user) — não medível em Lighthouse lab. Total Blocking Time (TBT) correlaciona; melhorar TBT melhora INP.

### 12.3 Sub-critérios D10

- Lighthouse CWV (LCP/INP/CLS dentro de Good)
- Console errors detectados (≥0 = -1; ≥5 = -3)
- Network failures 4xx/5xx pra endpoints tracking (1 falha tracking = -2; multiple = blocker técnico)
- Tags duplicadas (mesmo evento 2× no mesmo trigger = -2)
- Race conditions (eventos antes do gtm.js carregar = -2)
- Cross-browser parity (Safari ITP impacto vs Chrome — gap >30% sinaliza problema)
- Mobile viewport rendering correto
- Adblocker resilience (uBlock Origin block test cond.)

---

## 13. D11 — Pipeline pós-LP (opcional informativa)

Não pondera score — sub-seção informativa pra contextualizar pipeline downstream pra `measurement-architect`.

### 13.1 Padrões BR canônicos detectáveis

| Pattern URL destino | Pipeline inferido |
|---|---|
| `api.whatsapp.com/send/?phone=...&text=...` | Frictionless WhatsApp-first (sem CRM intermediário OU CRM via webhook server-side) |
| `wa.me/...` | Direct WhatsApp link (sem captura) |
| `rdstation.com/conversions/` ou `*rdstation*` | RD Station CRM |
| `forms.hubspot.com` ou `*hsforms*` | HubSpot |
| `pipefy.com` | Pipefy |
| `*.kommo.com` | Kommo CRM |
| Redirect próprio domínio `/obrigado` ou `/thank-you` | CRM próprio OR N8N webhook OR custom |

---

## 14. Severidades canônicas — hard blockers

Independente da nota ponderada, alguns achados forçam severidade crítica. Detalhe em `severidades-canonicas.md`.

| Achado | Severidade | Artigo/Spec |
|---|---|---|
| PII em claro em payload (D4) | **BLOCKER LGPD** | LGPD Art. 5º/7º |
| Universal Analytics ainda ativo (D1) | **BLOCKER técnico** | Google deprecation jul/2023 |
| Default consent state = `granted` sem aceite (D8) | **BLOCKER LGPD** | LGPD Art. 7º |
| Tags disparando antes do banner aceite (D8) | **BLOCKER LGPD** | LGPD Art. 8º |
| Banner consent ausente (D8) | **BLOCKER LGPD** | LGPD Art. 7º + ANPD Guia Cookies 2023 |
| **Banner consent cosmético** (D8) ⭐ | **BLOCKER LGPD** | LGPD Art. 8º + ANPD Guia Dark Patterns 2024 |
| Política de privacidade ausente (D8) | **BLOCKER LGPD** | LGPD Art. 9º + Art. 18º |
| `event_id` ausente com CAPI server-side ativo (D6) | **BLOCKER técnico** | Meta CAPI spec |

---

## 15. Confidence indicator por dimensão

Detalhe em `confidence-indicator-rubrica.md`. Cada dimensão recebe confidence indicator baseado em cobertura de evidência:

| Confidence | Quando |
|---|---|
| `alta` | Validável 100% client-side (D1 detecção plataformas, D7 click ID capture, D8 consent comportamento, D10 CWV) |
| `media` | Estimada client-side, confirmação requer API/server access (D5 EMQ estimado vs real, D6 dedup server-side, D9 purchase value confirmação) |
| `baixa` | Cobertura limitada client-side (D2 schema quality — vê o que disparou não config; D3 cobertura — só jornada testada) |

Cada flag de acesso adicional (Níveis 2-9) sobe confidence das dimensões correspondentes pra `alta`.

---

## 16. Scoring rubric — síntese do procedimento

1. **Aplica procedures Playwright** (referencia `procedures-playwright-tracking.md`) — captura D1 → D10
2. **Calcula score 0-10 por dimensão** com sub-critérios + thresholds acima
3. **Multiplica por peso do modo** (B2B leadgen / B2C ticket alto / B2C ticket baixo / site / ecommerce)
4. **Soma ponderada** → Nota final 0-10
5. **Categoria canônica** (estado-da-arte / maduro / funcional / subdesenvolvido / inexistente)
6. **Aplica severidades canônicas** independente da nota — hard blockers forçam destaque
7. **Atribui confidence indicator** (alta/media/baixa) por dimensão
8. **Constrói plano remediação** P0/P1/P2 (referencia `plano-remediacao-templates.md`)
9. **Mapeia handoffs canônicos** (skills downstream relevantes)
