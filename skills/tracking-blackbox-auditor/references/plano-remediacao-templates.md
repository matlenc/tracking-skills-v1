# Plano de Remediação — templates P0/P1/P2 + matrix ROI

> **Reference da skill `tracking-blackbox-auditor`** — templates canônicos pra §6 do output (Plano de Remediação) priorizado por severidade × esforço × impacto-quantificado × skill responsável downstream.

---

## 1. Filosofia de priorização

| Tier | Quando | Critério |
|---|---|---|
| **P0** | Semana-1 imediato | Blockers legais (LGPD) + Blockers técnicos críticos (dedup quebrada, UA deprecated, purchase incomplete) |
| **P1** | Sprint-1 (~30 dias) | Alto ROI técnico (EMQ <7, hash não-trafega, click IDs ausentes) — impacto quantificável >20% attribution |
| **P2** | Sprint-2+ (~60-90 dias) | Refinamento (schema versioning, Safari ITP mitigation, custom dimensions) — impacto incremental |
| Backlog | Quando capacity | Baixa severidade — refinamentos opcionais |

---

## 2. Template P0 — Blockers Legais + Técnicos Críticos

```markdown
### P0 — Semana-1 (Blockers Legais + Técnicos Críticos)

| # | Achado | Dim | Severidade | Artigo/Spec | Esforço (h) | Impacto | Skill responsável |
|---|--------|-----|-----------|-------------|-------------|---------|--------------------|
| P0.1 | [Achado canônico] | D# | BLOCKER LGPD | LGPD Art. Nº | X-Yh | Risco multa R$ 50M evitado + LGPD compliance | `tracking-engineer` OR `instrumentation-engineer` |
| P0.2 | [...] | [...] | [...] | [...] | [...] | [...] | [...] |
```

### 2.1 Casos canônicos P0

#### P0.A — Banner consent ausente (BLOCKER LGPD)

```markdown
| P0.A | Banner consent ausente | D8 | BLOCKER LGPD | LGPD Art. 7º + ANPD Guia Cookies 2023 | 8-16h | Risco multa R$ 50M evitado + LGPD compliance | `tracking-engineer` (config OneTrust/Cookiebot/CookieYes + GTM integration) |

**Procedimento:**
1. Selecionar vendor (OneTrust enterprise / Cookiebot mid-market / CookieYes pequeno-mid)
2. Configurar 4 categorias canônicas (estritamente necessários / funcionais / analytics / marketing)
3. Integrar com GTM via Consent Mode v2 setup (`gtag('consent', 'default', {denied})` ANTES das tags)
4. Validar empiricamente — navegar sem aceitar → tags não disparam; aceitar → `gtag('consent', 'update', {granted})` chamado
5. Documentar política de cookies em página separada linkada do banner
```

#### P0.B — Banner cosmético ⭐ (BLOCKER LGPD)

```markdown
| P0.B | Banner consent cosmético (presente mas update nunca chamado) | D8 | BLOCKER LGPD | LGPD Art. 8º + ANPD Guia Dark Patterns 2024 | 8-16h | Risco multa + correção dark pattern documentado ANPD 2024 | `tracking-engineer` (substituir banner native do page builder por vendor LGPD-compliant) |

**Procedimento:**
1. Substituir banner native do page builder (Klickpages/RD-Pages/etc.) por vendor LGPD-compliant
2. Configurar Consent Mode v2 default state `denied` ANTES das tags carregarem
3. Validar `gtag('consent', 'update', ...)` chamado pós-click no aceitar
4. Validar default consent state pré-aceite via Playwright
5. Tags só disparam pós-update
```

#### P0.C — Default consent state granted

```markdown
| P0.C | Default consent state = `granted` sem aceite | D8 | BLOCKER LGPD | LGPD Art. 7º | 2-4h | Risco multa + correção LGPD compliance | `tracking-engineer` |

**Procedimento:**
1. Editar GTM `Consent Initialization - All Pages` trigger
2. Configurar `gtag('consent', 'default', {ad_storage: 'denied', analytics_storage: 'denied', ad_user_data: 'denied', ad_personalization: 'denied', functionality_storage: 'denied', personalization_storage: 'denied', security_storage: 'granted', wait_for_update: 500})` ANTES de qualquer tag
3. Validar pre-aceite: cookies tracking não setados
4. Validar pós-aceite: `gtag('consent', 'update', {...granted})` dispara correto
```

#### P0.D — PII em claro

```markdown
| P0.D | PII em claro em payload Meta/Google | D4 | BLOCKER LGPD | LGPD Art. 5º + Art. 7º + Meta CAPI spec | 8-16h | Risco multa + risco Pixel suspension Meta | `instrumentation-engineer` + `tracking-engineer` |

**Procedimento:**
1. Identificar payload com PII plain — Meta `/tr` com `ud[em]=email@exemplo.com` (não-hashado) OR Google Ads conversion com phone plain
2. Configurar hash SHA-256 server-side (CAPI gateway Stape/Cloud Run/N8N) ANTES do POST Meta/Google
3. Normalização canônica na origem: lowercase email trimado + phone E.164 + remove_acentos
4. Test sintético: `test@example.com` → SHA-256 = `973dfe463ec85785f5f95af5ba3906eedb2d931c24e69824a89ea65dba4e813b`
5. Validar payload via Playwright — ud[em] tem 64 chars hex lowercase
```

#### P0.E — Universal Analytics ativo (BLOCKER técnico)

```markdown
| P0.E | Universal Analytics ainda ativo (deprecated jul/2023) | D1 | BLOCKER técnico | Google deprecation jul/2023 | 16-40h | Recovery 100% analytics data + GA4 migration completa | `tracking-engineer` + `tracking-ga4` |

**Procedimento:**
1. Auditar todas referências a `google-analytics.com/analytics.js` no GTM + código
2. Migrar tags UA → GA4 equivalentes (PageView, Event Custom)
3. Configurar GA4 property novo com Enhanced Measurement
4. Backfill historical data via BigQuery export (se UA tinha BQ export ativo)
5. Pausar tags UA + remover scripts inline
```

#### P0.F — event_id ausente com CAPI server-side ativo (BLOCKER técnico)

```markdown
| P0.F | event_id ausente em eventos com CAPI server-side ativo | D6 | BLOCKER técnico | Meta CAPI spec — dedup spec | 8-16h | Dedup 100% restaurada + EMQ +1-2 pontos | `instrumentation-engineer` + `tracking-meta-ads` |

**Procedimento:**
1. Setup UUID v4 generation client-side via crypto.randomUUID() OR uuid lib
2. Atribuir event_id ANTES do dataLayer.push + Pixel browser fbq + CAPI server payload
3. Garantir mesmo event_id browser + server (passa via Pixel `eventID:` + CAPI payload `event_id`)
4. Validar dedup rate via Meta Events Manager → Diagnostics
```

---

## 3. Template P1 — Alto ROI Técnico

```markdown
### P1 — Sprint-1 (Alto ROI Técnico, ~30 dias)

| # | Achado | Dim | Severidade | Esforço (h) | Impacto quantificado | Skill responsável |
|---|--------|-----|-----------|-------------|----------------------|--------------------|
| P1.1 | [Achado] | D# | Alta | X-Yh | [Impacto canônico] | [Skill] |
```

### 3.1 Casos canônicos P1

#### P1.A — Hash existe client-side mas não trafega ⭐

```markdown
| P1.A | Hash existe client-side mas não trafega (D4 sub-score intermediário Aquatro) | D4 | Alta | 4-8h | EMQ +2-4 pontos → +20-40% Meta attribution + Lead match recovery | `instrumentation-engineer` (correção fbq payload pra incluir `ud[*]` Advanced Matching) |

**Procedimento:**
1. Identificar payload `fbq('track', 'Lead', cd)` que NÃO inclui `ud[*]`
2. Adicionar ao payload `customData`: `{em: '<sha256_email>', ph: '<sha256_phone>', external_id: '<id>'}`
3. Confirmar normalização ANTES do hash (já feita server-side OR client-side)
4. Validar payload via Network tab — `/tr?...&ud[em]=973dfe463...` aparece
5. EMQ esperado: +2-4 pontos
```

#### P1.B — EMQ <5.0 ("Poor")

```markdown
| P1.B | EMQ <5.0 ("Poor" Meta) | D5 | Alta | 8-16h | EMQ +3-5 pontos → +40-60% Meta attribution + ad spend efficiency | `instrumentation-engineer` + `tracking-meta-ads` |

**Procedimento:**
1. Migrar Pixel browser → CAPI server-side (Stape gateway recommended pra setup ≤1h OR Cloud Run pra setup custom)
2. Hash SHA-256 server-side (email + phone + first_name + last_name + city + state + zipcode + country)
3. Adicionar client_ip_address + client_user_agent server-side
4. Persistir fbclid em cookie `_fbc` 90d 1st-party
5. external_id consistente cross-events (mesmo CRM ID em PageView + ViewContent + AddToCart + Lead/Purchase)
6. Validar EMQ via Events Manager — esperado 7.5-8.8 Lead / 8.8-9.3 Purchase
```

#### P1.C — Click IDs canônicos não capturados

```markdown
| P1.C | Click IDs canônicos (gclid OR fbclid) não capturados | D7 | Alta | 4-8h | Enhanced Conversions recovery 100% + offline conversions upload viável | `tracking-engineer` + `tracking-ga4` |

**Procedimento:**
1. Setup GTM Custom HTML pra capturar URL query params (gclid + fbclid + wbraid + gbraid + msclkid + ttclid + li_fat_id)
2. Persistir em cookie 1st-party com lifetime 90 dias
3. Push pra dataLayer em PageView + Lead event
4. Validar via Playwright test sintético com `?gclid=TEST123` — cookie `_gcl_aw` setado + included em CRM submission
```

#### P1.D — Eventos ecommerce críticos ausentes (`add_to_cart` / `begin_checkout` / `purchase`)

```markdown
| P1.D | Eventos ecommerce críticos ausentes | D9 | Alta (BLOCKER se `purchase`) | 8-16h | Funnel completo + remarketing audiences ativas + Smart Bidding optimization | `instrumentation-engineer` + `tracking-ga4` |

**Procedimento:**
1. Mapear gaps via Playwright jornada completa (browse → ATC → checkout)
2. Implementar dataLayer.push em cada evento canônico GA4 (11 eventos)
3. Items array completo (item_id + item_name + price + quantity + currency)
4. transaction_id único per-transaction (UUID OR order_id)
5. Meta `AddToCart` + `InitiateCheckout` + `Purchase` paralelos com event_id matching GA4 transaction_id
6. Google Ads `purchase` value-based bidding setup
```

#### P1.E — Política de privacidade incompleta (Art. 18 direitos)

```markdown
| P1.E | Política de privacidade sem listar 9 direitos LGPD Art. 18 | D8 | Alta | 4-8h | Risco crescente fiscalização ANPD 2026-2027 (Tema Prioritário 1 — direitos titular) | Operador + jurídico cliente |

**Procedimento:**
1. Drafting completo dos 9 direitos canônicos (acesso, correção, anonimização, portabilidade, eliminação, info compartilhamentos, info ausência consent, revogação, confirmação)
2. Incluir canal de exercício (email DPO OR formulário web)
3. Identificar controlador (CNPJ + endereço + DPO)
4. Listar compartilhamentos com Meta + Google + LinkedIn + outros 3rd parties
5. Revisão jurídica
6. Publicar em página linkada do banner e footer
```

#### P1.F — Duplicação Custom GTM + Enhanced Measurement ⭐

```markdown
| P1.F | Duplicação Custom GTM + Enhanced Measurement (refinamento Aquatro) | D2 | Alta | 1-2h | Métricas 50% mais limpas + custo extra hit eliminado | `tracking-engineer` |

**Procedimento:**
1. Identificar evento disparando 2× (típico: `form_start` Custom GTM + Enhanced Measurement automático)
2. Desativar Custom GTM **OU** desativar Enhanced Measurement pro evento específico
3. Manter um source único — preferência Custom GTM (controle granular sobre parâmetros)
4. Validar via Playwright — evento dispara 1× só
5. Documentar decisão no Measurement Plan
```

---

## 4. Template P2 — Refinamento

```markdown
### P2 — Sprint-2+ (Refinamento, ~60-90 dias)

| # | Achado | Dim | Severidade | Esforço (h) | Impacto | Skill responsável |
|---|--------|-----|-----------|-------------|---------|--------------------|
| P2.1 | [Achado] | D# | Média | X-Yh | [Impacto canônico] | [Skill] |
```

### 4.1 Casos canônicos P2

#### P2.A — Schema versioning Snowplow-style

```markdown
| P2.A | dataLayer sem `_schema_version` (Snowplow-style SemVer OR SchemaVer) | D2 | Média | 2-4h | Data quality estruturada + drift detection viável + refactor downstream seguro | `instrumentation-engineer` + `measurement-architect` |

**Procedimento:**
1. Adicionar campo `_schema_version: "1.0.0"` em todos dataLayer.push events
2. Documentar schema no Measurement Plan v1.0.0
3. Bump rules: minor pra additive changes; major pra breaking changes
4. Validation downstream (measurement-auditor Fase 3) consume schema_version pra drift detection
```

#### P2.B — Safari ITP mitigation via SGTM 1st-party

```markdown
| P2.B | Safari ITP sem mitigation (cliente Safari traffic ≥20%) | D7 | Média | 16-40h | Safari attribution recovery +70-85% + iOS conversion path preserved | `tracking-engineer` + `tracking-google-ads` + `tracking-meta-ads` |

**Procedimento:**
1. Setup SGTM 1st-party via custom domain CNAME (`events.cliente.com.br` → Stape OR Cloud Run)
2. Configurar cookies `_gcl_aw` + `_fbc` setados server-side via HTTP Set-Cookie 1st-party
3. Pre-cliente Safari traffic ≥20%, ROI esperado: +70-85% Safari attribution
4. Validar via Playwright Webkit test sintético — gclid persiste em cookie pós-Safari stripping
5. Confirmar via Meta Events Manager + Google Ads — attribution Safari volta a níveis pré-ITP
```

#### P2.C — Cobertura jornada multi-page (modo `site`)

```markdown
| P2.C | Cross-domain handling + content_group ausente em cliente multi-domain modo `site` | D3 | Média | 4-8h | Multi-domain attribution recovery + content categorization granular | `tracking-engineer` + `tracking-ga4` |

**Procedimento:**
1. Configurar GA4 cross-domain measurement (Allow Manual Domains setting + automatic linker)
2. Validar `_gl` parameter preservado em outbound links cross-domain
3. content_group dimension setup (mapeamento categoria conteúdo)
4. client_id persistente cross-page (cookie `_ga` mesmo across navigations)
5. Test E2E — user navega 3+ páginas + sai pra subdomínio + volta — mesma session GA4
```

#### P2.D — Custom dimensions GA4

```markdown
| P2.D | Custom dimensions GA4 ausentes (page_type / plan_tier / customer_segment / funnel_stage) | D2 | Média | 4-8h | Segmentação reports granular + análises por persona/produto | `tracking-engineer` + `tracking-ga4` |

**Procedimento:**
1. Mapear custom dimensions canônicas (page_type / plan_tier / customer_segment / funnel_stage) com `measurement-architect`
2. Setup custom dimensions GA4 Admin → Custom Definitions
3. Push pra dataLayer + GTM variable extraction
4. Aplicar em events relevantes (PageView, Conversion)
5. Validar via GA4 DebugView
```

---

## 5. Matrix ROI consolidada

| Achado | Esforço | Impacto típico | ROI |
|---|---|---|---|
| Banner cosmético | 8-16h | Risco multa + dark pattern fix | **ALTÍSSIMO** |
| PII em claro | 8-16h | Risco multa + Pixel suspension | **ALTÍSSIMO** |
| event_id ausente CAPI | 8-16h | Dedup 100% + EMQ +1-2 | **ALTO** |
| Hash não trafega | 4-8h | EMQ +2-4 → +20-40% attribution | **ALTO** |
| EMQ <5 | 8-16h | EMQ +3-5 → +40-60% attribution | **ALTO** |
| Click IDs não capturados | 4-8h | Enhanced Conversions 100% | **ALTO** |
| Eventos ecommerce críticos | 8-16h | Funnel completo + Smart Bidding | **ALTO** |
| Duplicação Custom+Enhanced | 1-2h | Métricas 50% mais limpas | **ALTO (effort baixo)** |
| Universal Analytics | 16-40h | Recovery 100% analytics | **ALTO mas esforço alto** |
| Política Art. 18 incompleta | 4-8h | Risco fiscalização ANPD 2026 | **ALTO** |
| Schema versioning | 2-4h | Data quality estruturada | **MÉDIO** |
| Safari ITP mitigation | 16-40h | Safari +70-85% (cliente ≥20% Safari) | **MÉDIO** se Safari traffic baixo / **ALTO** se ≥20% |
| Cross-domain handling | 4-8h | Multi-domain attribution | **MÉDIO** se cliente multi-domain |
| Custom dimensions GA4 | 4-8h | Segmentação granular | **MÉDIO** |

---

## 6. Skills downstream responsáveis

| Skill | Skills que aceita handoff de remediação |
|---|---|
| `instrumentation-engineer` v2 (Fase 2) | Implementação dataLayer.push + payloads CAPI + hash server-side |
| `tracking-engineer` (Fase 2 — orquestradora) | Config GTM + plataformas + Consent Mode v2 |
| `tracking-setup-gtm` (Fase 2) | GTM container setup + variables + triggers |
| `tracking-ga4` (Fase 2) | GA4 setup completo + Enhanced Measurement + custom dimensions |
| `tracking-google-ads` (Fase 2) | Google Ads conversion actions + Enhanced Conversions |
| `tracking-meta-ads` (Fase 2) | Meta Pixel + CAPI + Advanced Matching |
| `tracking-linkedin-ads` (Fase 2) | LinkedIn Insight Tag + Conversion API |
| `tracking-tiktok-ads` (Fase 2) | TikTok Pixel + Events API |
| `tracking-clarity` (Fase 2) | Clarity custom tags + segmentação |
| `measurement-architect` (Fundação 1.3.2) | Schema definitions + EMQ targets + hash policy |
| Operador + jurídico cliente | Política privacidade + DPO setup |

---

## 7. Template final do §6 do output

```markdown
## 6. Plano de Remediação

> Priorização canônica P0/P1/P2 baseada em severidade × ROI (esforço × impacto-quantificado).

### P0 — Semana-1 (Blockers Legais + Técnicos Críticos)

| # | Achado | Dim | Severidade | Artigo/Spec | Esforço | Impacto | Skill |
|---|--------|-----|-----------|-------------|---------|---------|-------|
| P0.1 | [Banner consent cosmético] | D8 | BLOCKER LGPD | LGPD Art. 8º + ANPD Dark Patterns 2024 | 8-16h | Risco multa evitado | `tracking-engineer` |
| P0.2 | [PII em claro] | D4 | BLOCKER LGPD | LGPD Art. 5º/7º + Meta CAPI spec | 8-16h | Risco multa + Pixel suspension evitado | `instrumentation-engineer` |
| P0.3 | [...] | [...] | [...] | [...] | [...] | [...] | [...] |

### P1 — Sprint-1 (Alto ROI Técnico, ~30 dias)

| # | Achado | Dim | Severidade | Esforço | Impacto quantificado | Skill |
|---|--------|-----|-----------|---------|----------------------|-------|
| P1.1 | [Hash não trafega] | D4 | Alta | 4-8h | EMQ +2-4 pontos → +20-40% Meta attribution | `instrumentation-engineer` |
| P1.2 | [EMQ <5.0] | D5 | Alta | 8-16h | EMQ +3-5 pontos → +40-60% attribution | `instrumentation-engineer` + `tracking-meta-ads` |
| P1.3 | [...] | [...] | [...] | [...] | [...] | [...] |

### P2 — Sprint-2+ (Refinamento, ~60-90 dias)

| # | Achado | Dim | Severidade | Esforço | Impacto | Skill |
|---|--------|-----|-----------|---------|---------|-------|
| P2.1 | [Safari ITP mitigation] | D7 | Média | 16-40h | Safari attribution +70-85% (cliente Safari traffic ≥20%) | `tracking-engineer` |
| P2.2 | [Schema versioning Snowplow] | D2 | Média | 2-4h | Data quality estruturada | `instrumentation-engineer` + `measurement-architect` |
| P2.3 | [...] | [...] | [...] | [...] | [...] | [...] |

### Total estimado

| Tier | Itens | Esforço total | ROI total |
|------|-------|---------------|-----------|
| P0 | X itens | Y-Zh | Risco multa LGPD + correção blockers técnicos |
| P1 | X itens | Y-Zh | EMQ +X pontos / attribution +Y% / ad spend efficiency +Z% |
| P2 | X itens | Y-Zh | Data quality estruturada + refinamentos incrementais |
| **Total** | **X itens** | **Y-Zh** | **Estado-da-arte alcançável em N sprints** |
```

---

## 8. References cross-skill

- [`severidades-canonicas.md`](severidades-canonicas.md) — mapping severidade → artigo legal / spec técnica
- [`rubrica-tracking-blackbox.md`](rubrica-tracking-blackbox.md) — 10 dimensões + sub-critérios
- [`lgpd-compliance-canonicas.md`](lgpd-compliance-canonicas.md) — LGPD artigos detalhados
