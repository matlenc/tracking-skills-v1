# Tracking Skills v1 — Claude Code

Três skills para Claude Code que cobrem o ciclo completo de tracking/mensuração em projetos de mídia paga: configurar a **origem** do dado, configurar a **captura e distribuição** via GTM/N8N, e **auditar** infraestrutura de tracking de um alvo externo via Playwright.

> **Versão:** v1 (estável, em produção). Validadas em Manchester (WordPress) e Aquatro Suprimentos (V4 Pages / GreatPages).
> **Idioma:** pt-BR.
> **Stack:** Claude Code (CLI / desktop / extensão IDE).

---

## As 3 skills

### 1. `instrumentation-engineer` — origem do dado

Configura a **origem dos dados de forma padronizada antes de qualquer GTM/plataforma**. Constrói o measurement plan junto com o operador via 12 perguntas estruturadas, produz snippets `dataLayer.push()` por evento canônico, scripts de captura GCLID/FBCLID/wbraid/gbraid + persistência cookie first-party 90 dias, snippet User-ID hash SHA-256, payloads N8N e o documento `handoff-tracking-engineer.md` consumido pela skill `tracking-engineer`.

**Princípios canônicos:**
- Avo "bad data is worse than no data" — recusa instrumentar evento mal-modelado
- Schema versioning Snowplow SemVer no payload (`_schema_version: "2-1-0"`)
- Hash SHA-256 normalizado (lowercase email trimado, phone E.164 Google / dígitos puros Meta)
- Event_id UUID v4 client+server pra dedup CAPI

**3 modos:** `front` (snippets dataLayer) / `server-prep` (payload N8N) / `híbrido`.

**Ativar quando:** operador precisa preparar a origem de eventos de tracking de um projeto novo.
**Não ativar para:** configurar GTM/tags/variáveis (é escopo do `tracking-engineer`).

---

### 2. `tracking-engineer` — captura + distribuição via GTM

Configura **GTM Web + tags por plataforma + QA técnico** a partir do handoff da `instrumentation-engineer`. Produz JSON GTM importável (`exportFormatVersion: 2`) com integridade referencial, tags GA4/Meta/Google Ads/Clarity, checklist de QA com decisão formal go/go-com-risco/no-go/retestar (classificação bloqueador/alto/médio/baixo), EMQ ≥ 7,0 como meta de done pra CAPI, matriz de decisão Stape × Cloud Run × N8N para evolução server-side, Partial Export Simo Ahava para containers herdados.

**4 modos:** `platform-only` / `platform-crm` / `repair` (auditoria container herdado) / `evolucao` (server-side roadmap).

**Inclui:**
- Defense-in-depth Custom HTML (3 camadas: trigger filter + guard JS + allowlist)
- Captura cross-session GCLID/FBCLID/wbraid/gbraid/fbp/fbc
- Consent Default Tag (HTML, prioridade 100)
- UPD setup manual (Enhanced Conversions Google Ads)
- Auditoria de container herdado (checklist 6 itens)

**Ativar quando:** operador recebeu o handoff da IE e precisa configurar GTM + plataformas + QA.
**Não ativar para:** preparar a origem do evento (é escopo da `instrumentation-engineer`).

---

### 3. `tracking-blackbox-auditor` — diagnóstico externo via Playwright

Auditora **black-box estado-da-arte 2026** de infraestrutura de tracking em LP/site/ecommerce externo. Recebe URL pública e produz nota ponderada 0-10 + severidades canônicas mapeadas pra artigo LGPD (Art. 5º/7º/8º/9º/18º + ANPD Guia Cookies 2023 + Dark Patterns 2024) ou spec técnica (Meta CAPI / Google Consent Mode v2 / Snowplow SchemaVer / Avo) + plano de remediação priorizado por ROI + handoffs canônicos.

**3 modos:** `lp` (LP B2B/B2C standalone — default) / `site` (cliente N2+ multi-page) / `ecommerce` (loja com cart+checkout).

**10 dimensões canônicas + D11 opcional, com pesos por modo:**
1. Cobertura de eventos críticos
2. Qualidade do payload (taxonomia + nomenclatura)
3. Cobertura de páginas/jornada
4. Identity (hash + User-ID strategy)
5. EMQ Meta (formula canônica 2-10)
6. Dedup CAPI (event_id UUID v4)
7. Click IDs (gclid/fbclid/wbraid/gbraid/msclkid/ttclid/li_fat_id)
8. Consent Mode v2 + LGPD compliance
9. Enhanced Ecommerce (modo `ecommerce`)
10. Performance (LCP via PerformanceObserver)
11. Pipeline pós-LP (opcional informativa)

**9 níveis de acesso opt-in (frictionless → server logs):**
- Nível 1 (default) — só URL via Playwright, baseline 70-80% cobertura
- Níveis 2-9 — GTM container / GA4 Admin / Meta Business / Google Ads / LinkedIn / TikTok / Clarity / CRM / server logs (cada flag sobe `confidence: alta` na dimensão correspondente)

**Pré-requisitos:**
- **Playwright MCP instalado** (universal P1) — skill recusa rodar sem
- **URL alvo** pública retornando 200 OK

**Caso de uso comercial dominante:** porta de entrada cliente novo (`--modo=lp` Nível 1 default). Vendável R$ 1.500-5.000 OU bundle no kickoff premium. Cliente vê gap concreto antes de aprovar Measurement Plan.

**Ativar quando:** auditar tracking de site externo (próprio ou de prospect).
**Não ativar para:** configurar tracking do zero (é escopo da IE + tracking-engineer).

---

## Fluxo recomendado

**Cenário 1 — Projeto novo (do zero):**
```
instrumentation-engineer → tracking-engineer
   (origem do dado)        (GTM + plataformas + QA)
```

**Cenário 2 — Projeto com tracking pré-existente:**
```
tracking-blackbox-auditor → instrumentation-engineer → tracking-engineer
   (diagnóstico inicial)        (rebaseline origem)       (refator GTM)
```

**Cenário 3 — Diagnóstico standalone (sem implementação):**
```
tracking-blackbox-auditor (modo lp/site/ecommerce)
   → entrega auditoria + plano de remediação ROI-priorizado
```

---

## Instalação

Skills do Claude Code ficam em `~/.claude/skills/`. Pra instalar:

```bash
# Clone o repo
git clone <URL-do-repo> tracking-skills-v1
cd tracking-skills-v1

# Copia as 3 skills pra ~/.claude/skills/
cp -r instrumentation-engineer ~/.claude/skills/
cp -r tracking-engineer ~/.claude/skills/
cp -r tracking-blackbox-auditor ~/.claude/skills/
```

**No Windows (PowerShell):**
```powershell
Copy-Item -Recurse instrumentation-engineer $HOME\.claude\skills\
Copy-Item -Recurse tracking-engineer $HOME\.claude\skills\
Copy-Item -Recurse tracking-blackbox-auditor $HOME\.claude\skills\
```

Depois reabra o Claude Code (CLI / desktop / VS Code) — as skills aparecem disponíveis automaticamente.

---

## Pré-requisitos por skill

| Skill | MCPs / dependências | Observação |
|---|---|---|
| `instrumentation-engineer` | Read, Write (nativos) | Nenhuma dependência externa |
| `tracking-engineer` | Read, Write (nativos) | Operador faz import manual no GTM (sem MCP GTM oficial) |
| `tracking-blackbox-auditor` | **Playwright MCP** (obrigatório) | Skill recusa rodar sem. Instala via `npx @anthropic/mcp-server-playwright` ou config no `.claude/settings.json` |

---

## Como invocar

No Claude Code, basta descrever a tarefa — a skill ativa sozinha pela `description` do frontmatter. Exemplos:

```
"preciso preparar o tracking do projeto novo da Loja X — começa pelo instrumentation"
"recebi o handoff da IE, vamos configurar o GTM"
"audita o tracking dessa LP: https://exemplo.com.br/oferta"
```

Você também pode invocar explicitamente:
```
"usa a skill tracking-blackbox-auditor no modo lp pra https://exemplo.com.br"
```

---

## Validação empírica (v1)

- **Manchester** (WordPress) — `instrumentation-engineer` + `tracking-engineer` em projeto vivo.
- **Aquatro Suprimentos** (V4 Pages / GreatPages) — fluxo completo: `tracking-blackbox-auditor` → diagnóstico → `instrumentation-engineer` rebaseline → `tracking-engineer` configuração. Auditoria do Aquatro foi a primeira validação empírica da `tracking-blackbox-auditor` (12 min execução, 75 network requests inspecionadas, 6 refinamentos da rubrica detectados e incorporados).

---

## Roadmap v2 (em produção — não inclusa neste repo)

A v2 fragmenta melhor o processo:

- `measurement-architect` + `taxonomy-builder` — par da Fundação que **co-produz Measurement Plan + Contrato de Dados + Taxonomia canônica** (esse escopo sai da IE v1)
- `instrumentation-engineer` v2 — só gera scripts da origem **consumindo o Plan** (perde construção do Plan)
- `tracking-setup-gtm` — base GTM Web (pré-requisito compartilhado)
- 6 skills `tracking-*` por destino — `tracking-ga4` / `tracking-meta-ads` / `tracking-google-ads` / `tracking-linkedin-ads` / `tracking-tiktok-ads` / `tracking-clarity`
- `n8n-event-pipeline-builder` — owna workflows N8N de eventos (hoje moram dentro da IE v1)

A v2 só faz sentido em projetos com Fundação Growth IA Ops v2.0 rodada antes. Em projetos sem essa Fundação, **a v1 deste repo continua sendo a recomendação**.

---

## Estrutura do repo

```
tracking-skills-v1/
├── README.md (este arquivo)
├── instrumentation-engineer/
│   ├── SKILL.md
│   └── referencias/
│       ├── sop_tracking_instrumentacao.md
│       └── taxonomia_tracking_instrumentacao.md
├── tracking-engineer/
│   ├── SKILL.md
│   ├── guias/
│   │   ├── checklist-auditoria-container-herdado.md
│   │   ├── gtm-config-reference.md
│   │   └── playbook-validacao-e2e-server-side.md
│   ├── referencias/
│   │   ├── sop_tracking_instrumentacao.md
│   │   └── taxonomia_tracking_instrumentacao.md
│   └── templates/
│       ├── capi-forwarder-tag.html
│       └── n8n-workflow-meta-capi.json
└── tracking-blackbox-auditor/
    ├── SKILL.md
    ├── references/  (10 references densos — rubrica + procedures Playwright + EMQ + Consent Mode v2 + LGPD + Click IDs + Enhanced Ecommerce + severidades + confidence indicator + plano remediação)
    ├── schemas/
    │   └── auditoria-tracking-manifest.yml
    └── templates/
        ├── auditoria-tracking-template.md (operador-facing)
        ├── auditoria-tracking-cliente-template.md (cliente-facing standalone)
        └── auditoria-tracking-cliente-template.html (Reveal.js render via `--render=reveal`)
```

---

## Licença

Uso interno. Para uso comercial externo ou redistribuição, consulte o autor.
