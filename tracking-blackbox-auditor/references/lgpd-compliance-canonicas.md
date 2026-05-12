# LGPD Compliance — artigos canônicos + ANPD guias + severidades 2026

> **Reference da skill `tracking-blackbox-auditor`** — LGPD Lei 13.709/2018 + ANPD Guia Cookies set/2023 + ANPD Guia Dark Patterns 2024 + ANPD Mapa Temas Prioritários 2026-2027 (fiscalização ativa) + mapeamento blockers severidade → artigo violado.

---

## 1. LGPD — artigos canônicos pra tracking

### 1.1 Art. 5º — Definições (escopo de "dados pessoais")

Define **dados pessoais** como informação que identifica ou torna identificável pessoa natural. Inclui:
- Email, telefone, nome (PII direto)
- IP address + cookies (PII indireto se cross-referenciável)
- Click IDs persistidos (`gclid`, `fbclid` se associados a user identity)
- Identifiers cross-platform (`user_id`, `external_id`)

**Implicação tracking:** TODO payload Meta/Google/LinkedIn com email/phone/IP é processamento de dados pessoais → exige base legal (Art. 7º).

### 1.2 Art. 7º — Bases legais pra tratamento

10 bases legais. Pra tracking marketing, a única operacional é **inciso I — Consentimento livre, informado e inequívoco**.

**Implicação:** sem consent explícito = sem base legal = BLOCKER LGPD.

### 1.3 Art. 8º — Características do consentimento

Consentimento deve ser:
- **Livre** — sem coação, sem pré-marcação, sem bundling com outros serviços
- **Informado** — usuário sabe exatamente o quê + por quê + finalidade
- **Inequívoco** — sem dúvida sobre aceite (opt-in explícito, não opt-out)

**Anti-pattern canônico (BLOCKER):**
- Banner com botão único "OK / Entendi" (sem opção rejeitar) = NÃO inequívoco
- Default consent state = `granted` sem clique = NÃO livre nem inequívoco
- Botão "Aceitar" gigante + "Rejeitar" micro = NÃO livre (coação visual = dark pattern)

### 1.4 Art. 9º — Direito à informação

Titular dos dados tem direito à informação clara sobre:
- Finalidade do tratamento
- Forma e duração
- Identificação do controlador (empresa)
- Compartilhamentos com terceiros (Meta, Google, etc.)

**Implicação tracking:** Política de Privacidade linkada no banner + identificação clara de qual data é compartilhada com Meta/Google/LinkedIn/etc.

### 1.5 Art. 18º — Direitos do titular

9 direitos canônicos do titular:
1. Confirmação da existência de tratamento
2. Acesso aos dados
3. Correção de dados incompletos/inexatos
4. Anonimização, bloqueio ou eliminação de dados desnecessários
5. Portabilidade
6. Eliminação de dados tratados com consentimento
7. Informação sobre compartilhamentos com entidades públicas/privadas
8. Informação sobre possibilidade de não dar consentimento + consequências
9. Revogação do consentimento

**Implicação tracking:** Política de Privacidade DEVE listar os 9 direitos + canal de exercício (email DPO ou formulário). Ausência = BLOCKER LGPD.

---

## 2. ANPD — guias canônicos 2023-2026

### 2.1 Guia Orientativo de Cookies (set/2023)

Padrão regulatório BR pra banners de cookie. Pontos canônicos:

- **Política de Cookies separada** da Política de Privacidade (recomendado)
- **Checkbox específico** pra rejeitar cookies não-estritamente-necessários
- **Banner em primeiro nível** (não escondido em segundo nível)
- **Granularidade obrigatória** — categorias: estritamente necessários / funcionais / analytics / marketing
- **Default state denied** pra categorias não-essenciais
- **Botão "Rejeitar"** com mesmo destaque visual de "Aceitar"

### 2.2 Guia Dark Patterns (2024)

ANPD publicou guia detalhando dark patterns proibidos sob LGPD:

| Dark pattern | Status ANPD |
|---|---|
| Botão "Aceitar" destacado vs "Rejeitar" micro/escondido | ⛔ Proibido |
| Banner cosmético (presente mas não funcional) | ⛔ Proibido |
| "Aceitar todos" como única opção primária | ⛔ Proibido |
| Pre-marcação de checkboxes de consent | ⛔ Proibido |
| "Configurar" escondido em 2º nível com fricção alta | ⛔ Proibido |
| Linguagem confusa/ambígua sobre finalidade | ⛔ Proibido |
| Loops infinitos pra rejeitar (re-aparição forçada) | ⛔ Proibido |

### 2.3 Mapa Temas Prioritários 2026-2027 (publicado dez/2025)

ANPD publicou 4 prioridades pra fiscalização 2026-2027:

1. **Direitos dos titulares de dados** (Art. 18)
2. **Proteção de crianças e adolescentes em ambientes digitais**
3. **Tratamento de dados pessoais pelo Poder Público**
4. **Inteligência artificial e tecnologias emergentes em processamento de dados pessoais**

**Implicação:** auditoria tracking 2026 deve sinalizar **risco aumentado de fiscalização** em casos de violação Art. 18 (direitos do titular não informados).

---

## 3. Sanções LGPD (Art. 52)

Sanções administrativas aplicáveis:

| Sanção | Quando |
|---|---|
| **Advertência** com prazo pra correção | Primeira violação não-grave |
| **Multa simples até 2% do faturamento** (cap R$ 50M/violação) | Violações graves OR recorrência |
| **Multa diária** | Descumprimento de obrigação |
| **Publicização da infração** | Após confirmada |
| **Bloqueio ou eliminação dos dados** | Casos graves |
| **Proibição parcial/total** de atividades de tratamento | Violação contínua |
| **Suspensão de banco de dados** (cond.) | Risco grave |

**Multa máxima por violação:** R$ 50.000.000 OR 2% do faturamento (o que for menor).

**2025-2026:** ANPD transformada em **agência reguladora com poderes ampliados** — fiscalização ativa (não mais apenas orientativa).

---

## 4. Mapeamento severidades canônicas → artigo LGPD

| Achado da auditoria | Severidade | Artigo violado | Spec ANPD |
|---|---|---|---|
| PII em claro em payload Meta/Google | **BLOCKER LGPD** | Art. 5º + Art. 7º | — |
| Banner consent ausente | **BLOCKER LGPD** | Art. 7º | Guia Cookies set/2023 |
| Default consent state = `granted` sem aceite | **BLOCKER LGPD** | Art. 7º + Art. 8º | Guia Cookies set/2023 |
| Tags disparam antes do banner aceite | **BLOCKER LGPD** | Art. 8º (consent não-inequívoco) | Guia Cookies set/2023 |
| Banner cosmético (não dispara consent update) ⭐ | **BLOCKER LGPD** | Art. 8º (consent não-livre nem inequívoco) | Guia Dark Patterns 2024 |
| Botão "Rejeitar" oculto/micro vs "Aceitar" destacado | **Alta severidade** | Art. 8º (consent não-livre — dark pattern) | Guia Dark Patterns 2024 |
| Política de Privacidade ausente | **BLOCKER LGPD** | Art. 9º + Art. 18º | — |
| Política sem listar 9 direitos Art. 18 | **Alta severidade** | Art. 9º + Art. 18º | Mapa Prioritário 2026-2027 (Tema 1) |
| Compartilhamentos com Meta/Google não declarados na Política | **Alta severidade** | Art. 9º (transparência) | Guia Cookies 2023 |
| Cookies pré-aceite incluem analytics/marketing | **BLOCKER LGPD** | Art. 7º (coleta sem base legal) | Guia Cookies 2023 |
| Canal de exercício de direitos ausente (sem email DPO ou formulário) | **Alta severidade** | Art. 18 + LGPD Art. 41 (DPO) | Mapa Prioritário 2026-2027 |
| Granularidade banner ausente (sem categorias) | **Alta severidade** | Art. 9º (informado) | Guia Cookies 2023 |

---

## 5. Procedure de validação Playwright

### 5.1 Banner present + funcional check

```python
# Detalhe em procedures-playwright-tracking.md §7
# Sub-critérios D8 detectáveis Nível 1:
checks = {
    "banner_present": (DOM scan multiple selectors),
    "default_consent_state": (intercept gtag('consent', 'default', ...)),
    "tags_pre_accept": (count tracking cookies pre-click),
    "consent_update_post_accept": (intercept gtag('consent', 'update', ...)),
    "cosmetic_banner": (banner present + accept clicked + update NOT called),
    "v2_fields_present": (ad_user_data + ad_personalization in consent state object),
    "privacy_policy_linked": (footer link href*='privacidade'),
    "reject_button_parity": (visual size + position vs accept button),
}
```

### 5.2 Política de Privacidade verification

```python
def audit_privacy_policy(page, target_url):
    """
    Sub-critérios canônicos da política:
    """
    privacy_url = None
    for sel in ['a[href*="privacidade"]', 'a[href*="privacy"]']:
        link = page.locator(sel).first
        if link.count() > 0:
            privacy_url = link.get_attribute("href")
            break
    
    if not privacy_url:
        return {"present": False, "blocker": "Politica de Privacidade ausente (LGPD Art. 9º)"}
    
    # Navega pra política + verifica menções Art. 18
    privacy_page = page.context.new_page()
    privacy_page.goto(privacy_url, wait_until="networkidle")
    text = privacy_page.locator("body").inner_text().lower()
    
    art_18_keywords = [
        "direito de acesso", "confirmação", "correção", "anonimização",
        "portabilidade", "eliminação", "revogação do consentimento",
        "art. 18", "direitos do titular"
    ]
    art_18_coverage = sum(1 for kw in art_18_keywords if kw in text)
    
    return {
        "present": True,
        "privacy_url": privacy_url,
        "art_18_coverage_score": art_18_coverage,  # 0-9 — quantos direitos mencionados
        "art_18_complete": art_18_coverage >= 7,
        "dpo_channel": "dpo@" in text or "encarregado" in text or "formulário" in text,
    }
```

---

## 6. Setor regulado — calibrações adicionais

### 6.1 Saúde (clínicas, planos)

- LGPD + Lei 8.078/90 (CDC)
- ANPD + CFM (Conselho Federal de Medicina)
- **CONAR + ANVISA** pra publicidade
- Dados de saúde são **sensíveis** (Art. 11 LGPD) — base legal mais estrita

### 6.2 Educação (escolas privadas)

- LGPD + ECA (Estatuto Criança Adolescente)
- ANPD **Tema Prioritário 2** (proteção crianças/adolescentes)
- Consentimento de pais obrigatório quando dados de menores

### 6.3 Financeiro (bancos, fintechs)

- LGPD + Lei 4.595/64 (Sistema Financeiro)
- Bacen + CVM (cond.)
- **Lei 14.790/2023** (apostas)

### 6.4 Apostas (Lei 14.790/2023)

- LGPD + Ministério da Fazenda + Lei 14.790
- Restrição publicidade pra menores
- Verificação idade obrigatória + consent reforçado

---

## 7. Quando declarar BLOCKER LGPD

| Critério | BLOCKER? |
|---|---|
| Single violation Art. 7º (consent ausente OR granted default) | Sim |
| Single violation Art. 8º (banner cosmético OR tags pre-accept) | Sim |
| Single violation Art. 9º (política ausente) | Sim |
| Single violation Art. 18 (canal exercício ausente) | Não — Alta severidade |
| Combinação Art. 8º + dark pattern documentado ANPD 2024 | Sim — BLOCKER acentuado |

---

## 8. References externas

- [LGPD — Lei 13.709/2018](http://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm)
- [ANPD — Guia Orientativo de Cookies set/2023](https://www.gov.br/anpd/pt-br/documentos-e-publicacoes/guia-cookies.pdf)
- [ANPD — Mapa Temas Prioritários 2026-2027](https://www.gov.br/anpd/pt-br/assuntos/noticias/anpd-publica-mapa-de-temas-prioritarios-para-o-bienio-2026-2027)
- [ANPD — Agenda Regulatória 2025-2026](https://www.gov.br/anpd/pt-br/assuntos/noticias/anpd-publica-mapa-de-temas-prioritarios-para-o-bienio-2026-2027-e-atualiza-agenda-regulatoria-2025-2026)
- [Mattos Filho — Data Privacy Protection Day 2025 + 2026 Outlook](https://www.mattosfilho.com.br/en/unico/data-privacy-protection-day/)
- [Baker McKenzie — Brazil Cookies & Online Tracking](https://resourcehub.bakermckenzie.com/en/resources/global-data-and-cyber-handbook/latin-america/brazil/topics/cookies-online-tracking-and-direct-marketing)
