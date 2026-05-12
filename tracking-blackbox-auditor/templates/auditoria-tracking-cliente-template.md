---
title: Diagnóstico de Tracking — {{cliente}}
type: external
diataxis: reference
schema_id: auditoria-tracking-blackbox-cliente
schema_version: 1.0.0
versao: 1.0.0
data: "{{YYYY-MM-DD}}"
operador: "{{nome_operador}}"
cliente: "{{nome_cliente}}"
status: aprovado
target_url: "{{URL auditada}}"
target_slug: "{{slug}}"
modo: "{{lp | site | ecommerce}}"
modelo_negocio: "{{b2b-leadgen | b2c-ticket-alto | b2c-consumer | ecommerce | plg-saas}}"
nivel_acesso_max: "{{1-9}}"
nota_final: "{{0-10}}"
categoria: "{{estado-da-arte | maduro | funcional | subdesenvolvido | inexistente}}"
hard_blockers_count: "{{integer}}"
formato_output_principal: MD cliente-facing + manifest YAML
---

# Diagnóstico de Tracking — {{cliente}}

> Auditoria técnica e jurídica da infraestrutura de mensuração de **{{cliente}}** ({{target_url}}) — {{data}}.
> Avalia 10 dimensões canônicas estado-da-arte 2026 + LGPD compliance + ROI de remediação.

---

## Sumário Executivo

**Estado atual:** Nota {{nota_final}}/10 — **{{categoria}}**

### O que isso significa pro seu negócio

{{1 parágrafo cliente-facing traduzindo nota+categoria pra impacto de negócio:}}

- Nota 9-10 (Estado-da-arte) → "Seu tracking está entre os 5% mais maduros do mercado BR. Refinamentos finos restantes — manter benchmark."
- Nota 7-8 (Maduro) → "Setup robusto com ≤2 gaps técnicos. Você está acima da média do mercado BR; remediação pontual destrava o último estágio."
- Nota 5-6 (Funcional) → "Tracking funciona mas tem gaps significativos. Atribuição está comprometida — está perdendo signal pras plataformas otimizarem suas campanhas."
- Nota 3-4 (Subdesenvolvido) → "Tracking deficiente com múltiplos blockers — sua atribuição não está confiável. Recuperação significativa de eficiência de mídia disponível."
- Nota 0-2 (Inexistente) → "Tracking quase ausente. Cada Real investido em mídia está sendo otimizado às cegas. Greenfield prático mesmo com algo instalado."

### 3 destaques desta auditoria

🟢 **O que tá bom** — {{achado_positivo_traduzido_negocio}}

🔴 **O que precisa atenção imediata** — {{achado_critico_traduzido_negocio}}

🟡 **Onde tem oportunidade** — {{achado_atenção_traduzido_negocio}}

### Investimento de remediação

| Tier | O que cobre | Esforço estimado |
|------|-------------|------------------|
| **P0 — Urgente (7 dias)** | {{N}} blockers legais e técnicos críticos | {{X-Y horas}} |
| **P1 — Sprint-1 (30 dias)** | {{N}} ganhos de ROI alto | {{X-Y horas}} |
| **P2 — Sprint-2+ (60-90 dias)** | {{N}} refinamentos incrementais | {{X-Y horas}} |
| **Total** | {{N}} itens priorizados | **{{X-Y horas}}** |

**ROI esperado pós-remediação:** {{X-Y%}} em atribuição Meta + {{X-Y%}} em Enhanced Conversions Google Ads + LGPD compliance assegurada.

---

## 1. O Que Foi Avaliado

### 1.1 Como funciona a auditoria

Esta auditoria é **black-box** — observamos sua LP/site externamente via automação Playwright (sem credenciais privadas necessárias) e validamos contra **rubrica universal estado-da-arte 2026** em 10 dimensões canônicas.

**Não tocamos em nada do seu site** — só observamos, medimos, diagnosticamos.

### 1.2 As 10 dimensões avaliadas

| # | Dimensão | O que mede |
|---|----------|------------|
| D1 | Stack instalada | Quais plataformas de tracking estão presentes (GA4 / GTM / Meta / Google Ads / LinkedIn / TikTok / etc.) vs benchmark do seu modelo de negócio |
| D2 | Qualidade do DataLayer | Como os eventos estão modelados — schema versioning, naming convention, parâmetros enriquecidos |
| D3 | Cobertura da jornada | Quais ações do usuário viram eventos trackados (cliques, formulários, scroll, vídeo, outbound, phone, WhatsApp) |
| D4 | Identidade e Hash (LGPD-aware) | Como dados pessoais (email, telefone) são hashados antes de irem pra Meta/Google — pré-requisito de matching |
| D5 | EMQ Meta | Event Match Quality que Meta usa pra avaliar atribuição. Target ≥7.0 (`Good`) |
| D6 | Dedup CAPI | Server-side tracking via Conversions API com deduplicação correta — recovery 30-50% atribuição pós-iOS 14 |
| D7 | Click IDs | Captura e persistência de identificadores de clique (gclid/fbclid/wbraid/gbraid/msclkid/ttclid/li_fat_id) |
| D8 | Consent Mode v2 + LGPD | Banner de consent + Consent Mode v2 + conformidade LGPD (Art. 5º/7º/8º/9º/18º + ANPD guias) |
| D9 | Enhanced Ecommerce GA4 | 11 eventos canônicos GA4 cobertos (só ecommerce) |
| D10 | Tracking health + performance | Core Web Vitals + console errors + race conditions + cross-browser parity |

---

## 2. Resultados Detalhados

### 2.1 Pontuação por dimensão

| Dimensão | Score | Status |
|----------|-------|--------|
| D1 Stack instalada | {{score}}/10 | {{🟢 Verde / 🟡 Amarelo / 🔴 Vermelho}} |
| D2 DataLayer schema | {{score}}/10 | {{...}} |
| D3 Cobertura jornada | {{score}}/10 | {{...}} |
| D4 Identidade & Hash | {{score}}/10 | {{...}} |
| D5 EMQ Meta | {{score}}/10 | {{...}} |
| D6 Dedup CAPI | {{score}}/10 | {{...}} |
| D7 Click IDs | {{score}}/10 | {{...}} |
| D8 Consent + LGPD | {{score}}/10 | {{...}} |
| D9 Enhanced Ecommerce | {{score/—}}/10 | {{...}} |
| D10 Tracking health | {{score}}/10 | {{...}} |
| **Total Ponderado** | **{{nota_final}}/10** | **{{categoria}}** |

### 2.2 O que cada nota quer dizer (traduzido pro negócio)

- **Score 9-10 (Verde)** — Estado-da-arte. Mantenha o setup. Use como benchmark interno.
- **Score 7-8 (Verde Claro)** — Maduro. Pequenos refinamentos restantes.
- **Score 5-6 (Amarelo)** — Funcional mas com gaps. Remediação pontual destrava ganhos.
- **Score 3-4 (Laranja)** — Subdesenvolvido. Múltiplas pontes a fortalecer. ROI alto de remediação.
- **Score 0-2 (Vermelho)** — Greenfield prático. Setup inteiro a construir.

---

## 3. Riscos LGPD Identificados

> A LGPD (Lei 13.709/2018) define multa até **2% do faturamento, máximo R$ 50 milhões por violação**. A ANPD (Autoridade Nacional de Proteção de Dados) foi transformada em agência reguladora com poderes ampliados em 2025 — fiscalização ativa começou em 2026.

### 3.1 Blockers LGPD detectados

{{Listar cond. — só se houver}}

| Achado | Severidade | Artigo LGPD | Risco Real |
|--------|-----------|-------------|------------|
| {{achado-1}} | 🔴 BLOCKER | Art. {{Nº}} | Multa + reputational + correção obrigatória ANPD |
| {{achado-2}} | 🔴 BLOCKER | Art. {{Nº}} | {{risco}} |
| ... | ... | ... | ... |

### 3.2 Conformidade com Mapa Temas Prioritários ANPD 2026-2027

ANPD publicou 4 prioridades pra fiscalização 2026-2027:
1. **Direitos dos titulares de dados** (Art. 18) — {{status do seu setup}}
2. Proteção de crianças e adolescentes em ambientes digitais
3. Tratamento de dados pessoais pelo Poder Público
4. Inteligência artificial e tecnologias emergentes

**Risco no seu caso:** {{análise alta/média/baixa baseada no setup}}

---

## 4. Plano de Ação Priorizado

### 4.1 P0 — Urgente (Próximos 7 dias)

> Blockers legais (risco multa LGPD) e técnicos críticos (UA deprecated, dedup quebrada, purchase incompleto).

| # | O que fazer | Por quê | Esforço | Quem faz |
|---|-------------|---------|---------|----------|
| P0.1 | {{Ação traduzida pra cliente}} | {{razão simples}} | {{X-Y h}} | Equipe técnica de tracking |
| P0.2 | {{Ação}} | {{razão}} | {{X-Y h}} | {{quem}} |
| ... | ... | ... | ... | ... |

### 4.2 P1 — Sprint-1 (Próximos 30 dias)

> Alto ROI técnico — EMQ baixo, hash não trafega, click IDs ausentes, eventos críticos faltantes.

| # | O que fazer | Impacto esperado | Esforço |
|---|-------------|------------------|---------|
| P1.1 | {{Ação}} | {{Impacto quantificado}} | {{X-Y h}} |
| P1.2 | {{Ação}} | {{Impacto quantificado}} | {{X-Y h}} |
| ... | ... | ... | ... |

### 4.3 P2 — Sprint-2+ (Próximos 60-90 dias)

> Refinamentos incrementais — schema versioning, Safari ITP mitigation, custom dimensions.

| # | O que fazer | Impacto | Esforço |
|---|-------------|---------|---------|
| P2.1 | {{Ação}} | {{Impacto}} | {{X-Y h}} |
| ... | ... | ... | ... |

---

## 5. Por Onde Começar

### 5.1 Recomendações imediatas

1. **{{Top 1 ação imediata canônica}}** — {{razão de negócio}}
2. **{{Top 2 ação imediata}}** — {{razão}}
3. **{{Top 3 ação imediata}}** — {{razão}}

### 5.2 Como executar

Esta auditoria pode informar próximos passos em 3 caminhos:

- **Caminho 1 — Equipe interna técnica:** seu time de marketing/tech executa P0 (semana-1) + P1 (sprint-1) seguindo o plano detalhado deste documento.
- **Caminho 2 — Bundle V4 Growth IA Ops:** fundação completa (mensuração + LGPD + estrutura RevOps) construída por especialistas em ~90 dias. Auditoria atual informa Measurement Plan + Contrato de Dados.
- **Caminho 3 — Sprint pontual:** remediação focada em P0 + P1 críticos por equipe especialista em ~30 dias.

---

## 6. Próximos Passos

### 6.1 Reunião de discussão (recomendado)

Sugerimos reunião 60 min cobrindo:
- Validação dos achados detalhados
- Priorização cliente-side baseada em contexto comercial
- Definição do caminho de execução (interno vs externo)
- Cronograma + responsabilidades

### 6.2 Documentos relacionados

- **Política de Privacidade** atual da empresa — para validar atualizações pós-remediação LGPD
- **Setup do GTM / Google Analytics 4** — para auditoria Nível 2-3 com upgrade de confidence
- **Acesso Meta Business Manager + Google Ads** — para auditoria Nível 4-5 com confirmação EMQ real

---

## 7. Limitações desta Auditoria

Esta auditoria foi executada em **Nível 1 (só URL)** — modo frictionless sem credenciais privadas. Algumas dimensões têm confidence parcial:

| Dimensão | Confidence atual | Confidence atingível com upgrade |
|----------|------------------|----------------------------------|
| D5 EMQ Meta estimado | Média (margem ±1 ponto) | Alta (Nível 4 — Meta Business Manager) |
| D6 Server-side detection | Média (heurísticas) | Alta (Nível 4 ou 9 — server logs) |
| D2 DataLayer schema | Limitada (só jornada testada) | Alta (Nível 2 — GTM container export) |
| D9 Purchase event (ecommerce) | Não testado (skill não-destrutiva) | Alta (Nível 3 — GA4 Admin) |

**Upgrade pra Nível 2-9 disponível mediante acesso opt-in** (operador-providas credenciais opcionais).

---

## 8. Glossário Técnico (cliente-facing)

| Termo | Significado simples |
|-------|---------------------|
| **EMQ (Event Match Quality)** | Score 0-10 que Meta atribui à qualidade da atribuição de cada conversão. Target ≥7.0 = "Good". |
| **CAPI (Conversions API)** | API server-side do Meta — paralela ao Pixel browser. Recovery 30-50% atribuição pós-iOS 14. |
| **Consent Mode v2** | Framework Google obrigatório desde mar/2024 — comunicação status de consent às tags Google. |
| **LGPD (Lei 13.709/2018)** | Lei brasileira de proteção de dados pessoais — equivalente ao GDPR europeu. |
| **ANPD** | Autoridade Nacional de Proteção de Dados — agência reguladora desde 2025. |
| **gclid / fbclid** | Click IDs do Google Ads / Meta — identificadores únicos por click usados pra attribution. |
| **Hash SHA-256** | Função criptográfica que converte dados pessoais (email, phone) em string irreversível — pré-requisito de matching plataformas. |
| **event_id (UUID v4)** | Identificador único de cada evento — pré-requisito de deduplicação client↔server. |
| **GTM Server-side (SGTM)** | Container Google Tag Manager servido do seu domínio (1st-party) — bypassa Safari ITP + iOS 14 ATT. |
| **Universal Analytics (UA)** | Versão deprecated do Google Analytics (descontinuada jul/2023). Substituída por GA4. |

---

> Auditoria realizada por **{{nome_operador}}** via **`tracking-blackbox-auditor` v1.0.0** — rubrica universal canônica estado-da-arte 2026.
> Tese Growth IA Ops v2.0 — V4 Colli&Co.
