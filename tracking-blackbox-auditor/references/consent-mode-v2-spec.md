# Google Consent Mode v2 — spec canônica 2024-2026

> **Reference da skill `tracking-blackbox-auditor`** — Consent Mode v2 spec mar/2024 + mudança upcoming junho/2026 + Basic vs Advanced + procedures de validação empírica.

---

## 1. O que é Consent Mode v2

**Consent Mode v2** é o framework Google de communicação de status de consent do usuário ao Google Tag (gtag.js / GTM). Lançado novembro/2023 como extensão obrigatória do Consent Mode v1, com 2 parâmetros novos pra conformidade DMA/LGPD/GDPR.

**Obrigatório desde março/2024** pra anunciantes operando em EEA (UE) + recomendado globalmente. Não-conformidade = ad personalization desabilitada + remarketing audiences perdem volume.

---

## 2. Os 7 signals canônicos

| Signal | Função | Lançado |
|---|---|---|
| `ad_storage` | Controla cookies de advertising (`_gcl_*`, etc.) | v1 |
| `analytics_storage` | Controla cookies de analytics (`_ga`, `_gid`, etc.) | v1 |
| `ad_user_data` | **NOVO v2** — consente envio de dados do usuário pra Google pra advertising measurement (CAPI, Enhanced Conversions) | v2 (nov/2023) |
| `ad_personalization` | **NOVO v2** — consente uso de dados pra personalização (remarketing, audiences) | v2 (nov/2023) |
| `functionality_storage` | Cookies funcionais (login, preferências) | v1 |
| `personalization_storage` | Cookies de personalização não-advertising | v1 |
| `security_storage` | Cookies de segurança (CSRF, etc.) | v1 |

**Diferença crítica:**
- `ad_storage` / `analytics_storage` = **upstream qualifiers** (controlam quais identifiers são enviados nos pings)
- `ad_user_data` / `ad_personalization` = **downstream instructions** (instruem Google services como processar o data recebido)

---

## 3. Default state canônico LGPD compliant

```javascript
// ANTES de qualquer tag carregar:
gtag('consent', 'default', {
  'ad_storage': 'denied',
  'analytics_storage': 'denied',
  'ad_user_data': 'denied',
  'ad_personalization': 'denied',
  'functionality_storage': 'denied',
  'personalization_storage': 'denied',
  'security_storage': 'granted',  // únicos sempre granted (essenciais)
  'wait_for_update': 500  // espera 500ms pelo banner antes de tag default
});
```

**Anti-pattern (BLOCKER LGPD):** default state com `granted` em qualquer signal não-essential.

---

## 4. Pattern de update pós-aceite

```javascript
// USUÁRIO clica "Aceito" no banner:
gtag('consent', 'update', {
  'ad_storage': 'granted',
  'analytics_storage': 'granted',
  'ad_user_data': 'granted',
  'ad_personalization': 'granted',
  'functionality_storage': 'granted',
  'personalization_storage': 'granted'
});
```

Granular update (user aceita só analytics):
```javascript
gtag('consent', 'update', {
  'analytics_storage': 'granted',
  // ad_storage / ad_user_data / ad_personalization permanecem denied
});
```

---

## 5. Basic vs Advanced Consent Mode

| Modo | Comportamento | Quando usar |
|---|---|---|
| **Basic** | Tags NÃO carregam até user aceitar. Zero data antes do consent. | Conservador LGPD — recomendado BR cliente N0/N1 |
| **Advanced** | Tags carregam ANTES do banner mas em modo "consent denied" — Google coleta cookieless pings + modeling. Após aceite, modo upgrades pra full data. | Maior recovery de attribution (~70-85%) mas requer banner + Consent Mode v2 setup robusto |

**Trade-off:**
- Basic = simpler + safer LGPD + perde data de não-consentidos
- Advanced = recovery via modeling Google + requer setup correto (consent default antes de tags + update pós-aceite)

---

## 6. Behavior modeling (Advanced only)

Quando Consent Mode v2 Advanced está ativo:
- User NÃO consente → Google envia cookieless pings com `gcs=G100` (consent denied)
- Google modela conversions perdidas via ML (model contribution)
- Reports GA4 / Google Ads mostram total = observed (consented) + modeled (not consented)
- Modeling threshold: precisa ≥1.000 conversions/30d pra modeling ativar (sites pequenos ficam sem modeling)

---

## 7. Mudança upcoming — 15 junho/2026

A partir de **15 jun/2026**, Google Analytics vai usar Consent Mode dentro de Google Ads como single control pra data linkage:

| Antes (atual) | Depois (jun/2026) |
|---|---|
| `ad_storage` controla cookies advertising globalmente | `ad_storage` controla GA4 separado de Google Ads |
| `analytics_storage` controla cookies GA4 | Mesmo + `ad_personalization` controla Analytics→Ads linkage |
| Sem split formal entre GA4 e Ads | **Split formal** — GA4 property linkada a Google Ads account só compartilha dados pra personalization se `ad_personalization: granted` |

**Implicação prática:** cliente com setup atual `ad_storage: granted + ad_personalization: denied` vai perder linkage GA4→Ads pra personalization após jun/2026.

**Sub-critério auditoria 2026 forward-looking:** sinaliza se cliente está preparado pra a mudança — `ad_personalization` explícito + linkagem GA4→Ads consciente.

---

## 8. Validação empírica via Playwright

### 8.1 Audit checklist

```python
def audit_consent_v2(page, target_url):
    """
    Sub-critérios canônicos D8:
    """
    checks = {
        "banner_present": False,
        "default_state_denied": False,
        "v2_fields_present": False,  # ad_user_data + ad_personalization
        "tags_pre_accept": False,
        "consent_update_called": False,
        "cosmetic_banner": False,  # banner presente mas update nunca chamado
        "granularity_ui": False,
        "reject_button_equal_prominence": False,
    }
    
    # Setup monkey-patch consent calls (ver procedures-playwright-tracking.md §7)
    # ...
    
    return checks
```

### 8.2 Detecção de banner cosmético ⭐ (refinamento Aquatro)

**Banner cosmético** = banner DOM presente mas clicar "aceito" NÃO dispara `gtag('consent', 'update', ...)`. Consent já tá `granted` no default state.

```python
def is_cosmetic_banner(banner_present, consent_calls_post_accept):
    """
    True se banner existe mas update nunca foi chamado pós-click.
    BLOCKER LGPD Art. 8º + ANPD Guia Dark Patterns 2024.
    """
    update_called = any(
        len(c["args"]) > 1 and c["args"][1] == "update"
        for c in consent_calls_post_accept
    )
    return banner_present and not update_called
```

---

## 9. Mapping signals → cookies controlados

| Signal | Cookies controlados |
|---|---|
| `ad_storage` | `_gcl_au`, `_gcl_aw`, `_gcl_dc`, `_gac_*`, `IDE` (DoubleClick), `_fbp`, `_fbc` (cond.) |
| `analytics_storage` | `_ga`, `_gid`, `_gat`, `_ga_<container_id>` (GA4), `_utma`, `_utmb`, etc. |
| `ad_user_data` | (não controla cookies — controla envio de PII pra Google CAPI/Enhanced Conversions) |
| `ad_personalization` | (não controla cookies — controla uso pra remarketing audiences) |
| `functionality_storage` | login session, preferências user |
| `personalization_storage` | recommendation, content personalization |
| `security_storage` | CSRF tokens, fraud detection |

---

## 10. Vendors de banner canônicos (Brasil 2024-2026)

| Vendor | LGPD compliance | Consent Mode v2 ready | Notas |
|---|---|---|---|
| **OneTrust** | ✅ | ✅ | Enterprise — caro |
| **Cookiebot** | ✅ | ✅ | Boa adoção BR mid-market |
| **Iubenda** | ✅ | ✅ | IT-based, mid-market BR |
| **Termly** | ✅ | ✅ | Acessível pequenas empresas |
| **CookieYes** | ✅ | ✅ | Free tier disponível |
| **Custom DIY** | ⚠️ | ⚠️ | Requer setup robusto — risco anti-pattern |
| **RD Marketing built-in** | ⚠️ Parcial | ⚠️ Parcial | Cobre LGPD básico, gaps Consent Mode v2 |
| **Page builder embeddeds (Klickpages, etc.)** | ❌ Geralmente cosmético | ❌ | **Caso Aquatro** — banner presente mas não funcional |

---

## 11. Anti-patterns canônicos D8

| Anti-pattern | Severidade | Spec violada |
|---|---|---|
| Default state = `granted` sem aceite | **BLOCKER LGPD** | LGPD Art. 7º + Consent Mode v2 spec |
| Tags disparam antes do banner aceite | **BLOCKER LGPD** | LGPD Art. 8º |
| Banner ausente em LP que coleta dados | **BLOCKER LGPD** | LGPD Art. 7º + ANPD Guia Cookies 2023 |
| **Banner cosmético** (presente mas update nunca chamado) | **BLOCKER LGPD** | LGPD Art. 8º + ANPD Guia Dark Patterns 2024 |
| `ad_user_data` / `ad_personalization` ausentes | **Alta severidade** | Consent Mode v2 spec mar/2024 |
| Botão "Rejeitar" escondido / micro / sem destaque | **Alta severidade** | LGPD Art. 8º (consent livre) + ANPD dark patterns |
| Política privacidade ausente | **BLOCKER LGPD** | LGPD Art. 9º + Art. 18º |
| Cookies pré-aceite incluem analytics/marketing | **BLOCKER LGPD** | LGPD Art. 7º (sem base legal) |
| Consent state não persistido cross-page | **Alta severidade** | UX issue + double consent |

---

## 12. References externas

- [Google — Consent Mode for Websites](https://developers.google.com/tag-platform/security/guides/consent)
- [Google — Consent Mode Reference (Ads Help)](https://support.google.com/google-ads/answer/13802165)
- [Simo Ahava — Consent Mode V2 For Google Tags](https://www.simoahava.com/analytics/consent-mode-v2-google-tags/)
- [Secure Privacy — ad_user_data Explained](https://secureprivacy.ai/blog/ad-user-data-consent-mode)
- [Google — GA4 Consent Settings Verification](https://support.google.com/analytics/answer/14275483)
- [UniConsent — Google Consent Mode June 2026 Update](https://www.uniconsent.com/blog/google-ads-consent-mode-change-2026)
- [ALM Corp — GA4/Ads Consent Split June 15 2026](https://almcorp.com/blog/ga4-google-ads-consent-controls-split-june-2026/)
