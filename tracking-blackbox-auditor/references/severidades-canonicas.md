# Severidades Canônicas — hard blockers + alta/média/baixa + matrix ROI

> **Reference da skill `tracking-blackbox-auditor`** — taxonomia de severidades + mapping artigo legal / spec técnica + matrix ROI (esforço × impacto) por achado canônico.

---

## 1. Taxonomia de severidades

| Severidade | Definição | Mapping |
|---|---|---|
| **BLOCKER LGPD** | Violação direta de Art. LGPD com risco real de sanção ANPD | LGPD Art. + ANPD spec |
| **BLOCKER técnico** | Violação spec técnica que quebra functionality core | Meta CAPI / Google deprecation / Consent Mode v2 |
| **Alta severidade** | Gap significativo com impacto operacional alto mas sem violação legal direta | — |
| **Média severidade** | Gap técnico com impacto operacional moderado | — |
| **Baixa severidade** | Refinamento técnico, optimization opportunity | — |
| **Informativa** | Observação sem impacto (D11 pipeline pós-LP) | — |

---

## 2. Hard blockers LGPD

| # | Achado | Dimensão | LGPD Art. | ANPD Spec |
|---:|---|---|---|---|
| 1 | PII em claro detectado em payload Meta/Google | D4 | Art. 5º + Art. 7º | — |
| 2 | Banner consent ausente em LP que coleta dados | D8 | Art. 7º | Guia Cookies 2023 |
| 3 | Default consent state = `granted` sem aceite | D8 | Art. 7º + Art. 8º | Guia Cookies 2023 |
| 4 | Tags disparam antes do banner aceite | D8 | Art. 8º | Guia Cookies 2023 |
| 5 | **Banner cosmético** (presente mas update nunca chamado) ⭐ | D8 | Art. 8º | Guia Dark Patterns 2024 |
| 6 | Política de privacidade ausente | D8 | Art. 9º + Art. 18º | — |
| 7 | Cookies pré-aceite incluem analytics/marketing | D8 | Art. 7º | Guia Cookies 2023 |

⭐ Refinamento Aquatro 2026-05-11

### 2.1 Implicações de cada hard blocker LGPD

| Achado | Risco real |
|---|---|
| #1 PII em claro | Multa direta + reputational + perda de Pixel pode acontecer |
| #2 Banner ausente | Multa + remediation obrigatória ANPD |
| #3 Default granted | Multa + tem aplicação enforced em fiscalização 2026-2027 (Tema Prioritário 1 — direitos titular) |
| #4 Tags pré-aceite | Multa por consent inválido + reclamação titular |
| #5 Banner cosmético | Multa por dark pattern (Guia ANPD 2024 explícito) — **risco crescente** com fiscalização 2026 |
| #6 Política ausente | Multa por opacidade + obstrução de direitos Art. 18 |
| #7 Cookies pré-aceite tracking | Multa por coleta sem base legal |

---

## 3. Hard blockers técnicos

| # | Achado | Dimensão | Spec violada |
|---:|---|---|---|
| 1 | Universal Analytics ainda ativo | D1 | Google deprecation jul/2023 |
| 2 | `event_id` ausente com CAPI server-side ativo | D6 | Meta CAPI spec — dedup spec |
| 3 | `purchase` sem `transaction_id` em ecommerce | D9 | Google GA4 Enhanced Ecommerce spec |
| 4 | Items array vazio em `purchase` ecommerce | D9 | Google GA4 Enhanced Ecommerce spec |
| 5 | `item_id` ausente em items array | D9 | Google GA4 + Merchant Center spec |

---

## 4. Alta severidade (gaps significativos)

| Achado | Dimensão | Por quê alta |
|---|---|---|
| Hash existe client-side mas NÃO trafega (refinamento Aquatro) | D4 | EMQ baixo — perda 40-60% attribution Meta |
| EMQ <5.0 ("Poor" Meta) | D5 | Atribuição comprometida + custo aquisição inflado |
| Click IDs canônicos não capturados (gclid OR fbclid) | D7 | Quebra de Enhanced Conversions + offline conversions |
| Política de privacidade sem listar 9 direitos Art. 18 | D8 | Risco crescente fiscalização Tema Prioritário 1 ANPD 2026-2027 |
| Botão "Rejeitar" oculto/micro vs "Aceitar" destacado | D8 | Dark pattern documentado ANPD 2024 |
| Compartilhamentos com Meta/Google não declarados na Política | D8 | Art. 9º (transparência) — opacidade |
| Canal de exercício de direitos ausente (sem DPO/formulário) | D8 | Art. 18 — risco crescente Tema Prioritário 1 |
| **Duplicação Custom GTM + Enhanced Measurement** ⭐ | D2 | Métricas infladas 2× + custo extra de hit |
| LCP >2.5s + INP >200ms em mobile | D10 | UX degradada + bid signal Google Ads quality score |
| Tags duplicadas (mesmo evento 2× no mesmo trigger) | D10 | Conversion inflada + custo extra de hit |
| `add_to_cart` ausente em ecommerce | D9 | Funnel incompleto — remarketing audiences vazias |
| `begin_checkout` ausente em ecommerce | D9 | Checkout abandonment audience vazia |
| Meta `Purchase` paralelo ausente em ecommerce | D9 | Meta Ads otimização sub-ótima |
| Currency inconsistente cross-events ecommerce | D9 | Reports zonzos |
| Tags pre-launch (race conditions) | D10 | Eventos perdidos em primeiros 1-2s page load |

⭐ Refinamento Aquatro

---

## 5. Média severidade

| Achado | Dimensão |
|---|---|
| Stack parcial — falta GTM Server-side em B2B leadgen | D1 |
| Schema versioning ausente (dataLayer sem `_schema_version`) | D2 |
| Cobertura jornada cross-domain handling ausente em modo `site` | D3 |
| `external_id` não consistente cross-events Meta | D4 |
| EMQ 5.0-6.9 (Fair Meta) | D5 |
| `wbraid` / `gbraid` não capturado (iOS Google Ads attribution) | D7 |
| Safari ITP sem mitigation em cliente com Safari traffic 20-30% | D7 |
| Granularidade banner ausente (sem categorias visíveis) | D8 |
| IP anonymization GA4 desativado | D8 |
| Cross-browser parity gap >30% Safari vs Chrome | D10 |
| Adblocker resilience ausente | D10 |
| `refund` event ausente em ecommerce | D9 |
| `add_shipping_info` / `add_payment_info` ausentes em ecommerce | D9 |
| Google Ads value-based bidding setup ausente | D9 |

---

## 6. Baixa severidade (refinamentos)

| Achado | Dimensão |
|---|---|
| `_gcl_au` cross-domain linker ausente em cliente multi-domain | D7 |
| TikTok/Pinterest pixel ausente em vertical lifestyle/beleza | D1 |
| `dclid` capture ausente (DV360 attribution) | D7 |
| Custom dimensions GA4 ausentes (`page_type`, `plan_tier`) | D2 |
| Console errors não-críticos (deprecation warnings) | D10 |
| Schema custom dimension naming inconsistente | D2 |
| Form field errors event ausente | D3 |
| Search interno event ausente | D3 |
| Video tracking (YouTube embed) ausente | D3 |
| File download tracking ausente | D3 |

---

## 7. Informativas (não-pondera score)

| Observação | Dimensão | Função |
|---|---|---|
| Pipeline pós-LP detectado: WhatsApp frictionless | D11 | Informa contexto pipeline downstream |
| Pipeline pós-LP detectado: RD Station CRM | D11 | Informa CRM identificável |
| Pipeline pós-LP detectado: HubSpot | D11 | Informa CRM identificável |
| Pipeline pós-LP detectado: N8N webhook custom | D11 | Informa orquestração custom |
| Página de obrigado própria domain (CRM próprio OR N8N) | D11 | Informa pipeline opaco |

---

## 8. Matrix ROI — esforço × impacto

Mapping canônico achado → esforço-horas estimado × impacto quantificável:

### 8.1 BLOCKER LGPD (sempre P0)

| Achado | Esforço | Impacto | ROI |
|---|---|---|---|
| Banner ausente | 8-16h (vendor setup + integração GTM) | Risco multa R$ 50M evitado | **ALTÍSSIMO** |
| Default state granted | 2-4h (correção `gtag('consent', 'default', {denied})`) | Risco multa evitado | **ALTÍSSIMO** |
| Tags pre-accept | 4-8h (Consent Mode v2 setup correto) | Risco multa evitado | **ALTÍSSIMO** |
| Banner cosmético | 8-16h (substituir page builder native ou trocar vendor) | Risco multa evitado | **ALTÍSSIMO** |
| Política ausente | 4-8h (drafting + revisão jurídica) | Risco multa evitado | **ALTÍSSIMO** |
| PII em claro | 8-16h (server-side hashing setup) | Risco multa + Pixel suspension | **ALTÍSSIMO** |

### 8.2 BLOCKER técnico (P0-P1)

| Achado | Esforço | Impacto |
|---|---|---|
| Universal Analytics ativo | 16-40h (migração completa pra GA4) | Recovery 100% analytics data |
| event_id ausente CAPI | 8-16h (setup UUID v4 client+server) | Dedup 100% + EMQ +1-2 pontos |
| purchase sem transaction_id | 2-4h (correção checkout layer) | Revenue accuracy 100% |

### 8.3 Alta severidade (P1 — sprint-1)

| Achado | Esforço | Impacto |
|---|---|---|
| Hash não trafega | 4-8h (correção ud[*] payload) | EMQ +2-4 pontos → +20-40% Meta attribution |
| EMQ <5 ("Poor") | 8-16h (CAPI server-side migration) | EMQ +3-5 pontos → +40-60% attribution |
| Click IDs não capturados | 4-8h (GTM setup gclid/fbclid capture) | Enhanced Conversions recovery 100% |
| Duplicação Custom + Enhanced Measurement | 1-2h (desativar duplicação no GTM) | Métricas 50% mais limpas |

### 8.4 Média severidade (P2 — sprint-2+)

| Achado | Esforço | Impacto |
|---|---|---|
| Schema versioning ausente | 2-4h (refator dataLayer) | Data quality estruturada |
| Cross-domain linker ausente | 1-2h (config GA4 linker) | Multi-domain attribution recovery |
| Safari ITP mitigation | 16-40h (SGTM 1st-party setup) | Safari attribution recovery 70-85% |

### 8.5 Baixa severidade (P2-P3 — backlog)

Refinamentos opcionais — esforço-impacto baixo. Aplicar quando capacity disponível.

---

## 9. Priorização canônica P0/P1/P2

### P0 — Blockers legais + técnicos críticos (semana-1)
- Todos hard blockers LGPD (#1-#7)
- Universal Analytics ativo (BLOCKER técnico #1)
- event_id ausente com CAPI ativo (BLOCKER técnico #2)
- purchase sem transaction_id em ecommerce (BLOCKER técnico #3-#5)

### P1 — Alto ROI técnico (sprint-1)
- Hash não trafega (D4 sub-score intermediário)
- EMQ <7.0 — migração CAPI server-side
- Click IDs canônicos não capturados (gclid + fbclid mínimo)
- Eventos ecommerce críticos ausentes (`add_to_cart`, `begin_checkout`, `purchase`)
- Política de privacidade incompleta (Art. 18 direitos)
- Duplicação Custom + Enhanced Measurement

### P2 — Refinamento (sprint-2+)
- Schema versioning Snowplow-style
- Cross-domain linker em multi-domain
- Safari ITP mitigation (cliente com Safari traffic ≥20%)
- Cobertura jornada multi-page modo `site`
- Custom dimensions GA4
- Adblocker resilience
- Cross-browser parity

### Backlog
- TikTok/Pinterest pixel cond. vertical
- dclid capture (DV360)
- Form field errors / search interno events
- Video tracking detalhado

---

## 10. Format canônico do plano de remediação no output

```markdown
## 6. Plano de Remediação

### P0 — Semana-1 (Blockers Legais + Técnicos Críticos)

| # | Achado | Dimensão | Severidade | Artigo/Spec | Esforço (h) | Impacto | Skill responsável |
|---|--------|----------|-----------|-------------|-------------|---------|--------------------|
| P0.1 | Banner consent cosmético | D8 | BLOCKER LGPD | LGPD Art. 8º + ANPD Dark Patterns 2024 | 8-16h | Risco multa evitado | tracking-engineer |
| P0.2 | Default consent state granted | D8 | BLOCKER LGPD | LGPD Art. 7º | 2-4h | Risco multa evitado | tracking-engineer |
| ... | ... | ... | ... | ... | ... | ... | ... |

### P1 — Sprint-1 (Alto ROI Técnico)

| # | Achado | Dimensão | Severidade | Esforço (h) | Impacto quantificado | Skill responsável |
|---|--------|----------|-----------|-------------|----------------------|--------------------|
| P1.1 | Hash não trafega | D4 | Alta | 4-8h | EMQ +2-4 pontos → +20-40% attribution Meta | instrumentation-engineer |
| P1.2 | EMQ <5.0 ("Poor") | D5 | Alta | 8-16h | EMQ +3-5 pontos → +40-60% attribution | instrumentation-engineer + tracking-meta-ads |
| ... | ... | ... | ... | ... | ... | ... |

### P2 — Sprint-2+ (Refinamento)

| # | Achado | Dimensão | Severidade | Esforço (h) | Impacto | Skill responsável |
|---|--------|----------|-----------|-------------|---------|--------------------|
| P2.1 | Safari ITP mitigation | D7 | Média | 16-40h | Safari attribution +70-85% (cliente Safari traffic ≥20%) | tracking-engineer + tracking-ga4 |
| ... | ... | ... | ... | ... | ... | ... |
```

---

## 11. Critérios pra elevar/rebaixar severidade

### 11.1 Elevar severidade

- Cliente em vertical regulado (saúde + financeiro + apostas) — eleva 1 grau em compliance issues
- Volume de tráfego alto (>100k visits/mês) — eleva 1 grau em tracking issues
- Cliente com NDA confidencial OR setor sensível (CONAR/CFM/ANVISA) — eleva 1 grau em PII issues

### 11.2 Rebaixar severidade

- Cliente N0 sem volume — pode rebaixar 1 grau em alguns gaps técnicos refinados
- Cliente em pivot — pode rebaixar 1 grau em gaps não-críticos antes de validar PMF

**Justificativa:** severidade default é canônica universal. Calibrações por contexto exigem justificativa no §5 do output (Severidades Canônicas) ou §11 (Limitações).

---

## 12. References cross-skill

- [`lgpd-compliance-canonicas.md`](lgpd-compliance-canonicas.md) — LGPD artigos + ANPD guias detalhados
- [`emq-formula-meta.md`](emq-formula-meta.md) — EMQ targets canônicos
- [`consent-mode-v2-spec.md`](consent-mode-v2-spec.md) — Consent Mode v2 spec + mudança jun/2026
- [`plano-remediacao-templates.md`](plano-remediacao-templates.md) — templates P0/P1/P2 detalhados
