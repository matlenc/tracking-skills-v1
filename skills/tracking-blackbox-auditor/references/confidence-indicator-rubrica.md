# Confidence Indicator — rubrica por dimensão × nível de acesso

> **Reference da skill `tracking-blackbox-auditor`** — confidence indicator (alta/media/baixa) por dimensão × por nível de acesso 1-9. Quando confidence sobe pra `alta`. Threshold canônico pra upgrade.
>
> **Princípio Avo aplicado:** confidence indicator sinaliza quando evidência é insuficiente. Não inflar confidence pra parecer mais robusto — anti-pattern §8.3 da skill canônica.

---

## 1. Definições

| Confidence | Definição operacional |
|---|---|
| **`alta`** | Dimensão validável **100% client-side** (Playwright captura evidência direta e completa); OR Nível 2+ disponível confirma diretamente |
| **`media`** | Dimensão **estimada** client-side mas confirmação requer API/server access; Nível 1 entrega aproximação canônica com margem documentada |
| **`baixa`** | Cobertura **limitada** client-side; só vê o que disparou (não vê config); só vê jornada testada (não vê cenários não-cobertos) |

---

## 2. Matrix confidence × dimensão × nível

| Dimensão | Nível 1 (só URL) | Nível 2 (GTM) | Nível 3 (GA4 Admin) | Nível 4 (Meta) | Nível 5 (Google Ads) | Nível 6 (LinkedIn) | Nível 7 (TikTok) | Nível 8 (Clarity) | Nível 9 (Server) |
|---|---|---|---|---|---|---|---|---|---|
| **D1** Stack instalada | `alta` | `alta` | = | = | = | = | = | = | = |
| **D2** DataLayer schema | `baixa` | `alta` | = | = | = | = | = | = | = |
| **D3** Cobertura jornada | `baixa` | `media` | `alta` | = | = | = | = | = | = |
| **D4** Identity & Hash | `media` | `media` | `media` | `alta` | `alta` | = | = | = | = |
| **D5** EMQ Meta estimado | `media` | `media` | `media` | **`alta`** | = | = | = | = | = |
| **D6** Dedup CAPI prontidão | `media` | `media` | `media` | `alta` | = | = | = | = | `alta` |
| **D7** Click IDs capture | `alta` | `alta` | `alta` | `alta` | `alta` | `alta` | `alta` | = | = |
| **D8** Consent Mode v2 + LGPD | `alta` | `alta` | `alta` | = | = | = | = | = | = |
| **D9** Enhanced Ecommerce GA4 | `media` | `media` | `alta` | = | = | = | = | = | = |
| **D10** Tracking health + CWV | `alta` | `alta` | = | = | = | = | = | `alta` | = |
| **D11** Pipeline pós-LP (informativa) | `media` | `media` | = | = | = | = | = | = | `alta` |

Convenção: `=` significa "mantém confidence do nível anterior" (sem incremento).

---

## 3. Justificativa por dimensão

### 3.1 D1 — Stack instalada

**Nível 1 = `alta`** — Playwright captura DOM + globals + network requests com 100% precisão. Plataformas canônicas detectáveis via regex/globals são determinísticas.

**Nível 2-9 não incrementa** — stack instalada é observable client-side.

### 3.2 D2 — DataLayer schema quality

**Nível 1 = `baixa`** — só vê pushes que dispararam DURANTE a jornada testada. Não vê:
- Eventos disparados em outras páginas/jornadas não-testadas
- Schema variables config no GTM
- Tags pausadas/ocultas no GTM

**Nível 2 (GTM Container) = `alta`** — vê config completa (variables, tags pausadas, workspace state, versioning).

### 3.3 D3 — Cobertura jornada

**Nível 1 = `baixa`** — só vê jornada que Playwright executou. Não vê:
- Eventos esperados em outras páginas (modo `site` multi-page)
- Conversion events que requerem state não-reproduzível (logged in user, etc.)

**Nível 2 (GTM) = `media`** — vê triggers + tags config.
**Nível 3 (GA4 Admin) = `alta`** — vê Enhanced Measurement state + custom events config + conversions marked.

### 3.4 D4 — Identity & Hash Policy

**Nível 1 = `media`** — Playwright vê payload Meta `/tr` browser-side. Não vê:
- Hash setado server-side via CAPI (cliente pode ter hash correto server-only)
- Identity resolution cross-platform completa (só vê browser payloads)

**Nível 4 (Meta Business) = `alta`** — Meta Events Manager confirma hash recebido + Advanced Matching coverage real.

### 3.5 D5 — EMQ Meta estimado

**Nível 1 = `media`** — Estimativa via fórmula canônica com margem ±1 ponto vs valor real. Limitações:
- Não vê parâmetros que Meta hasheia internamente após Pixel browser
- Não vê AEM priority
- Não vê eventos rejeitados pelo Meta (data quality issues)

**Nível 4 (Meta Business) = `alta`** — Events Manager → Diagnostics tab confirma EMQ real última 48h.

### 3.6 D6 — Dedup CAPI prontidão

**Nível 1 = `media`** — Heurísticas inferência server-side (refinamento Aquatro) detectam Stape signature / Cloud Run / N8N webhook patterns. Limitações:
- Não confirma dedup rate real
- Não vê event_id matching browser↔server

**Nível 4 (Meta Business) = `alta`** — Events Manager confirma deduplication rate real + server events errors.

**Nível 9 (Server logs) = `alta`** — Logs Stape/Cloud Run confirmam CAPI gateway flow end-to-end.

### 3.7 D7 — Click IDs capture & persistence

**Nível 1 = `alta`** — Test sintético com query params + inspeção cookies/localStorage é determinístico. Playwright Webkit (Safari) testa ITP resilience.

**Nível 2-9 não incrementa** — captura client-side é observable.

### 3.8 D8 — Consent Mode v2 + LGPD

**Nível 1 = `alta`** — Validação empírica via Playwright (navegar sem aceitar / com aceitar) é determinística. Banner cosmético detectável.

**Nível 2-9 não incrementa** — comportamento client-side é observable.

### 3.9 D9 — Enhanced Ecommerce GA4

**Nível 1 = `media`** — Captura eventos disparados na jornada simulada (browse → ATC → checkout, sem completar purchase). Limitações:
- Não vê `refund` event (server-side typically)
- Não vê `purchase` real (skill não-destrutiva)
- Não vê conversion config GA4 Admin

**Nível 3 (GA4 Admin) = `alta`** — vê conversion events config + items array validation real + Enhanced Ecommerce state.

### 3.10 D10 — Tracking health + performance

**Nível 1 = `alta`** — PerformanceObserver via Playwright captura LCP/INP/CLS determinístico. Console errors + network failures observable.

**Nível 8 (Clarity) = `alta`** — Reforça com qualitative data (heatmaps, rage clicks).

### 3.11 D11 — Pipeline pós-LP (informativa)

**Nível 1 = `media`** — Detecta URL destino do submit + infere pipeline via padrões BR canônicos. Não confirma:
- N8N flow interno
- CRM ingestion real

**Nível 9 (Server logs / CDP) = `alta`** — Logs confirmam pipeline end-to-end.

---

## 4. Confidence global do output

| Confidence global | Critério |
|---|---|
| `alta` | ≥80% das dimensões com confidence `alta` (típico Nível 4+) |
| `media` | 50-79% dimensões `alta` + restantes `media` (típico Nível 1-3) |
| `baixa` | <50% dimensões `alta` OR ≥2 dimensões `baixa` (típico cliente N0 sem GTM acessível) |

---

## 5. Anti-patterns (não fazer)

| Anti-pattern | Por quê |
|---|---|
| Confidence `alta` em D5 EMQ Nível 1 | EMQ Nível 1 é **estimativa** — confidence sempre `media` |
| Confidence `alta` em D6 server-side sem Nível 4 ou 9 | Heurísticas inferem, não confirmam |
| Confidence `alta` em D9 Enhanced Ecommerce sem testar purchase real | Skill não-destrutiva, confidence `media` no purchase value |
| Não declarar confidence (output incompleto) | Cada dimensão obrigatoriamente tem confidence (8.1 critérios done) |
| Inflar confidence pra parecer mais robusto | Anti-pattern Avo — bad data is worse than no data |

---

## 6. Como sinalizar no output

Cada linha da §3 "Pontuação por Dimensão" inclui coluna `confidence`:

```markdown
| # | Dimensão | Peso | Score 0-10 | Ponderado | Confidence | Justificativa |
|---|----------|------|-----------|-----------|------------|---------------|
| D1 | Stack instalada vs benchmark | 12% | 7.5 | 0.90 | **alta** | Playwright detectou GA4 + GTM + Meta + RD Station |
| D2 | DataLayer schema quality | 12% | 4.5 | 0.54 | **baixa** | Só vista a jornada testada — `--gtm-export` recomendado pra subir pra `alta` |
| D5 | EMQ Meta estimado | 15% | 2.5 | 0.38 | **media** | Estimativa via fórmula — Nível 4 (Meta Business) recomendado pra confirmação |
| ... | ... | ... | ... | ... | ... | ... |
```

E §8 "Confidence Indicator Global":

```markdown
## 8. Confidence Indicator Global

| Dimensão | Confidence | Razão | Upgrade recomendado |
|---|---|---|---|
| D1 | alta | 100% validável client-side | — |
| D2 | baixa | Só jornada testada — não vê config GTM | `--gtm-export=<path>.json` (Nível 2) |
| D5 | media | Estimativa fórmula EMQ — margem ±1 ponto vs Events Manager real | `--meta-business-token=<token>` (Nível 4) |
| D6 | media | Heurísticas server-side inferem mas não confirmam dedup rate | `--meta-business-token` (Nível 4) OR `--server-logs-path` (Nível 9) |
| D9 | media | Skill não-destrutiva — não completa purchase real | `--ga4-credentials` (Nível 3) confirma conversion events config |
| ...resto |
| **Global** | **media** | 5/10 dimensões `alta` + 4 `media` + 1 `baixa` | Nível 2-4 recomendado pra confidence global `alta` |
```

---

## 7. References cross-skill

- [`rubrica-tracking-blackbox.md`](rubrica-tracking-blackbox.md) — 10 dimensões + scoring rubric
- [`procedures-playwright-tracking.md`](procedures-playwright-tracking.md) — procedures por dimensão
- SKILL.md §"Flags de acesso opt-in" — descrição completa Níveis 1-9
