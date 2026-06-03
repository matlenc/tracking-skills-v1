---
name: tracking-blackbox-auditor
description: |
  Auditora black-box estado-da-arte 2026 da Tese Growth IA Ops v2.0 — primeira skill
  da linha `auditor-*` cross-fase. Recebe URL pública (default Nível 1 frictionless)
  com upgrade opt-in via flags de acesso (Níveis 2-9 — GTM container / GA4 Admin /
  Meta Business / Google Ads / LinkedIn / TikTok / Clarity / CRM / server logs) e
  produz nota ponderada 0-10 + severidades canônicas mapeadas pra artigo LGPD ou
  spec técnica + plano de remediação priorizado por ROI + handoffs canônicos pra
  skills downstream — tudo via Playwright nativo (universal P1).

  Bounded context: auditoria black-box de infraestrutura de tracking/mensuração em
  LP/site/ecommerce externo. NÃO modifica nada do alvo — só observa, mede, diagnostica.
  Rubrica universal estado-da-arte 2026 com 10 dimensões canônicas + D11 opcional ×
  pesos por modo (lp B2B / lp B2C / site / ecommerce). Validação empírica realizada
  no piloto LP Aquatro Suprimentos 2026-05-11 — 6 refinamentos incorporados (banner
  cosmético D8 / hash não-trafega D4 / duplicação D2 / inferência server-side D6 /
  LCP via PerformanceObserver D10 / pipeline pós-LP D11).

  3 modos canônicos mutuamente exclusivos (D-AUDITORES-1):
  (a) `--modo=lp` — default standalone; LP única com objetivo de Lead capture
      (form OR redirect WhatsApp OR phone). Ênfase D4+D5 (Identity & Hash + EMQ)
      + D7 (Click IDs) — juntos 39% peso B2B leadgen / 37% B2C ticket alto;
  (b) `--modo=site` — cliente N2+ multi-page institucional/blog com ≥3 PageView
      paths + content marketing + Lead em múltiplas páginas. Ênfase D3 (18% peso)
      + cross-domain handling + content_group dimension + client_id persistente;
  (c) `--modo=ecommerce` — loja com cart + checkout + product pages + payment +
      confirmation. D9 Enhanced Ecommerce GA4 obrigatório (8% peso) com 11 eventos
      canônicos (view_item_list → ... → purchase → refund).

  9 níveis de acesso opt-in aditivos (D-AUDITORES-7) — cada flag sobe confidence
  indicator da dimensão correspondente pra `alta`. Nível 1 (só URL via Playwright)
  entrega baseline 70-80% confidence média em algumas dimensões. Níveis 2-9
  empilháveis: GTM Container (--gtm-export) / GA4 Admin (--ga4-credentials) /
  Meta Business (--meta-business-token) / Google Ads (--google-ads-credentials) /
  LinkedIn (--linkedin-token) / TikTok (--tiktok-token) / Clarity (--clarity-project-id) /
  Server logs ou CDP (--server-logs-path | --cdp-token).

  Severidades canônicas mapeadas pra artigo legal/spec técnica (D-AUDITORES-3) —
  BLOCKER LGPD (Art. 5º/7º/8º/9º/18º + ANPD Guia Cookies 2023 + ANPD Guia Dark
  Patterns 2024) | BLOCKER técnico (Meta CAPI spec + Google deprecation notices
  + Consent Mode v2 spec mar/2024) | Alta/Média/Baixa severidade técnica. Não
  inflar score sem evidência — confidence indicator obrigatório por dimensão.

  Output canônico em 12 seções obrigatórias (Executive Summary · Inventário ·
  Pontuação por dimensão · Justificativas · Severidades · Plano remediação P0/P1/P2 ·
  Handoffs · Confidence global · Meta-rubrica OR Sumário cliente · Frontmatter ·
  Limitações · Anexos) + manifest YAML co-publicado. Dual output:
  (a) modo Fundação 1.3.2 / Fase 3 — `<vault>/20-snapshots/YYYY-MM/`;
  (b) modo standalone comercial — `<output-dir>/` cliente-facing (MD default +
      HTML Reveal.js opcional via `--render=reveal`).

  Pré-condição operacional crítica (D-AUDITORES-5): NUNCA confiar só no JS
  interceptor (monkey-patch dataLayer.push / fbq / fetch override) — `window.location.href`
  setter escapa interceptação JS no Chrome. Estratégia canônica: Playwright nativo
  `browser_network_requests` como redundância obrigatória sempre, capturando
  dentro do browser engine independente do JS rodar. Lição empírica Aquatro.

  Cutoff temporal explícito: "estado-da-arte 2026" — atualização anual obrigatória
  (D-AUDITORES-6; pendência política `arquitetura/politicas/calibracao-rubricas-auditoras.md`
  B7). Validação empírica via piloto real ANTES da auditoria fechar é convenção da
  linha (D-AUDITORES-8) — modo `lp` B2B validado Aquatro; modos `site` + `ecommerce`
  ficam pendência Bloco D piloto.

  Ative quando o operador disser auditar tracking, audit tracking, analisar
  mensuração de URL, diagnóstico de tracking black-box, EMQ estimate, EMQ score
  black-box, LGPD audit cookie banner, Consent Mode v2 audit, validar implementação
  Meta CAPI / Google Ads / GA4 / GTM externo, rubrica tracking, tracking-blackbox-auditor.
  Também ative quando consumido como input formal pela Fundação 1.3.2 pra dimensionar
  Measurement Plan/Contrato pelo gap real (não premissa), ou pela Fase 3 Loop
  trimestral como auditoria cross-temporal cerimonial.

  Não ative para: produzir Measurement Plan / Contrato de Dados greenfield (use
  measurement-architect — esta skill INFORMA mas não SUBSTITUI), drift detection
  contra Plan/Contrato declarados pelo MESMO projeto (use measurement-auditor
  Fase 3 — bounded context disjunto: drift interno vs black-box estado-da-arte),
  auditoria de saúde técnica de CONTA DE ADS (use account-health-auditor — Health
  Score 0-100 + Kill Rules + Quality Gates conta cross-canal), SEO técnico ou
  crawl/indexação (use seo-technical-auditor W-1 — discoverability ≠ mensuração),
  email deliverability (use outbound-deliverability-auditor — SPF/DKIM/DMARC/BIMI),
  implementar fixes técnicos detectados (use instrumentation-engineer v2 + família
  tracking-* da Fase 2 — esta skill produz plano de remediação, não executa).
version: "1.0.0"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, WebSearch, WebFetch
---

# `tracking-blackbox-auditor` — Auditora black-box estado-da-arte 2026 (linha `auditor-*` cross-fase)

Skill é **auditora black-box** — recebe input externo público (URL/repo/doc) e valida contra **rubrica universal canônica estado-da-arte 2026**, NÃO contra contrato interno do projeto. Diferencial vs `measurement-auditor` (drift contra `measurement-plan.yml` próprio do projeto): aqui não precisa de Plan/Contrato prévios; rubrica é universal externa. **Não modifica nada do alvo** — só observa, mede, diagnostica. Output é input pra skills downstream que sim executam mudanças.

> **Princípios canônicos:** Avo "bad data is worse than no data" aplicado à auditoria — `confidence` indicator por dimensão sinaliza quando evidência é insuficiente; rubrica recusa fechar score sem evidência ≥ média na dimensão crítica + 6 refinamentos do piloto Aquatro 2026-05-11 + cutoff temporal explícito "estado-da-arte 2026" (D-AUDITORES-6 atualização anual obrigatória).

---

## Quando usar

- **Caso de uso comercial dominante** (modo standalone) — porta de entrada cliente novo (`--modo=lp` Nível 1 default). Vendável R$ 1.500-5.000 OR bundle no kickoff premium. Cliente vê gap concreto antes de aprovar Measurement Plan. Tier 2 (Níveis 2-9 com credenciais opt-in) vira upsell pós-kickoff.
- **Caso de uso interno dominante** — input formal pra **Fundação Subfase 1.3.2** (`measurement-architect` consome auditoria pra dimensionar Plan/Contrato pelo gap real, não premissa). Reduz over-engineering em cliente N2+ que já tem 70% maduro.
- **Fase 3 Loop trimestral cerimonial** — auditoria cross-temporal comparativa (baseline trimestral vs. atual) com baseline persistido em `measurement-auditor` Fase 3.

## Quando NÃO usar

- **Produzir Measurement Plan / Contrato de Dados greenfield** → use `measurement-architect` (1.3.2) — esta skill **informa** mas não **substitui**
- **Drift detection contra `measurement-plan.yml` declarado no MESMO projeto** → use `measurement-auditor` (Fase 3 D36) — bounded context disjunto (drift interno vs black-box estado-da-arte)
- **Saúde técnica de CONTA DE ADS Meta/Google/LinkedIn/TikTok** → use `account-health-auditor` (Camada Data IA Fase 3 — D-MCP-20 + Sessão C) — Health Score 0-100 + Kill Rules + Quality Gates conta cross-canal
- **SEO técnico (crawl + indexação + structured data)** → use `seo-technical-auditor` (W-1) — discoverability ≠ mensuração (bounded contexts ortogonais)
- **Email deliverability (SPF/DKIM/DMARC/BIMI/sender reputation)** → use `outbound-deliverability-auditor` (W-1) — infra entregabilidade ≠ infra mensuração
- **Implementar fixes técnicos detectados** → use `instrumentation-engineer` v2 + família `tracking-*` × 7 (Fase 2) — esta skill produz plano de remediação P0/P1/P2, não executa

---

## Modos canônicos (D-AUDITORES-1 — skill única com modos)

Operador declara **um modo por execução** — modos mutuamente exclusivos. Vocabulário+rubrica+jornada simulada diferentes cross-modo. Promoção pra família flat (`tracking-blackbox-lp-auditor` / `-site` / `-ecommerce`) avaliada em B7 quando 2+ projetos pedirem isolamento por target.

| Modo | Default? | Trigger canônico | Pesos peculiares |
|---|---|---|---|
| **`--modo=lp`** | Default standalone | LP única com objetivo Lead capture (form OR redirect WhatsApp OR phone OR similar) | D4+D5 = 27% B2B / 25% B2C; D7 = 12% (identidade do contato é o que vale) |
| **`--modo=site`** | Cliente N2+ multi-page | Site institucional/blog/B2B multi-page com ≥3 PageView paths + content marketing + Lead em múltiplas páginas | D3 = 18% (cobertura jornada multi-page + cross-domain handling + content_group + client_id persistente) |
| **`--modo=ecommerce`** | Cliente vertical ecommerce | Loja com cart + checkout + product pages + payment + confirmation | D9 Enhanced Ecommerce GA4 obrigatório = 8%; 11 eventos canônicos (view_item_list → ... → purchase → refund) |

**Calibração por modelo de negócio dentro de `--modo=lp`:**

| Modelo | Pesos calibrados |
|---|---|
| `lp` B2B leadgen (default) | D5 EMQ 15% + D4 Identity 12% + D7 Click IDs 12% (identidade do contato é o que vale) |
| `lp` B2C ticket alto (clínicas premium, escolas privadas, plano saúde) | D8 Consent + LGPD 12% (regulação mais escrutinada CONAR/PROCON) + D3 jornada 10% |
| `lp` B2C ticket baixo (consumer) | Próximo do default mas com D8 12% |

**Validação empírica via piloto real** (D-AUDITORES-8 — convenção da linha): modo `lp` B2B validado Aquatro 2026-05-11. Modos `site` + `ecommerce` ficam **pendência Bloco D piloto** — sub-críterios podem ganhar refinamento adicional cross-modo.

---

## Flags de acesso opt-in — Níveis 1-9 (D-AUDITORES-7)

Cada flag é **aditiva** sobre o anterior. Operador empilha flags. Confidence indicator por dimensão sobe pra `alta` conforme acesso adicional disponível.

| # | Nível | Flag | Adiciona cobertura |
|---:|---|---|---|
| 1 | **Só URL** (default frictionless) | (nenhuma) | Baseline 70-80% — tudo client-side via DOM + globals + dataLayer + cookies + network requests via Playwright |
| 2 | GTM Container | `--gtm-export=<path>.json` | +10% — triggers config, tags pausadas/ocultas, variables, workspace state, versionamento, GTM consent mode setup |
| 3 | GA4 Admin | `--ga4-credentials=<path>.json` | +5% — conversion events config, custom dimensions/metrics, audiences, data filters, Enhanced Measurement state, BigQuery export |
| 4 | Meta Events Manager | `--meta-business-token=<token>` | +5% — **EMQ REAL** (não estimado), CAPI events recebidos vs enviados, deduplication rate, server events errors, AEM priority |
| 5 | Google Ads | `--google-ads-credentials=<path>.json` | +3% — conversion actions config, Customer Match, Enhanced Conversions state, Conversion Linker, Attribution model |
| 6 | LinkedIn Campaign Manager | `--linkedin-token=<token>` | +2% — Insight Tag conversion tracking, LinkedIn Conversions API, audience setup |
| 7 | TikTok Ads Manager | `--tiktok-token=<token>` | +2% — Pixel Events Manager, CAPI events, audiences |
| 8 | Clarity / Hotjar / FullStory | `--clarity-project-id=<id>` (etc.) | +2% qualitativo — heatmaps, recordings, rage clicks, dead clicks |
| 9 | Server logs / CDP / CRM | `--server-logs-path=<path>` ou `--cdp-token=<token>` | +3% — end-to-end attribution, CAPI server-side trace, conversion path completa |

**Extras (não-acesso):**
- `--cwv=true/false` (default `true` modos `lp`/`ecommerce`) — Core Web Vitals via PerformanceObserver (refinamento Aquatro — não `performance.getEntriesByType` direto)
- `--lgpd-scan=true/false` (default `true`) — cookie scanner LGPD categorizado
- `--browsers=chrome[,safari][,firefox]` — default `chrome`; Safari adiciona ~3 min execução (ITP impact analysis)
- `--measurement-plan=<path>.yml` — auditoria híbrida (compara implementação observada vs Plan documentado — adiciona §"Drift vs Plan declarado")
- `--render=reveal|markdown-only` (default `markdown-only`) — Reveal.js HTML render pra cliente-facing modo standalone

**Princípio operativo:** auditoria roda em qualquer nível. Nível 1 entrega diagnóstico completo com `confidence: media` em algumas dimensões; cada flag adicional sobe `confidence: alta` na dimensão correspondente. Operador escolhe profundidade vs friction.

---

## Inputs / Outputs canônicos

### Inputs

| Input | Obrigatório? |
|---|---|
| URL alvo público (Nível 1) | Sim |
| `--modo={lp,site,ecommerce}` | Sim |
| `--output-dir=<path>` | Sim modo standalone; opcional Fundação/F3 (default `<vault>/20-snapshots/YYYY-MM/`) |
| Flags Níveis 2-9 + extras | Opt-in conforme acesso disponível |

### Outputs

**Modo Fundação 1.3.2 OR Fase 3 (operador-facing — Categoria 2 Snapshot):**

| # | Output | Path canônico |
|---|--------|--------------|
| 1 | Auditoria principal MD | `<vault>/20-snapshots/YYYY-MM/auditoria-tracking-<target-slug>.md` |
| 2 | Manifest YAML | `<vault>/20-snapshots/YYYY-MM/auditoria-tracking-<target-slug>.manifest.yml` |

**Modo standalone comercial (cliente-facing — externa):**

| # | Output | Path canônico |
|---|--------|--------------|
| 1 | Auditoria principal MD | `<output-dir>/auditoria-tracking-<target-slug>-<YYYY-MM-DD>.md` |
| 2 | Auditoria HTML Reveal.js (cond.) | `<output-dir>/auditoria-tracking-<target-slug>-<YYYY-MM-DD>.html` (quando `--render=reveal`) |
| 3 | Manifest YAML | `<output-dir>/auditoria-tracking-<target-slug>-<YYYY-MM-DD>.manifest.yml` |

---

## Repertório (references densos)

- [`references/rubrica-tracking-blackbox.md`](references/rubrica-tracking-blackbox.md) — **CORE** — 10 dimensões canônicas × sub-critérios × pesos por modo (lp B2B / lp B2C / site / ecommerce) × thresholds × scoring rubric 0-10 × confidence por dimensão. Embute rubrica testada empiricamente Aquatro 2026-05-11. Cutoff "estado-da-arte 2026" — atualização anual obrigatória (D-AUDITORES-6).
- [`references/procedures-playwright-tracking.md`](references/procedures-playwright-tracking.md) — Snippets Playwright literais + monkey-patches dataLayer/fbq + interceptors fetch/XHR/sendBeacon/Image.src + estratégia `browser_network_requests` cross-navigation. **Pattern crítico (D-AUDITORES-5):** "NUNCA confiar só no JS interceptor — Playwright nativo `browser_network_requests` como redundância obrigatória" (lição Aquatro — `window.location.href` setter escapa monkey-patch JS).
- [`references/emq-formula-meta.md`](references/emq-formula-meta.md) — Fórmula EMQ Meta 2024-2026 (base 2.0 + email_match 0-2.0 + phone_match 0-2.0 + name_match 0-1.5 + address_match 0-1.0 + external_id 0-1.5 + IP/UA 0-0.5 cada + fbc 0-1.0 + fbp 0-0.5). Targets ≥7.0 ideal / 5.0-6.9 Fair / <5.0 Poor. Benchmarks por evento (Purchase 8.8-9.3 / AddToCart 8.0+ / Lead 7.0+ / PageView 6.5-7.5).
- [`references/consent-mode-v2-spec.md`](references/consent-mode-v2-spec.md) — Spec Google Consent Mode v2 (mar/2024) — 7 signals (ad_storage + analytics_storage + ad_user_data + ad_personalization + functionality_storage + personalization_storage + security_storage) + Basic vs Advanced + behavior modeling + **mudança junho/2026** (split GA4/Ads — `ad_storage` controla GA4 separado de Ads).
- [`references/lgpd-compliance-canonicas.md`](references/lgpd-compliance-canonicas.md) — LGPD Art. 5º (definições) + Art. 7º (bases legais) + Art. 8º (consentimento livre+informado+inequívoco) + Art. 9º (info ao titular) + Art. 18º (direitos do titular) + ANPD Guia Cookies set/2023 + ANPD Guia Dark Patterns 2024 + ANPD Mapa Temas Prioritários 2026-2027 (fiscalização ativa + sanções até 2% receita / R$ 50M/violação). Mapeamento blockers severidade → artigo violado.
- [`references/click-ids-taxonomy-2026.md`](references/click-ids-taxonomy-2026.md) — Taxonomia completa cross-platform (gclid/fbclid/wbraid/gbraid/msclkid/ttclid/li_fat_id/dclid/epik) + persistência canônica (90 dias 1st-party cookie + localStorage backup) + first-touch vs last-touch + **Safari ITP / Link Tracking Protection 2024-2026** (strips gclid+fbclid das URLs — sub-critério "Safari ITP resilience" via SGTM 1st-party).
- [`references/enhanced-ecommerce-ga4-canonicas.md`](references/enhanced-ecommerce-ga4-canonicas.md) — 11 eventos canônicos GA4 (view_item_list / select_item / view_item / add_to_cart / remove_from_cart / view_cart / begin_checkout / add_shipping_info / add_payment_info / purchase / refund) + items array schema + currency consistency cross-events + `transaction_id` único (não-duplicado em refresh) + value = soma items + Meta `Purchase` paralelo.
- [`references/severidades-canonicas.md`](references/severidades-canonicas.md) — Hard blockers (LGPD Art. 5º/7º/8º/9º/18º + Universal Analytics deprecated jul/2023 + Consent Mode default granted + PII em claro + event_id ausente com CAPI server-side + banner cosmético D8 ⭐ Aquatro) + alta/média/baixa severidade técnica + matrix ROI esforço × impacto.
- [`references/confidence-indicator-rubrica.md`](references/confidence-indicator-rubrica.md) — Confidence por dimensão × por nível de acesso 1-9. Quando confidence é alta (validável 100% client-side) / media (estimada client-side, confirmação requer API server) / baixa (cobertura limitada client-side). Threshold canônico pra upgrade.
- [`references/plano-remediacao-templates.md`](references/plano-remediacao-templates.md) — Templates P0 (blockers legais — semana-1) / P1 (alto ROI técnico — sprint-1) / P2 (refinamento — sprint-2+) × esforço-horas × impacto-quantificado + matrix ROI (esforço × impacto).

## Templates

- [`templates/auditoria-tracking-template.md`](templates/auditoria-tracking-template.md) — Esqueleto operador-facing Fundação 1.3.2 / Fase 3 — 12 seções obrigatórias densas + frontmatter completo + meta-rubrica.
- [`templates/auditoria-tracking-cliente-template.md`](templates/auditoria-tracking-cliente-template.md) — Esqueleto cliente-facing standalone Markdown — 12 seções traduzidas pra impacto de negócio (não jargão técnico denso) + chamada-de-ação no Sumário Executivo.
- [`templates/auditoria-tracking-cliente-template.html`](templates/auditoria-tracking-cliente-template.html) — Reveal.js render cliente-facing (quando `--render=reveal`) — slides estruturados das 12 seções com transições + tema dark default + responsivo.

## Schemas

- [`schemas/auditoria-tracking-manifest.yml`](schemas/auditoria-tracking-manifest.yml) — Manifest YAML co-publicado com a auditoria MD. Schema validável (Pydantic v2 cond.) — target_url + target_slug + modo + nivel_acesso_max + cutoff_rubrica + nota_final + categoria + scores_por_dimensao + hard_blockers + plano_remediacao + handoffs + linkbacks + limitacoes.

---

## Workflow canônico (any modo, any nível)

### Passo 0 — Iniciação

1. **Pré-condição Playwright MCP** (universal P1 — biblioteca shared D-OPERADOR-1) — `claude mcp list` confirma. Se ausente → recusa rodar + recomenda `system-installer setup-platform-skill tracking-blackbox-auditor`.
2. **Pré-condição URL alvo** — `GET <target_url>` retorna 200 OK? Se 404/500 → recusa fechar + recomenda operador validar URL. Se cross-origin redirect → audita destino final + reporta redirect chain.
3. **Pré-condição modo declarado** — operador declarou `--modo={lp,site,ecommerce}`? Se não → pede declaração + valida coerência (modo `ecommerce` em LP sem cart/checkout = recusa).
4. **Pré-condição output-dir** — modo standalone: operador declarou `--output-dir`? Modo Fundação/F3: vault inicializado em `<vault>/20-snapshots/YYYY-MM/`?
5. Lê `references/rubrica-tracking-blackbox.md` + `references/procedures-playwright-tracking.md` + reference da dimensão crítica do modo (D5 EMQ pra `lp` / D3 cobertura pra `site` / D9 EE pra `ecommerce`).

### Passo 1 — Inventário e detecção stack (D1)

Aplica `references/procedures-playwright-tracking.md §1` (Stack detection):
- Navega `<target_url>` via Playwright Chromium (default) — captura `page.content()` + `window.*` globals + network requests filtrados (`googletagmanager.com|facebook.com|google-analytics.com|...`)
- Detecta 14 plataformas canônicas (GA4 + GTM + GTM Server-side + Meta Pixel + Google Ads + LinkedIn Insight + TikTok + Microsoft UET + Clarity + Hotjar + FullStory + RD Station + HubSpot + Universal Analytics deprecated)
- Score D1 vs benchmark esperado por modelo de negócio (B2B leadgen / B2C ticket alto / ecommerce / PLG SaaS)

### Passo 2 — Captura passiva durante jornada simulada (D2 + D3)

Aplica `references/procedures-playwright-tracking.md §2-§3`:
- **Monkey-patch dataLayer.push** (D2) — `page.evaluate("() => { window._auditPushes=[]; const orig=window.dataLayer.push.bind(window.dataLayer); window.dataLayer.push=(...args)=>{window._auditPushes.push({time:Date.now(),args:JSON.parse(JSON.stringify(args))});return orig(...args);}; }")`
- **Habilita `page.on('request')` + `page.on('response')`** — captura nativa Playwright (NÃO confiar só no monkey-patch — `window.location.href` setter escapa)
- **Simulação jornada user real** (D3 modo `lp`/`site`):
  - PageView automático em 3-5 páginas (modo `site`)
  - Click cada CTA principal (hero + body + footer)
  - Focus em form → Form start
  - Preencher form com dados sintéticos (`email: audit-test+<timestamp>@auditor-blackbox.com` + `phone: +5511999999999` + `name: Audit Tester`) + submit → Lead/MQL/generate_lead
  - Scroll milestones 25/50/75/100%
  - Click outbound + phone (`tel:`) + WhatsApp (`wa.me/`)
- **Modo `ecommerce`** — D9: browse listing → click item → add to cart → checkout → confirmation (11 eventos)

### Passo 3 — Identity & Hash Policy (D4) + EMQ Meta estimado (D5)

Aplica `references/emq-formula-meta.md` + `references/procedures-playwright-tracking.md §4`:
- Intercepta payload Meta `/tr` via `page.on('request')` filtrado em `facebook.com/tr|tr/?id=`
- Inspeciona parâmetros `ud[em]`, `ud[ph]`, `ud[fn]`, `ud[ln]`, `ud[ct]`, `ud[st]`, `ud[zp]`, `ud[country]`, `ud[ge]`, `ud[db]`, `ud[external_id]`, `ud[client_ip_address]`, `ud[client_user_agent]`, `fbc`, `fbp`
- **Hash verification:** cada campo PII tem exatamente 64 chars hex lowercase? Test sintético — passa `email: test@example.com` → SHA-256 esperado = `973dfe463ec85785f5f95af5ba3906eedb2d931c24e69824a89ea65dba4e813b`. Se bate → normalização correta; se não → sem trim/lowercase
- **Sub-score intermediário D4 "hash existe mas não trafega"** (refinamento Aquatro) — cookie `gpages_conversions` tem `ph` SHA-256 perfeito MAS `fbq('track', 'Lead', cd)` NÃO inclui `ud[*]`. Hash existe client-side mas não chega no destination. Score 3-5/10 (intermediário entre "hash ausente" 0 e "hash trafega" 10)
- **EMQ score** estimado via fórmula canônica (base 2.0 + componentes 0.0-2.0 cada). Confidence `media` Nível 1; `alta` Nível 4 (Meta Business Manager confirma EMQ real)

### Passo 4 — Dedup CAPI prontidão (D6) + Click IDs (D7)

Aplica `references/procedures-playwright-tracking.md §5-§6` + `references/click-ids-taxonomy-2026.md`:
- **D6 Dedup CAPI:** procura `event_id` em eventos críticos (Lead/Purchase/CompleteRegistration). Formato UUID v4 canônico (`xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`). Heurísticas inferência server-side sem precisar Nível 4 (refinamento Aquatro):
  - Subdomínios suspeitos: `events.*` / `tracking.*` / `tagm.*` / `gtm-server.*` (Stape signature)
  - Cloud Run subdomain pattern (`*.run.app`)
  - N8N webhook visible em network (`/webhook/...`)
  - CNAME cloaking patterns
- **D7 Click IDs:** Playwright navega URL com query param sintético — `?gclid=AUDIT_TEST_GCLID_123&fbclid=AUDIT_TEST_FBCLID_456&wbraid=AUDIT_WBRAID&gbraid=AUDIT_GBRAID&msclkid=AUDIT_MSCLKID&ttclid=AUDIT_TTCLID&li_fat_id=AUDIT_LI_FAT_ID`
- Após navegação: inspeciona cookies (`_gcl_aw`, `_fbp`, `_fbc`) + localStorage + sessionStorage pra detectar persistência
- Form submit + verifica se click ID é incluído em payload outbound (CRM submission OR Meta CAPI payload)
- **Safari ITP / Link Tracking Protection 2024-2026** — sub-critério "Safari ITP resilience" via SGTM 1st-party (cookies setados via HTTP server-side bypassam ITP)

### Passo 5 — Consent Mode v2 + LGPD compliance (D8)

Aplica `references/consent-mode-v2-spec.md` + `references/lgpd-compliance-canonicas.md`:
- Observa `gtag('consent', ...)` calls antes/depois do banner via monkey-patch + cookies pré-aceite + inspeção banner DOM
- **Validação empírica obrigatória:** navega sem aceitar → tags disparam? Se sim = BLOCKER LGPD Art. 8º (consent inválido)
- Aceita banner → `gtag('consent', 'update', {...granted})` é chamado? Se NÃO = **banner cosmético** ⭐ (refinamento Aquatro — BLOCKER LGPD Art. 8º + ANPD Guia Dark Patterns 2024)
- Default consent state = `denied` LGPD compliant OR `granted` = BLOCKER LGPD Art. 7º (coleta sem base legal)
- Consent Mode v2 fields `ad_user_data` + `ad_personalization` presentes (obrigatórios desde mar/2024)
- Política privacidade linkada + direitos LGPD Art. 18º explícitos + botão "Rejeitar" igual destaque "Aceitar" (LGPD/CONAR/ANPD dark pattern avoidance)

### Passo 6 — Tracking health + performance (D10) + Pipeline pós-LP (D11 cond.)

Aplica `references/procedures-playwright-tracking.md §7-§8`:
- **D10 LCP via PerformanceObserver** (refinamento Aquatro — NÃO `performance.getEntriesByType` direto):
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
- Targets canônicos 2026: LCP <2.5s + INP <200ms + CLS <0.1 = "Good" (web.dev)
- Console errors + network failures 4xx/5xx pra endpoints tracking + tags duplicadas + race conditions
- Cross-browser parity cond. (`--browsers=chrome,safari` — Safari ITP impacto)
- **D11 Pipeline pós-LP** (opcional informativa — não pondera score) — observa URL destino do submit (`api.whatsapp.com/send/?...` = frictionless WhatsApp / `rdstation.com/conversions/` = RD / `forms.hubspot.com` = HubSpot / `pipefy.com` / redirect próprio `/obrigado` = CRM próprio OR N8N)

### Passo 7 — Pontuação + severidades + plano remediação

Aplica `references/rubrica-tracking-blackbox.md §scoring`:
- Score 0-10 por dimensão × peso modo × confidence indicator (alta/media/baixa)
- Soma ponderada → Nota final 0-10 → Categoria canônica (estado-da-arte 9-10 / maduro 7-8 / funcional 5-6 / subdesenvolvido 3-4 / inexistente 0-2)
- Mapeia achados pra **severidades canônicas** (`references/severidades-canonicas.md`):
  - BLOCKER LGPD (Art. + ANPD spec)
  - BLOCKER técnico (Meta CAPI spec / Google deprecation / Consent Mode v2)
  - Alta/Média/Baixa severidade técnica
- Constrói **plano remediação P0/P1/P2** (`references/plano-remediacao-templates.md`):
  - P0 = blockers legais (LGPD + UA deprecated) — sprint-0 imediato
  - P1 = alto ROI técnico (EMQ <5 → Meta CAPI server-side / hash não-trafega / event_id ausente) — sprint-1
  - P2 = refinamento (duplicação D2 / cobertura jornada gaps / cross-browser parity) — sprint-2+

### Passo 8 — Output 12 seções + manifest YAML + render cond.

Lê template apropriado:
- Modo Fundação/F3: `templates/auditoria-tracking-template.md`
- Modo standalone Markdown: `templates/auditoria-tracking-cliente-template.md`
- Modo standalone Reveal.js (quando `--render=reveal`): `templates/auditoria-tracking-cliente-template.html`

Escreve **12 seções obrigatórias** (Executive Summary + Inventário + Pontuação por dimensão + Justificativas + Severidades + Plano remediação + Handoffs + Confidence global + Meta-rubrica OR Sumário cliente + Frontmatter + Limitações + Anexos). Co-publica manifest YAML (`schemas/auditoria-tracking-manifest.yml` — validável Pydantic v2 cond.).

Aplica **critérios de done** (§8.1 da skill canônica) — recusa fechar se ≥1 critério blocker falha (output 12 seções incompleto / nota sem categoria / dimensão sem confidence / severidade sem mapping artigo legal / plano sem priorização ROI).

---

## Anti-patterns invariantes (recusados sob "siga assim mesmo")

- ❌ Confidence `alta` em dimensão validável só com Nível 4+ (EMQ real, CAPI server-side rate) sem flag correspondente
- ❌ "Generic tracking is bad" sem mapear pra dimensão+severidade+remediation
- ❌ Plano remediação sem priorização ROI (P0/P1/P2 + esforço-horas + impacto-quantificado)
- ❌ Severidade sem mapping pra artigo legal (LGPD Art. Nº) ou spec técnica (Meta CAPI spec / Google deprecation / Consent Mode v2)
- ❌ Output cliente-facing com jargão denso sem traduzir pra impacto de negócio
- ❌ Modo `ecommerce` rodado em LP sem cart/checkout — recusa, valida modo correto
- ❌ Nível 1 + operador exige EMQ exato — declara `confidence: media` em D5 + recomenda Nível 4
- ❌ Confiar só no monkey-patch JS sem `browser_network_requests` nativo Playwright (D-AUDITORES-5 — lição Aquatro `window.location.href` setter escapa)

## Cross-skill (handoffs canônicos)

**Downstream (consome auditoria):**

| Skill | Quando | O que extrai |
|---|---|---|
| `measurement-architect` (Fundação 1.3.2) | Cliente N1+ que já tem tracking parcial | Stack atual + gaps identificados + EMQ baseline + hash policy presente vs ausente — informa Plan/Contrato |
| `instrumentation-engineer` v2 (Fase 2) | Implementação técnica dos fixes P0-P2 | Plano de remediação P0-P2 + procedures técnicos detalhados |
| `tracking-engineer` família (Fase 2 × 7) | Config GTM + plataformas + Consent Mode v2 | Gaps Consent v2 + tags faltantes + LGPD remediation |
| `account-curator` (cross-fase) | Decisões arquiteturais detectadas viram Decision Doc | Decisões implícitas (ex: "consent default granted" é decisão a renegociar) |
| `measurement-auditor` (Fase 3 D36) | Trimestre seguinte — drift interno vs baseline black-box | Baseline de eventos canônicos detectados + EMQ baseline + plataformas instaladas |
| `taxonomy-builder` (Fundação 1.3.0) | Cliente N1+ sem taxonomia formal | Eventos canônicos detectados informam vocabulário core |
| Cliente (acima Linha Visibilidade) | Standalone comercial | MD cliente-facing OR HTML Reveal.js (`--render=reveal`) |

**Upstream (consumido por esta skill):**

| Input | Skill upstream OR origem | Função |
|---|---|---|
| URL pública | Operador | Alvo da auditoria |
| Flags Níveis 2-9 cond. | Operador (credenciais opt-in) | Sobe confidence dimensões correspondentes |
| `--measurement-plan=<path>.yml` cond. | `measurement-architect` ou operador | Auditoria híbrida (drift vs Plan declarado) |

## Princípios incidentes

- **P1** (Especialização — black-box estado-da-arte ortogonal a `measurement-auditor` drift interno + `seo-technical-auditor` discoverability + `account-health-auditor` saúde conta ads)
- **P5** (Camadas — auditoria é skill Nível 2; consome `playwright` MCP universal; produz output consumível por `measurement-architect` Nível 3+)
- **P8** (Vault Estado — auditoria é Categoria 2 Snapshot, fonte de verdade do estado tracking vigente)
- **P10** (Linha Visibilidade — modo Fundação/F3 = abaixo da linha (operador); modo standalone = acima da linha (cliente))
- **P11** (CODE — Capture estruturado Playwright + Organize 10 dimensões + Distill scoring rubric + Express output 12 seções)

---

## Decisões fechadas (canônicas — sessão dedicada 2026-05-11)

| ID | Decisão | Reabertura |
|---|---|---|
| **D-AUDITORES-1** | Skill única com modos (`lp`/`site`/`ecommerce`) ao invés de família flat | 2+ projetos pedirem isolamento por target |
| **D-AUDITORES-2** | Modo `lp` B2B validado piloto Aquatro 2026-05-11; modos `site`+`ecommerce` validados em piloto Bloco D real | Trigger: projetos N2+ multi-page OR ecommerce entrando Bloco D |
| **D-AUDITORES-3** | 6 refinamentos do piloto incorporados na rubrica (D8 banner cosmético, D4 hash não-trafega, D2 duplicação, D6 inferência server-side, D10 LCP via observer, D11 pipeline pós-LP) | Refinamentos adicionais em pilotos `site`/`ecommerce` |
| **D-AUDITORES-4** | Skill canônica vive em `cross-fase/` (cobre standalone + Fundação 1.3.2 + Fase 3 Loop trimestral) | — |
| **D-AUDITORES-5** | Procedures Playwright literais em `references/procedures-playwright-tracking.md` — pattern crítico "NUNCA confiar só no JS interceptor — `browser_network_requests` nativo como redundância obrigatória" | — |
| **D-AUDITORES-6** | Cutoff temporal explícito "estado-da-arte 2026" — atualização anual obrigatória via política B7 | Quando ano vira (2027) |
| **D-AUDITORES-7** | Confidence indicator embutido no schema (alta/media/baixa por dimensão) — flags 2-9 sobem confidence cross-dimensão | — |
| **D-AUDITORES-8** | Validação empírica via piloto real ANTES da auditoria fechar (convenção da linha) | — |

---

## Referências externas-âncora

- [Meta — Event Match Quality](https://www.facebook.com/business/help/2364892187272978) — Fórmula EMQ 2024-2026
- [Meta — Conversions API](https://developers.facebook.com/docs/marketing-api/conversions-api) — Spec dedup + event_id + server events
- [Google — Consent Mode v2](https://developers.google.com/tag-platform/security/guides/consent) — Spec mar/2024 + mudança junho/2026
- [LGPD — Lei 13.709/2018](http://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm) — Art. 5º/7º/8º/9º/18º
- [ANPD — Guia Cookies set/2023](https://www.gov.br/anpd/pt-br/documentos-e-publicacoes/guia-cookies.pdf)
- [ANPD — Mapa Temas Prioritários 2026-2027](https://www.gov.br/anpd/pt-br/assuntos/noticias/anpd-publica-mapa-de-temas-prioritarios-para-o-bienio-2026-2027)
- [Snowplow — Schema Versioning (SchemaVer)](https://docs.snowplow.io/docs/api-reference/iglu/common-architecture/schemaver/)
- [Avo — Tracking Principles](https://www.avo.app/principles) — Bad Data is Worse Than No Data
- [Google Lighthouse — Core Web Vitals](https://web.dev/articles/vitals) — LCP / INP / CLS thresholds
- [Playwright Network](https://playwright.dev/docs/network) — Network interception 2026
