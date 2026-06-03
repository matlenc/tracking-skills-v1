---
title: Auditoria de Tracking Black-Box — {{cliente}} / {{target_slug}}
type: snapshot
diataxis: reference
schema_id: auditoria-tracking-blackbox
schema_version: 1.0.0
versao: 1.0.0
data: "{{YYYY-MM-DD}}"
operador: "{{nome_operador}}"
cliente: "{{nome_cliente}}"
status: draft | revisao | aprovado
target_url: "{{URL auditada completa}}"
target_slug: "{{slug-curto-pra-path}}"
modo: "{{lp | site | ecommerce}}"
modelo_negocio: "{{b2b-leadgen | b2c-ticket-alto | b2c-consumer | ecommerce | plg-saas}}"
nivel_acesso_max: "{{1-9}}"
tempo_execucao_minutos: "{{integer}}"
cutoff_rubrica: "estado-da-arte 2026"
nota_final: "{{0-10}}"
categoria: "{{estado-da-arte | maduro | funcional | subdesenvolvido | inexistente}}"
confidence_global: "{{alta | media | baixa}}"
hard_blockers_count: "{{integer}}"
alta_severidade_count: "{{integer}}"
ferramentas_usadas: [playwright]
flags_acesso_usadas: []
linkbacks_upstream:
  - url: "{{URL auditada}}"
linkbacks_downstream:
  - skill: measurement-architect
    output: measurement-plan.yml
    quando: Fundação 1.3.2
mcp_requerido_input: [playwright, filesystem]
mcp_requerido_output: [filesystem]
aplicacao_externa_destino: [filesystem-vault]
formato_output_principal: dual MD + manifest YAML
fallback_documentado: "Níveis aditivos 1-9 — Nível 1 só URL via Playwright sempre funciona"
bibliotecas_consumidas: [playwright, pydantic-v2, hashlib]
mecanismos_claude_code: []
---

# Auditoria de Tracking Black-Box — {{cliente}}

> **Cutoff temporal:** estado-da-arte 2026 — atualização anual obrigatória (D-AUDITORES-6).
> **Modo:** `{{lp|site|ecommerce}}` — pesos calibrados por modelo `{{modelo_negocio}}`.
> **Nível de acesso máximo:** {{nivel_acesso_max}} (de 9 disponíveis).
> **Tempo de execução:** {{tempo_execucao_minutos}} min.

---

## 1. Executive Summary

**Nota Final:** {{nota_final}}/10 — Categoria **{{categoria}}**

**3 highlights críticos:**

🟢 **Verde** — {{achado_positivo_destaque}}

🔴 **Vermelho** — {{achado_critico_destaque}}

🟡 **Amarelo** — {{achado_atenção_destaque}}

**Hard blockers:** {{hard_blockers_count}} (todos LGPD/técnicos críticos)
**Alta severidade:** {{alta_severidade_count}} (impacto operacional alto)
**Confidence global:** {{alta|media|baixa}}

---

## 2. Inventário Detectado

### 2.1 Stack instalada

| Plataforma | Detectada? | ID | Versão |
|---|---|---|---|
| GA4 | {{sim/não}} | `G-XXXXXXX` | `gtag.js` |
| GTM | {{sim/não}} | `GTM-XXXXX` | `gtm.js` |
| GTM Server-side | {{sim/não}} | Stape signature / Cloud Run / Custom | — |
| Meta Pixel | {{sim/não}} | `Pixel ID XXXXX` | `fbevents.js` |
| Google Ads | {{sim/não}} | `AW-XXXXXX` | — |
| LinkedIn Insight Tag | {{sim/não}} | `_linkedin_partner_id: XXXXX` | — |
| TikTok Pixel | {{sim/não}} | `ttq id: XXXXX` | — |
| Microsoft Ads UET | {{sim/não}} | `uetq id: XXXXX` | — |
| Microsoft Clarity | {{sim/não}} | `clarity id: XXXXX` | — |
| Hotjar / FullStory | {{sim/não}} | — | — |
| RD Station | {{sim/não}} | — | — |
| HubSpot | {{sim/não}} | — | — |
| **Universal Analytics (DEPRECATED)** | {{sim/não}} | — | **BLOCKER técnico se sim** |

### 2.2 Eventos capturados durante jornada

| Evento | Disparado? | Frequência | Trigger observado |
|---|---|---|---|
| `page_view` | {{sim/não}} | {{N vezes}} | Automático Enhanced Measurement |
| `scroll` (25/50/75/100%) | {{sim/não}} | — | Enhanced Measurement |
| `cta_click` / custom CTA | {{sim/não}} | — | Click no hero/body/footer |
| `form_start` | {{sim/não}} | — | Focus em primeiro campo |
| `form_submit` / `generate_lead` / `Lead` | {{sim/não}} | — | Submit form com dados sintéticos |
| `outbound_click` | {{sim/não}} | — | Click em link externo |
| `phone_click` (`tel:`) | {{sim/não}} | — | Click em `<a href="tel:...">` |
| `whatsapp_click` (`wa.me/`) | {{sim/não}} | — | Click em link WhatsApp |
| `video_start/progress/complete` | {{sim/não}} | — | Embed YouTube/Vimeo cond. |
| `file_download` | {{sim/não}} | — | Click PDF/doc cond. |

### 2.3 Cookies detectados (categorizados)

| Categoria | Cookies presentes |
|---|---|
| Estritamente necessários | session, csrf, etc. |
| Funcionais | language, theme |
| Analytics | `_ga`, `_gid`, `_gat`, `_ga_<container>` |
| Marketing | `_gcl_aw`, `_fbp`, `_fbc`, `_uetsid`, etc. |
| Desconhecidos | {{lista}} |

### 2.4 Click IDs canônicos persistidos (test sintético)

| Click ID | Capturado? | Cookie name | Lifetime |
|---|---|---|---|
| `gclid` | {{sim/não}} | `_gcl_aw` | {{90d}} |
| `fbclid` | {{sim/não}} | `_fbc` | {{90d}} |
| `wbraid` | {{sim/não}} | — | — |
| `gbraid` | {{sim/não}} | — | — |
| `msclkid` | {{sim/não}} | — | — |
| `ttclid` | {{sim/não}} | `_ttp` | — |
| `li_fat_id` | {{sim/não}} | `li_fat_id` | — |

---

## 3. Pontuação por Dimensão

| # | Dimensão | Peso (modo {{modo}}) | Score 0-10 | Ponderado | Confidence | Justificativa |
|---|----------|---------------------|-----------|-----------|------------|---------------|
| D1 | Stack instalada vs benchmark | {{12%}} | {{score}} | {{ponderado}} | {{alta/media/baixa}} | {{justificativa-1-linha}} |
| D2 | DataLayer schema quality | {{12%}} | {{score}} | {{ponderado}} | {{alta/media/baixa}} | {{justificativa}} |
| D3 | Cobertura de jornada | {{8%}} | {{score}} | {{ponderado}} | {{alta/media/baixa}} | {{justificativa}} |
| D4 | Identity & Hash Policy (LGPD-aware) | {{12%}} | {{score}} | {{ponderado}} | {{alta/media/baixa}} | {{justificativa}} |
| D5 | EMQ Meta estimado | {{15%}} | {{score}} | {{ponderado}} | {{alta/media/baixa}} | {{justificativa}} |
| D6 | Dedup CAPI prontidão | {{12%}} | {{score}} | {{ponderado}} | {{alta/media/baixa}} | {{justificativa}} |
| D7 | Click IDs capture & persistence | {{12%}} | {{score}} | {{ponderado}} | {{alta/media/baixa}} | {{justificativa}} |
| D8 | Consent Mode v2 + LGPD | {{8%}} | {{score}} | {{ponderado}} | {{alta/media/baixa}} | {{justificativa}} |
| D9 | Enhanced Ecommerce GA4 | {{—/8%}} | {{score}} | {{ponderado}} | {{alta/media/baixa}} | {{só modo ecommerce}} |
| D10 | Tracking health + performance | {{9%}} | {{score}} | {{ponderado}} | {{alta/media/baixa}} | {{justificativa}} |
| D11 | Pipeline pós-LP (informativa) | informativa | — | — | {{media}} | {{pattern detectado: WhatsApp/RD/HS/etc.}} |
| **TOTAL** | | **100%** | | **{{nota_final}}** | **{{confidence_global}}** | **Categoria: {{categoria}}** |

---

## 4. Justificativas por Score

### 4.1 D1 — Stack instalada vs benchmark
{{Parágrafo 3-5 linhas explicando score D1 — quais plataformas detectadas vs benchmark esperado pro modelo de negócio. Universal Analytics ativo? GTM Server-side ausente?}}

### 4.2 D2 — DataLayer schema quality
{{Parágrafo — schema versioning Snowplow-style presente? Naming convention canônica? Cobertura eventos canônicos vs 13 eventos core taxonomy? Duplicação Custom GTM + Enhanced Measurement?}}

### 4.3 D3 — Cobertura de jornada
{{Parágrafo — PageView capturado em todas páginas? CTAs trackados? Form start + submit (Lead) capturado? Scroll milestones? Phone/WhatsApp click? Cross-domain handling (modo `site`)?}}

### 4.4 D4 — Identity & Hash Policy (LGPD-aware)
{{Parágrafo — Hash SHA-256 normalizado ANTES do hash? Cobertura campos AM Meta? Hash existe mas não trafega? PII em claro = BLOCKER LGPD?}}

### 4.5 D5 — EMQ Meta estimado
{{Parágrafo — fórmula canônica resultado vs targets ≥7.0 Good / 5.0-6.9 Fair / <5.0 Poor. Confidence `media` Nível 1 / `alta` Nível 4.}}

### 4.6 D6 — Dedup CAPI prontidão
{{Parágrafo — event_id UUID v4 presente em eventos críticos? Server-side endpoint detectado? Heurísticas inferência server-side (Stape/Cloud Run/N8N)?}}

### 4.7 D7 — Click IDs capture & persistence
{{Parágrafo — captura via URL parsing? Persistência ≥90 dias? Cookies canônicos _gcl_aw + _fbp + _fbc? wbraid + gbraid capturados? Safari ITP mitigation via SGTM 1st-party?}}

### 4.8 D8 — Consent Mode v2 + LGPD
{{Parágrafo — banner presente? Default state `denied`? Tags pré-aceite? Banner cosmético detectado? Consent Mode v2 fields (`ad_user_data` + `ad_personalization`)? Política linkada? Direitos Art. 18 explícitos?}}

### 4.9 D9 — Enhanced Ecommerce GA4 (só modo `ecommerce`)
{{Parágrafo — 11 eventos canônicos capturados? Items array integrity? Currency consistente? transaction_id único? Meta `Purchase` paralelo? Google Ads value-based bidding?}}

### 4.10 D10 — Tracking health + performance
{{Parágrafo — LCP <2.5s / INP <200ms / CLS <0.1? Console errors? Network failures tracking? Tags duplicadas? Race conditions? Cross-browser parity Safari vs Chrome?}}

### 4.11 D11 — Pipeline pós-LP (informativa)
{{Parágrafo — pattern detectado (WhatsApp frictionless / RD Station / HubSpot / Pipefy / CRM próprio). Implicação pra `measurement-architect` Fundação 1.3.2.}}

---

## 5. Severidades Canônicas

### 5.1 Hard Blockers ({{hard_blockers_count}})

| # | Achado | Dimensão | Severidade | Artigo/Spec Violada |
|---|--------|----------|-----------|---------------------|
| HB.1 | {{achado-1}} | D{{N}} | BLOCKER LGPD | LGPD Art. {{Nº}} + ANPD {{guia}} |
| HB.2 | {{achado-2}} | D{{N}} | BLOCKER técnico | {{Meta CAPI spec / Google deprecation / Consent Mode v2}} |
| ... | ... | ... | ... | ... |

### 5.2 Alta Severidade ({{alta_severidade_count}})

| # | Achado | Dimensão | Razão | Impacto |
|---|--------|----------|-------|---------|
| AS.1 | {{achado}} | D{{N}} | {{razão técnica/legal}} | {{impacto-operacional-quantificável}} |
| ... | ... | ... | ... | ... |

### 5.3 Média Severidade

{{Lista resumida}}

### 5.4 Baixa Severidade (Refinamentos)

{{Lista resumida}}

---

## 6. Plano de Remediação

> Priorização canônica P0/P1/P2 baseada em severidade × ROI (esforço × impacto-quantificado).

### P0 — Semana-1 (Blockers Legais + Técnicos Críticos)

| # | Achado | Dim | Severidade | Artigo/Spec | Esforço | Impacto | Skill responsável |
|---|--------|-----|-----------|-------------|---------|---------|--------------------|
| P0.1 | {{Achado}} | D{{N}} | BLOCKER LGPD | LGPD Art. {{Nº}} | {{X-Y h}} | Risco multa evitado | `tracking-engineer` |
| P0.2 | {{Achado}} | D{{N}} | BLOCKER técnico | {{spec}} | {{X-Y h}} | {{impacto}} | `instrumentation-engineer` |
| ... | ... | ... | ... | ... | ... | ... | ... |

### P1 — Sprint-1 (Alto ROI Técnico, ~30 dias)

| # | Achado | Dim | Severidade | Esforço | Impacto quantificado | Skill responsável |
|---|--------|-----|-----------|---------|----------------------|--------------------|
| P1.1 | {{Achado}} | D{{N}} | Alta | {{X-Y h}} | {{Impacto canônico}} | `instrumentation-engineer` + `tracking-{{plataforma}}` |
| ... | ... | ... | ... | ... | ... | ... |

### P2 — Sprint-2+ (Refinamento, ~60-90 dias)

| # | Achado | Dim | Severidade | Esforço | Impacto | Skill responsável |
|---|--------|-----|-----------|---------|---------|--------------------|
| P2.1 | {{Achado}} | D{{N}} | Média | {{X-Y h}} | {{Impacto}} | `tracking-engineer` |
| ... | ... | ... | ... | ... | ... | ... |

### Total estimado

| Tier | Itens | Esforço total | ROI total |
|------|-------|---------------|-----------|
| P0 | {{N}} | {{X-Y h}} | Risco multa LGPD evitado + correção blockers técnicos |
| P1 | {{N}} | {{X-Y h}} | EMQ +{{X}} pontos / attribution +{{Y}}% / ad spend efficiency |
| P2 | {{N}} | {{X-Y h}} | Data quality estruturada + refinamentos incrementais |
| **Total** | **{{N}}** | **{{X-Y h}}** | **Estado-da-arte alcançável em {{N}} sprints** |

---

## 7. Handoffs Canônicos

| Skill consumer | Quando | O que extrai dessa auditoria |
|---|---|---|
| `measurement-architect` (Fundação 1.3.2) | Cliente N1+ que já tem tracking parcial | Stack atual + gaps + EMQ baseline + hash policy gap |
| `instrumentation-engineer` v2 (Fase 2) | Implementação fixes P0-P2 técnicos | Plano remediação técnico P0-P2 |
| `tracking-engineer` (Fase 2) | Config GTM + plataformas + Consent v2 | Gaps Consent v2 + tags faltantes + LGPD remediation |
| `tracking-{plataforma}-*` (Fase 2 × 7) | Config plataforma específica | Sub-tasks por plataforma |
| `account-curator` (cross-fase) | Decisões arquiteturais detectadas | Decisões implícitas viram Decision Doc |
| `measurement-auditor` (Fase 3 D36) | Trimestre seguinte — drift vs baseline | Baseline cross-temporal pra drift comparativo |

---

## 8. Confidence Indicator Global

| Dimensão | Confidence | Razão | Upgrade recomendado |
|---|---|---|---|
| D1 | alta | 100% validável client-side via Playwright | — |
| D2 | {{baixa/alta}} | {{razão}} | `--gtm-export=<path>.json` (Nível 2) |
| D3 | {{baixa/media/alta}} | {{razão}} | `--ga4-credentials` (Nível 3) |
| D4 | {{media/alta}} | {{razão}} | `--meta-business-token` (Nível 4) |
| D5 | {{media/alta}} | Estimativa fórmula EMQ ±1 ponto | `--meta-business-token` (Nível 4) |
| D6 | {{media/alta}} | {{razão server-side}} | `--meta-business-token` (Nível 4) OR `--server-logs-path` (Nível 9) |
| D7 | alta | Test sintético via Playwright determinístico | — |
| D8 | alta | Validação empírica via Playwright determinística | — |
| D9 | {{media/alta}} | {{só modo ecommerce}} | `--ga4-credentials` (Nível 3) |
| D10 | alta | PerformanceObserver via Playwright determinístico | `--clarity-project-id` (Nível 8) qualitative |
| **Global** | **{{alta/media/baixa}}** | **{{N}}/10 dimensões `alta` + {{N}} `media` + {{N}} `baixa`** | **Nível {{N}} recomendado pra `alta` global** |

---

## 9. Meta-Rubrica — refinamentos detectados na execução

> Esta seção captura refinamentos da rubrica detectados durante esta execução. Alimentam pendência B7 (calibração rubrica anual).

{{Listar refinamentos detectados — novos sub-critérios não cobertos pela rubrica vigente; edge cases; padrões emergentes.}}

Exemplos canônicos do piloto Aquatro 2026-05-11:
- ⭐ D8 separar "banner ausente" vs "banner cosmético" (incorporado v1.0.0)
- ⭐ D4 sub-score intermediário "hash existe mas não trafega" (incorporado v1.0.0)
- ⭐ D2 sub-critério "Duplicação Custom GTM + Enhanced Measurement" (incorporado v1.0.0)
- ⭐ D6 heurísticas inferência server-side sem precisar Nível 4 (incorporado v1.0.0)
- ⭐ D10 LCP via PerformanceObserver (não `getEntriesByType` direto — incorporado v1.0.0)
- ⭐ D11 Pipeline pós-LP (dimensão nova informativa — incorporado v1.0.0)

---

## 10. Frontmatter — Metadata da execução

| Campo | Valor |
|---|---|
| **target_url** | {{URL}} |
| **target_slug** | {{slug}} |
| **modo** | {{lp/site/ecommerce}} |
| **modelo_negocio** | {{b2b-leadgen/b2c-ticket-alto/b2c-consumer/ecommerce/plg-saas}} |
| **nivel_acesso_max** | {{1-9}} |
| **flags_usadas** | {{lista}} |
| **tempo_execucao** | {{X min}} |
| **data_execucao** | {{YYYY-MM-DD HH:MM}} |
| **cutoff_rubrica** | estado-da-arte 2026 |
| **ferramentas_usadas** | playwright, hashlib, pydantic-v2 |
| **operador** | {{nome}} |
| **cliente** | {{nome}} |

---

## 11. Limitações + Pendências do Diagnóstico

### 11.1 Limitações Nível 1 (atual)

- D5 EMQ é **estimativa** (margem ±1 ponto vs Events Manager real) — upgrade pra Nível 4 (Meta Business) confirma valor real
- D6 server-side é **inferência heurística** (subdomínios Stape signature / Cloud Run / N8N) — upgrade pra Nível 4 ou 9 confirma dedup rate real
- D9 `purchase` value não testado (skill não-destrutiva) — upgrade pra Nível 3 (GA4 Admin) confirma conversion config
- D2 só vê schema dos pushes que dispararam DURANTE a jornada — upgrade pra Nível 2 (GTM Container) vê config completa

### 11.2 Recomendações de upgrade

| Upgrade | Flag | Beneficia | Esforço |
|---|---|---|---|
| Nível 2 | `--gtm-export=<path>.json` | D2 + D3 confidence sobe pra `alta` | Operador exporta JSON GTM |
| Nível 3 | `--ga4-credentials=<path>.json` | D3 + D9 confidence sobe pra `alta` | Operador cria Service Account Google |
| Nível 4 | `--meta-business-token=<token>` | D4 + D5 + D6 confidence sobe pra `alta` | Operador gera System User Token Meta |
| Nível 9 | `--server-logs-path=<path>` | D6 + D11 confidence sobe pra `alta` | Operador exporta logs Stape/Cloud Run |

### 11.3 Cenários não-cobertos pela auditoria atual

- {{Lista cond.: jornadas não-testadas / páginas restritas / state-based features (logged-in user, members area) / etc.}}

---

## 12. Anexos

### 12.1 Network Requests Inspecionados (filtrado tracking-only)

| Timestamp | URL | Method | Status |
|---|---|---|---|
| {{HH:MM:SS}} | `https://www.facebook.com/tr/?id=XXX&ev=Lead&...` | POST | 200 |
| {{HH:MM:SS}} | `https://www.google-analytics.com/g/collect?...` | POST | 204 |
| ... | ... | ... | ... |

Total: {{N}} requests tracking inspecionados / {{M}} totais (filtrado static assets).

### 12.2 Cookies Detectados (completo)

```json
[
  {"name": "_ga", "domain": ".cliente.com.br", "expires": "...", "secure": true, "httpOnly": false},
  {"name": "_fbp", "domain": ".cliente.com.br", "expires": "...", "secure": true, "httpOnly": false},
  ...
]
```

### 12.3 DataLayer Pushes Capturados (durante jornada)

```json
[
  {"time": 1715098234000, "args": [{"event": "page_view", "page_title": "..."}]},
  {"time": 1715098240000, "args": [{"event": "form_start", "form_id": "..."}]},
  {"time": 1715098245000, "args": [{"event": "generate_lead", "value": "..."}]},
  ...
]
```

### 12.4 Meta `/tr` Payloads Capturados

```json
[
  {
    "url": "https://www.facebook.com/tr/?id=XXX&ev=Lead&...",
    "event": "Lead",
    "event_id": "uuid-v4",
    "ud_em": "sha256_hash_or_null",
    "ud_ph": "sha256_hash_or_null",
    "fbp": "fb.1.timestamp.value",
    "fbc": "fb.1.timestamp.fbclid_value"
  },
  ...
]
```

### 12.5 Console Errors + Network Failures

{{Lista cond.}}

---

> **Auditoria gerada via skill `tracking-blackbox-auditor` v1.0.0** — Tese Growth IA Ops v2.0 — linha `auditor-*` cross-fase. Rubrica universal canônica estado-da-arte 2026 (D-AUDITORES-6 — atualização anual obrigatória).
