# Click IDs Taxonomy 2026 — cross-platform attribution

> **Reference da skill `tracking-blackbox-auditor`** — taxonomia completa click IDs + persistência canônica + Safari ITP / Link Tracking Protection 2024-2026 + procedures de captura via Playwright.

---

## 1. O que é click ID

**Click ID** é token único per-click que plataformas de ads injetam na URL de destino no momento do clique. Identifica unicamente aquele click event. Advertiser captura na landing page, persiste (cookie / localStorage / DB), e devolve via postback/API quando user converte → backbone da attribution server-side em 2026.

---

## 2. Click IDs canônicos cross-platform

| Click ID | Plataforma | URL param exemplo | Cookie 1st-party canônico | Lifetime canônico |
|---|---|---|---|---|
| **gclid** | Google Ads (Search + Display + Shopping) | `?gclid=Cj0KCQjw...` | `_gcl_aw` | 90 dias (configurável) |
| **gbraid** | Google Ads iOS web (post-IDFA) | `?gbraid=ABCD...` | `_gcl_aw` (compartilhado) | 90 dias |
| **wbraid** | Google Ads iOS app-to-web (post-IDFA) | `?wbraid=ABCD...` | `_gcl_aw` (compartilhado) | 90 dias |
| **dclid** | Display & Video 360 | `?dclid=ABCD...` | `_gcl_dc` | 90 dias |
| **fbclid** | Meta (Facebook + Instagram) | `?fbclid=IwAR...` | `_fbc` | 90 dias |
| **msclkid** | Microsoft Ads (Bing) | `?msclkid=ABCD...` | `_uetmsclkid` | 90 dias |
| **ttclid** | TikTok Ads | `?ttclid=ABCD...` | `_ttp` (combinado) | 13 meses |
| **li_fat_id** | LinkedIn Ads | `?li_fat_id=ABCD` | `li_fat_id` | 90 dias |
| **epik** | Pinterest Ads | `?epik=ABCD...` | `_pin_unauth` | 365 dias |
| **twclid** | Twitter/X Ads | `?twclid=ABCD...` | `_tw_clk` | 30 dias |
| **scid** | Snapchat Ads | `?ScCid=ABCD...` | `_scid` | 30 dias |

---

## 3. Hierarquia Google Ads — gclid / gbraid / wbraid

**Pós-iOS 14.5 ATT (abril/2021)** Google introduziu identifiers privacy-preserving:

| Identifier | Quando ativo | Funcionalidade |
|---|---|---|
| **gclid** | User com cookies de terceiros ativos (Chrome, Firefox, Safari pré-ATT) | Full attribution + Enhanced Conversions |
| **gbraid** | iOS user clica em web ad → web | Privacy-preserving attribution (sem PII) |
| **wbraid** | iOS user clica em in-app ad → web | App-to-web attribution preservation |

**Implicação auditoria:** sub-critério D7 verifica se cliente captura wbraid + gbraid (não só gclid). Sem essa cobertura = perde 30-40% de iOS attribution.

---

## 4. Cookies canônicos Google + Meta

### 4.1 Google Ads cookies

| Cookie | Função | Owner | Lifetime |
|---|---|---|---|
| `_gcl_aw` | Persiste gclid (Google Ads tag) | `googletagmanager.com` OR 1st-party | 90 dias |
| `_gcl_dc` | Persiste dclid (Display & Video 360) | Google | 90 dias |
| `_gcl_au` | Cross-domain measurement (Google Analytics linker) | 1st-party | 90 dias |
| `_gac_*` | Google Ads UTM tracking | Google | 90 dias |
| `_ga` | GA4 client_id | 1st-party | 13 meses |
| `_ga_<CONTAINER_ID>` | GA4 session tracking | 1st-party | 13 meses |

### 4.2 Meta cookies

| Cookie | Função | Owner | Lifetime |
|---|---|---|---|
| `_fbp` | Meta first-party browser ID (Pixel auto-set) | 1st-party | 90 dias |
| `_fbc` | Persiste fbclid + timestamp + version | 1st-party | 90 dias |
| `fr` | Meta 3rd-party advertising cookie | `facebook.com` | 90 dias |

**Format `_fbc`:** `fb.1.<timestamp>.<fbclid_value>`

```
Exemplo: _fbc=fb.1.1715098234.IwAR0abc123def456...
```

---

## 5. Safari ITP / Link Tracking Protection 2024-2026

### 5.1 O que Safari strippa

Safari 17+ (iOS 17 + macOS Sonoma) tem **Link Tracking Protection** ativo by default em Private Browsing:

| Click ID | Safari strippa? |
|---|---|
| `fbclid` | ⛔ Sim (sempre) |
| `gclid` | ⛔ Sim (Safari 26+ ITP — set 2025) |
| `msclkid` | ⛔ Sim |
| `mc_eid` (Mailchimp) | ⛔ Sim |
| Outros (`utm_*`, custom) | ✅ Preservados |

**Implicação prática:** ~25-35% do traffic Safari perde attribution sem mitigation.

### 5.2 Mitigations canônicas

#### 5.2.1 SGTM 1st-party (canônica recomendada)

Server-side GTM (Stape / Cloud Run) com **custom domain CNAME** servindo container do subdomínio do cliente (`events.cliente.com.br`):

```
1. User clica anúncio Google em Safari → URL com gclid
2. Safari strippa gclid da URL
3. SGTM 1st-party setta cookie `_gcl_aw` VIA HTTP (server-side) ANTES do Safari strippar
4. Cookie 1st-party persiste 90 dias (Safari ITP não strippa 1st-party Set-Cookie)
5. Conversion server-side envia gclid recuperado
```

**Resultado:** recovery 70-85% de attribution Safari.

#### 5.2.2 TAGGRS Click ID Recovery (alternativa community)

TAGGRS open-source tag pra GTM Server-side restaura gclid/gbraid/wbraid strippados pelo Safari 26. Replica funcionalidade SGTM 1st-party.

#### 5.2.3 Sem mitigation = perda aceita

Cliente B2B leadgen com Safari traffic <20% pode aceitar perda. Cliente ecommerce com Safari traffic 30-40% (iOS user base) **deve** ter mitigation.

### 5.3 Sub-critério D7 Safari ITP resilience

```python
def test_safari_itp_resilience(target_url):
    """
    Sub-critério: cliente tem mitigation SGTM 1st-party?
    """
    with sync_playwright() as p:
        browser_webkit = p.webkit.launch()
        context = browser_webkit.new_context()
        page = context.new_page()
        
        test_url = f"{target_url}?gclid=AUDIT_GCLID_TEST&fbclid=AUDIT_FBCLID_TEST"
        page.goto(test_url, wait_until="networkidle")
        
        # 1. Safari strippa da URL?
        final_url = page.url
        gclid_in_url = "AUDIT_GCLID_TEST" in final_url
        
        # 2. Cookie _gcl_aw setado (mitigation funcionou)?
        cookies = context.cookies()
        gcl_aw = next((c for c in cookies if c["name"] == "_gcl_aw"), None)
        fbc = next((c for c in cookies if c["name"] == "_fbc"), None)
        
        # 3. Cookie domain = 1st-party (não googletagmanager.com)?
        gcl_1st_party = gcl_aw and target_url.split("/")[2] in gcl_aw["domain"]
        fbc_1st_party = fbc and target_url.split("/")[2] in fbc["domain"]
        
        browser_webkit.close()
        return {
            "safari_strips_gclid_from_url": not gclid_in_url,
            "gclid_persisted_via_cookie": gcl_aw is not None,
            "sgtm_1st_party_mitigation": gcl_1st_party,
            "fbclid_persisted_via_cookie": fbc is not None,
            "fbc_1st_party": fbc_1st_party,
        }
```

---

## 6. Persistência canônica — first-touch vs last-touch

### 6.1 First-touch attribution

Click ID inicial é preservado em cookie/localStorage por ≥90 dias. CRM submission inclui first-touch click ID. Modelo conservador, preserva crédito do canal de descoberta.

### 6.2 Last-touch attribution

Cada novo click sobrescreve o cookie. CRM submission inclui last-touch (mais recente).

### 6.3 Multi-touch (canônico 2026)

Hybrid — cookie persiste **dois timestamps:**
- `_first_touch_gclid` (lifetime 365 dias)
- `_last_touch_gclid` (lifetime 90 dias, sobrescreve a cada click)

CRM submission inclui ambos → modelo MTA (multi-touch attribution) downstream.

---

## 7. Captura via Playwright (procedure)

Detalhe completo em [`procedures-playwright-tracking.md §6`](procedures-playwright-tracking.md#§6--click-ids-d7).

### 7.1 Test sintético

```python
SYNTHETIC_CLICK_IDS = {
    "gclid": "AUDIT_TEST_GCLID_123",
    "fbclid": "AUDIT_TEST_FBCLID_456",
    "wbraid": "AUDIT_WBRAID_789",
    "gbraid": "AUDIT_GBRAID_321",
    "msclkid": "AUDIT_MSCLKID_654",
    "ttclid": "AUDIT_TTCLID_987",
    "li_fat_id": "AUDIT_LI_FAT_ID_111",
    "dclid": "AUDIT_DCLID_555",
    "epik": "AUDIT_EPIK_222",
}
```

Navega URL com query params → inspeciona cookies + localStorage + sessionStorage → form submit → verifica payload outbound.

### 7.2 Sub-critérios D7

| Sub-critério | Threshold |
|---|---|
| Captura via URL parsing | Click ID detected em request URL initial |
| Persistência ≥90 dias | Cookie `expires` ≥ now+90d |
| First-touch preserved | Reload com novo gclid não sobrescreve first-touch cookie |
| Incluído em CRM submission | Network request pra CRM/webhook inclui click ID |
| `_gcl_aw` cookie GA4 detectado | Cookie `_gcl_aw` setado 1st-party |
| `_fbp` + `_fbc` cookies Meta detectados | Cookies Meta auto-set 1st-party |
| Cross-platform IDs (≥3 click IDs canônicos) | gclid + fbclid + 1 extra mínimo |
| Safari ITP resilience | SGTM 1st-party mitigation ativa |

---

## 8. Scoring D7 (rubrica)

| Score | Critério |
|---|---|
| 10 | Captura completa 7+ click IDs canônicos + persistência 90+ dias 1st-party + first-touch preserved + CRM submission inclui IDs + Safari ITP mitigation via SGTM 1st-party |
| 7-9 | Captura ≥4 click IDs + 1st-party cookies setados + CRM inclui ≥1 ID; Safari mitigation parcial OR ausente em B2B com <20% Safari traffic |
| 4-6 | Captura básica (gclid + fbclid only) + persistência 30-90d + CRM submission falha em incluir IDs OR sem Safari mitigation em alta-Safari traffic |
| 1-3 | Captura ausente OR só URL parsing sem persistência OR Safari Strippping sem mitigation perdendo >30% Safari attribution |
| 0 | Sem captura nenhuma de click IDs |

---

## 9. Anti-patterns canônicos D7

| Anti-pattern | Severidade | Razão |
|---|---|---|
| Persistência só em sessionStorage (perde após fechar tab) | Alta | Lifetime <1 dia inviabiliza attribution |
| Cookie 3rd-party `_gcl_aw` em `googletagmanager.com` (não 1st-party) | Média | Safari ITP strippa em iOS |
| CRM submission sem incluir gclid/fbclid | Alta | Quebra Enhanced Conversions + offline conversions upload |
| wbraid/gbraid não capturado | Média | Perde 30-40% iOS Google Ads attribution |
| li_fat_id ignorado em cliente B2B | Média | LinkedIn é canal B2B principal — auditoria sinaliza |
| Apenas gclid (sem `_gcl_au` cross-domain linker) | Baixa | Multi-domain client perde cross-domain attribution |

---

## 10. References externas

- [Wikipedia — Click Identifier](https://en.wikipedia.org/wiki/Click_identifier)
- [CustomerLabs — Google Click Identifiers Guide 2026](https://www.customerlabs.com/blog/what-are-gclid-gbraid-and-wbraid-parameters/)
- [Simo Ahava — Click Identifiers Common Mistakes](https://www.simoahava.com/analytics/common-mistakes-click-identifiers/)
- [Analytics Mania — Preserve Ad Click IDs with Server-Side Tagging](https://www.analyticsmania.com/post/preserve-ad-click-ids-with-google-tag-manager/)
- [TAGGRS Click ID Recovery (GitHub)](https://github.com/TAGGRS/taggrs-click-id-recovery)
- [Affect — Safari Removes gclid & fbclid](https://affectgroup.com/blog/safari-s-new-update-click-identifiers-removed-from-urls/)
- [Coinis — Click ID Attribution Guide](https://coinis.com/glossary/click-id)
