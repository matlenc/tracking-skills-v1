# Procedures Playwright Tracking — snippets literais 2026

> **Reference técnico da skill `tracking-blackbox-auditor`** — snippets Playwright + monkey-patches + interceptors + estratégia cross-navigation pra capturar payloads pós-redirect cross-origin.
>
> **Pattern crítico (D-AUDITORES-5):** **NUNCA confiar só no JS interceptor — Playwright nativo `browser_network_requests` como redundância obrigatória sempre.** Lição empírica Aquatro 2026-05-11: `window.location.href = ...` setter escapa Object.defineProperty no Chrome (security restriction). Tool nativo Playwright captura DENTRO do browser engine, independente do JS rodar.

---

## §1 — Stack detection (D1)

```python
# Playwright Python — detecção de plataformas via DOM + globals + network
from playwright.sync_api import sync_playwright

TRACKING_DOMAINS = [
    "googletagmanager.com", "google-analytics.com", "googleadservices.com",
    "facebook.com", "connect.facebook.net", "facebook.net",
    "linkedin.com", "ads.linkedin.com",
    "analytics.tiktok.com", "tiktok.com",
    "bat.bing.com", "clarity.ms",
    "static.hotjar.com", "fullstory.com",
    "d335luupugsy2.cloudfront.net",  # RD Station
    "js.hs-scripts.com",  # HubSpot
    "ct.pinterest.com", "sc-static.net",  # Pinterest
    "tr.snapchat.com",
]

PLATFORM_DETECTORS = {
    "ga4": {
        "html_regex": r'googletagmanager\.com/gtag/js\?id=G-',
        "global": "gtag",
    },
    "gtm": {
        "html_regex": r'googletagmanager\.com/gtm\.js\?id=GTM-',
        "global": "google_tag_manager",
    },
    "meta_pixel": {
        "html_regex": r'connect\.facebook\.net/.*fbevents\.js',
        "global": "fbq",
    },
    "google_ads": {
        "html_regex": r'googletagmanager\.com/gtag/js\?id=AW-',
    },
    "linkedin_insight": {
        "global": "_linkedin_partner_id",
    },
    "tiktok_pixel": {
        "global": "ttq",
    },
    "microsoft_uet": {
        "global": "uetq",
    },
    "clarity": {
        "global": "clarity",
    },
    "hotjar": {
        "global": "hj",
    },
    "fullstory": {
        "global": "FS",
    },
    "rd_station": {
        "html_regex": r'd335luupugsy2\.cloudfront\.net|RdIntegration',
        "global": "RdIntegration",
    },
    "hubspot": {
        "html_regex": r'js\.hs-scripts\.com',
        "global": "_hsq",
    },
    # DEPRECATED — BLOCKER técnico
    "universal_analytics": {
        "html_regex": r'google-analytics\.com/analytics\.js',
    },
}

def detect_stack(page):
    html = page.content()
    detected = {}
    for name, det in PLATFORM_DETECTORS.items():
        present = False
        if "html_regex" in det:
            import re
            present = bool(re.search(det["html_regex"], html))
        if not present and "global" in det:
            present = page.evaluate(f"() => typeof window.{det['global']} !== 'undefined'")
        if present:
            detected[name] = True
    return detected
```

### §1.1 GTM Server-side detection

Subdomínio próprio servindo `gtm.js` (não `googletagmanager.com`) sinaliza SGTM. Patterns:

```python
def detect_sgtm(network_requests, primary_domain):
    """
    SGTM signature: gtm.js servido de subdomínio do cliente (não googletagmanager.com)
    OU Cloud Run *.run.app OU Stape signature events.*/tracking.*/tagm.*/gtm-server.*
    """
    sgtm_patterns = [
        f"events.{primary_domain}", f"tracking.{primary_domain}",
        f"tagm.{primary_domain}", f"gtm-server.{primary_domain}",
        f"sgtm.{primary_domain}", f"server.{primary_domain}",
    ]
    cloud_run_pattern = r"\.run\.app/"
    for req in network_requests:
        if any(p in req.url for p in sgtm_patterns):
            return "stape_pattern"
        if "/gtm.js" in req.url and "googletagmanager.com" not in req.url:
            return "sgtm_custom_subdomain"
        if re.search(cloud_run_pattern, req.url) and "/gtm" in req.url:
            return "cloud_run_sgtm"
    return None
```

---

## §2 — DataLayer monkey-patch (D2)

```javascript
// Injetar ANTES do load via page.add_init_script (Playwright)
page.add_init_script(`
  (function() {
    window._auditPushes = [];
    Object.defineProperty(window, 'dataLayer', {
      configurable: true,
      get() { return this._dataLayerStore; },
      set(val) {
        this._dataLayerStore = val;
        if (Array.isArray(val)) {
          const origPush = val.push.bind(val);
          val.push = function(...args) {
            window._auditPushes.push({
              time: Date.now(),
              args: JSON.parse(JSON.stringify(args))
            });
            return origPush(...args);
          };
        }
      }
    });
  })();
`)
```

### §2.1 Coletando pushes após jornada

```python
# Após jornada simulada (clicks/scroll/form fill):
pushes = page.evaluate("() => window._auditPushes")
for p in pushes:
    print(f"[{p['time']}] {p['args']}")
```

### §2.2 Schema versioning detection (Snowplow-style)

```python
def detect_schema_versioning(pushes):
    """
    Procura `schema_version` (SemVer) ou `_schema` (SchemaVer MODEL-REVISION-ADDITION) no payload
    """
    for push in pushes:
        for arg in push["args"]:
            if isinstance(arg, dict):
                if "schema_version" in arg or "_schema" in arg or "_schema_version" in arg:
                    return True
    return False
```

---

## §3 — Jornada simulada (D3)

```python
# Modo `lp` — jornada B2B leadgen padrão
def simulate_lp_journey(page):
    # 1. PageView automático (Playwright já navegou)
    # 2. Click hero CTA
    hero_cta = page.locator('a[href*="form"], button:has-text("Quero"), button:has-text("Falar"), .hero a').first
    if hero_cta.count() > 0:
        hero_cta.click()
    
    # 3. Scroll milestones 25/50/75/100%
    for pct in [25, 50, 75, 100]:
        page.evaluate(f"window.scrollTo(0, document.body.scrollHeight * {pct/100})")
        page.wait_for_timeout(500)
    
    # 4. Form fill + submit com dados sintéticos
    form = page.locator("form").first
    if form.count() > 0:
        # Email
        email_input = form.locator('input[type="email"], input[name*="email"]').first
        if email_input.count() > 0:
            email_input.fill(f"audit-test+{int(time.time())}@auditor-blackbox.com")
        # Phone
        phone_input = form.locator('input[type="tel"], input[name*="phone"], input[name*="telefone"]').first
        if phone_input.count() > 0:
            phone_input.fill("+5511999999999")
        # Name
        name_input = form.locator('input[name*="name"], input[name*="nome"]').first
        if name_input.count() > 0:
            name_input.fill("Audit Tester Black-Box")
        # Submit
        submit = form.locator('button[type="submit"], input[type="submit"]').first
        if submit.count() > 0:
            submit.click()
            page.wait_for_load_state("networkidle", timeout=10000)
    
    # 5. Outbound clicks (cond.)
    outbound_links = page.locator('a[href^="http"]:not([href*="' + page.url.split("/")[2] + '"])')
    if outbound_links.count() > 0:
        # Não clicamos — só inspecionamos se outbound_click event capturaria
        pass
    
    # 6. Phone tel: + WhatsApp wa.me/
    tel_link = page.locator('a[href^="tel:"]').first
    whatsapp_link = page.locator('a[href*="wa.me/"], a[href*="api.whatsapp.com"]').first
```

### §3.1 Modo `site` — multi-page

```python
def simulate_site_journey(page, target_url):
    pages_to_visit = [target_url, "/sobre", "/contato", "/blog", "/servicos"]
    for path in pages_to_visit[:5]:
        full_url = target_url.rstrip("/") + path if path.startswith("/") else path
        try:
            page.goto(full_url, wait_until="networkidle", timeout=10000)
            page.wait_for_timeout(2000)
        except Exception as e:
            continue
```

### §3.2 Modo `ecommerce` — checkout flow (D9)

```python
def simulate_ecommerce_journey(page):
    # 1. Listing
    # 2. Click item → PDP
    # 3. Add to cart
    # 4. View cart
    # 5. Begin checkout
    # 6. Add shipping info
    # 7. Add payment info
    # 8. NÃO completar purchase (skill é não-destrutiva)
    pass
```

---

## §4 — Captura payload Meta `/tr` (D4 + D5)

```python
# ⭐ CRÍTICO — usar Playwright nativo, NÃO confiar só em monkey-patch fbq
meta_payloads = []

def on_request(request):
    if "facebook.com/tr" in request.url or "/tr/?id=" in request.url:
        meta_payloads.append({
            "url": request.url,
            "method": request.method,
            "post_data": request.post_data,
            "headers": request.headers,
        })

page.on("request", on_request)

# Após jornada, parsear payloads:
from urllib.parse import urlparse, parse_qs

def parse_meta_payload(payload_url):
    parsed = urlparse(payload_url)
    params = parse_qs(parsed.query)
    return {
        "event": params.get("ev", [None])[0],
        "event_id": params.get("eid", [None])[0],
        "pixel_id": params.get("id", [None])[0],
        "ud_em": params.get("ud[em]", [None])[0],
        "ud_ph": params.get("ud[ph]", [None])[0],
        "ud_fn": params.get("ud[fn]", [None])[0],
        "ud_ln": params.get("ud[ln]", [None])[0],
        "ud_external_id": params.get("ud[external_id]", [None])[0],
        "fbp": params.get("fbp", [None])[0],
        "fbc": params.get("fbc", [None])[0],
    }
```

### §4.1 Hash verification

```python
import hashlib

def verify_hash_normalization(detected_hash, expected_plain):
    """
    Test sintético — passa `test@example.com` → SHA-256 esperado.
    Se hash detectado bate com SHA-256(lowercase + trim(plain)) → normalização correta.
    """
    normalized = expected_plain.lower().strip()
    expected_hash = hashlib.sha256(normalized.encode()).hexdigest()
    return detected_hash == expected_hash

# Conhecidos:
KNOWN_HASHES = {
    "test@example.com": "973dfe463ec85785f5f95af5ba3906eedb2d931c24e69824a89ea65dba4e813b",
    "audit@test.com": hashlib.sha256("audit@test.com".encode()).hexdigest(),
}
```

### §4.2 EMQ score estimation

```python
def estimate_emq(meta_payload):
    """
    Fórmula Meta 2024-2026 — detalhe em emq-formula-meta.md
    """
    score = 2.0  # base
    if meta_payload.get("ud_em") and len(meta_payload["ud_em"]) == 64:
        score += 2.0
    if meta_payload.get("ud_ph") and len(meta_payload["ud_ph"]) == 64:
        score += 2.0
    if meta_payload.get("ud_fn") and meta_payload.get("ud_ln"):
        score += 1.5
    if all(meta_payload.get(k) for k in ["ud_ct", "ud_st", "ud_zp", "ud_country"]):
        score += 1.0
    if meta_payload.get("ud_external_id"):
        score += 1.5
    if meta_payload.get("ud_client_ip_address"):
        score += 0.5
    if meta_payload.get("ud_client_user_agent"):
        score += 0.5
    if meta_payload.get("fbc"):
        score += 1.0
    if meta_payload.get("fbp"):
        score += 0.5
    return min(score, 10.0)
```

---

## §5 — Dedup CAPI (D6)

```python
def detect_server_side_dedup(network_requests, browser_payload_event_id):
    """
    Procura request server-side com mesmo event_id do browser payload.
    Heurísticas inferência server-side sem precisar Nível 4:
    """
    sgtm_signatures = ["events.", "tracking.", "tagm.", "gtm-server.", "sgtm.", "server."]
    
    server_side_endpoints = []
    for req in network_requests:
        url = req.url
        # 1. Subdomain suspect (Stape signature)
        if any(sig in url for sig in sgtm_signatures):
            server_side_endpoints.append({"type": "stape_pattern", "url": url})
        # 2. Cloud Run subdomain
        if ".run.app" in url:
            server_side_endpoints.append({"type": "cloud_run", "url": url})
        # 3. N8N webhook visible
        if "/webhook/" in url:
            server_side_endpoints.append({"type": "n8n_webhook", "url": url})
    
    # Browser-server matching
    server_request_with_same_eid = None
    for req in network_requests:
        if browser_payload_event_id and browser_payload_event_id in (req.post_data or "") + req.url:
            server_request_with_same_eid = req.url
    
    return {
        "server_side_endpoints_detected": server_side_endpoints,
        "matching_event_id": server_request_with_same_eid,
    }
```

### §5.1 UUID v4 format check

```python
import re

UUID_V4_REGEX = re.compile(
    r"^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$",
    re.IGNORECASE
)

def is_valid_uuid_v4(event_id):
    return bool(event_id and UUID_V4_REGEX.match(event_id))
```

---

## §6 — Click IDs (D7)

```python
SYNTHETIC_CLICK_IDS = {
    "gclid": "AUDIT_TEST_GCLID_123",
    "fbclid": "AUDIT_TEST_FBCLID_456",
    "wbraid": "AUDIT_WBRAID_789",
    "gbraid": "AUDIT_GBRAID_321",
    "msclkid": "AUDIT_MSCLKID_654",
    "ttclid": "AUDIT_TTCLID_987",
    "li_fat_id": "AUDIT_LI_FAT_ID_111",
}

def test_click_ids_capture(page, target_url):
    """
    Navega URL com query params sintéticos, inspeciona persistência.
    """
    query_string = "&".join(f"{k}={v}" for k, v in SYNTHETIC_CLICK_IDS.items())
    test_url = f"{target_url}?{query_string}"
    page.goto(test_url, wait_until="networkidle")
    
    # Inspeciona cookies
    cookies = page.context.cookies()
    cookie_results = {}
    for c in cookies:
        for click_id, syn_value in SYNTHETIC_CLICK_IDS.items():
            if syn_value in c["value"]:
                cookie_results[click_id] = {"cookie_name": c["name"], "expires": c.get("expires"), "domain": c.get("domain")}
    
    # Inspeciona localStorage / sessionStorage
    ls_results = page.evaluate(f"""() => {{
        const targets = {list(SYNTHETIC_CLICK_IDS.values())};
        const found = {{}};
        for (let i = 0; i < localStorage.length; i++) {{
            const key = localStorage.key(i);
            const val = localStorage.getItem(key);
            for (const target of targets) {{
                if (val && val.includes(target)) {{
                    found[key] = {{store: 'localStorage', value: val}};
                }}
            }}
        }}
        for (let i = 0; i < sessionStorage.length; i++) {{
            const key = sessionStorage.key(i);
            const val = sessionStorage.getItem(key);
            for (const target of targets) {{
                if (val && val.includes(target)) {{
                    found[key] = {{store: 'sessionStorage', value: val}};
                }}
            }}
        }}
        return found;
    }}""")
    
    # Inspeciona cookies GA4 + Meta
    canonical_cookies = {
        "_gcl_aw": next((c for c in cookies if c["name"] == "_gcl_aw"), None),
        "_fbp": next((c for c in cookies if c["name"] == "_fbp"), None),
        "_fbc": next((c for c in cookies if c["name"] == "_fbc"), None),
    }
    
    return {
        "cookie_results": cookie_results,
        "ls_results": ls_results,
        "canonical_cookies_present": {k: v is not None for k, v in canonical_cookies.items()},
    }
```

### §6.1 Safari ITP / Link Tracking Protection resilience

```python
def test_safari_itp_resilience(target_url):
    """
    Testa se cliente tem mitigation via SGTM 1st-party
    (cookies setados via HTTP server-side bypassam ITP).
    """
    with sync_playwright() as p:
        # Safari WebKit
        browser_webkit = p.webkit.launch()
        context_webkit = browser_webkit.new_context()
        page_webkit = context_webkit.new_page()
        page_webkit.goto(target_url + f"?gclid={SYNTHETIC_CLICK_IDS['gclid']}&fbclid={SYNTHETIC_CLICK_IDS['fbclid']}")
        # Inspeciona se gclid/fbclid foram strippped pelo Safari
        url_after = page_webkit.url
        gclid_stripped = SYNTHETIC_CLICK_IDS["gclid"] not in url_after
        fbclid_stripped = SYNTHETIC_CLICK_IDS["fbclid"] not in url_after
        
        # Se SGTM 1st-party está ativo, cookies persistem mesmo com stripping
        cookies_webkit = context_webkit.cookies()
        gclid_persisted_via_sgtm = any(SYNTHETIC_CLICK_IDS["gclid"] in c["value"] for c in cookies_webkit)
        
        browser_webkit.close()
        return {
            "safari_strips_gclid": gclid_stripped,
            "safari_strips_fbclid": fbclid_stripped,
            "sgtm_1st_party_resilience": gclid_persisted_via_sgtm,
        }
```

---

## §7 — Consent Mode v2 + LGPD (D8)

```python
def audit_consent_mode(page, target_url):
    """
    Observa gtag('consent', ...) calls antes/depois do banner + scan cookies pré-aceite.
    """
    # 1. Setup monkey-patch consent calls
    page.add_init_script("""
        window._consentCalls = [];
        const origGtag = window.gtag;
        Object.defineProperty(window, 'gtag', {
            configurable: true,
            get() { return this._gtagStore || function() {}; },
            set(val) {
                this._gtagStore = function(...args) {
                    if (args[0] === 'consent') {
                        window._consentCalls.push({time: Date.now(), args: args});
                    }
                    return val(...args);
                };
            }
        });
    """)
    
    # 2. Navega SEM aceitar banner
    page.goto(target_url, wait_until="networkidle")
    page.wait_for_timeout(3000)
    
    consent_calls_pre_accept = page.evaluate("() => window._consentCalls || []")
    cookies_pre_accept = page.context.cookies()
    
    # 3. Detecta banner
    banner_selectors = [
        '[id*="cookie"], [class*="cookie"]',
        '[id*="consent"], [class*="consent"]',
        '[id*="lgpd"], [class*="lgpd"]',
        '.iubenda-cs-container, #onetrust-banner-sdk, #cookiebot-banner',
        '.cookieyes-banner, #cky-consent',
    ]
    banner_present = False
    for sel in banner_selectors:
        if page.locator(sel).count() > 0:
            banner_present = True
            break
    
    # 4. Tenta aceitar
    accept_button_selectors = [
        'button:has-text("Aceitar")', 'button:has-text("Aceito")',
        'button:has-text("Entendi")', 'button:has-text("Concordo")',
        'button:has-text("Accept")', '#onetrust-accept-btn-handler',
        '.cky-btn-accept', '.iubenda-cs-accept-btn',
    ]
    accept_button = None
    for sel in accept_button_selectors:
        if page.locator(sel).count() > 0:
            accept_button = page.locator(sel).first
            break
    
    consent_update_called = False
    if accept_button:
        try:
            accept_button.click()
            page.wait_for_timeout(2000)
            consent_calls_post_accept = page.evaluate("() => window._consentCalls || []")
            consent_update_called = any(
                len(c["args"]) > 1 and c["args"][1] == "update"
                for c in consent_calls_post_accept[len(consent_calls_pre_accept):]
            )
        except Exception:
            pass
    
    # 5. Categoriza cookies pré-aceite
    cookies_pre_accept_categorized = {
        "tracking": [c for c in cookies_pre_accept if any(t in c["name"] for t in ["_ga", "_fbp", "_gcl", "_uet", "_ttp", "li_at"])],
        "functional": [c for c in cookies_pre_accept if c["name"] in ("session", "csrf", "auth")],
    }
    
    # 6. Default consent state (denied/granted)
    default_state = None
    for c in consent_calls_pre_accept:
        if len(c["args"]) > 1 and c["args"][1] == "default":
            default_state = c["args"][2] if len(c["args"]) > 2 else None
            break
    
    return {
        "banner_present": banner_present,
        "default_consent_state": default_state,
        "tags_fired_pre_accept": len(cookies_pre_accept_categorized["tracking"]) > 0,
        "consent_update_called_post_accept": consent_update_called,
        "cosmetic_banner_detected": banner_present and accept_button is not None and not consent_update_called,
        "consent_mode_v2_fields_present": _check_v2_fields(consent_calls_pre_accept),
    }

def _check_v2_fields(consent_calls):
    """
    Consent Mode v2 obrigatório: ad_user_data + ad_personalization (desde mar/2024).
    """
    for c in consent_calls:
        if len(c["args"]) > 2 and isinstance(c["args"][2], dict):
            keys = c["args"][2].keys()
            if "ad_user_data" in keys and "ad_personalization" in keys:
                return True
    return False
```

### §7.1 Política privacidade + Art. 18 direitos titular

```python
def detect_privacy_policy(page):
    """
    Footer com link /privacidade + direitos LGPD Art. 18 explícitos?
    """
    privacy_link_selectors = [
        'a[href*="privacidade"]', 'a[href*="privacy"]',
        'a:has-text("Política de Privacidade")', 'a:has-text("Privacidade")',
    ]
    privacy_present = any(page.locator(sel).count() > 0 for sel in privacy_link_selectors)
    return privacy_present
```

---

## §8 — LCP via PerformanceObserver (D10) ⭐

```python
# ⭐ CRÍTICO — NÃO usar performance.getEntriesByType direto
def measure_lcp_correctly(page):
    """
    Refinamento Aquatro — PerformanceObserver com event loop tick.
    `performance.getEntriesByType('largest-contentful-paint')` retorna []
    se observer não registrado antes do load.
    """
    lcp = page.evaluate("""() => new Promise(resolve => {
        let lcp = null;
        new PerformanceObserver(list => {
            const entries = list.getEntries();
            if (entries.length > 0) {
                lcp = entries[entries.length - 1].startTime;
            }
        }).observe({ type: 'largest-contentful-paint', buffered: true });
        setTimeout(() => resolve(lcp), 3000);
    })""")
    return lcp

# CLS via PerformanceObserver
def measure_cls(page):
    cls = page.evaluate("""() => new Promise(resolve => {
        let cls = 0;
        new PerformanceObserver(list => {
            for (const entry of list.getEntries()) {
                if (!entry.hadRecentInput) {
                    cls += entry.value;
                }
            }
        }).observe({ type: 'layout-shift', buffered: true });
        setTimeout(() => resolve(cls), 3000);
    })""")
    return cls

# FCP via PerformanceObserver
def measure_fcp(page):
    fcp = page.evaluate("""() => new Promise(resolve => {
        new PerformanceObserver(list => {
            const entries = list.getEntries();
            const fcp = entries.find(e => e.name === 'first-contentful-paint');
            resolve(fcp ? fcp.startTime : null);
        }).observe({ type: 'paint', buffered: true });
        setTimeout(() => resolve(null), 3000);
    })""")
    return fcp
```

### §8.1 Console errors + network failures

```python
console_errors = []
network_failures = []

def on_console(msg):
    if msg.type == "error":
        console_errors.append({"text": msg.text, "location": msg.location})

def on_response(response):
    if response.status >= 400:
        network_failures.append({"url": response.url, "status": response.status})

page.on("console", on_console)
page.on("response", on_response)

# Filtrar network_failures pra endpoints tracking-only:
tracking_failures = [f for f in network_failures if any(d in f["url"] for d in TRACKING_DOMAINS)]
```

---

## §9 — Pattern crítico: cross-navigation network capture (D-AUDITORES-5)

```python
# ⭐ LIÇÃO AQUATRO — window.location.href setter escapa monkey-patch JS
# Solução canônica: page.on('request') captura DENTRO do browser engine

all_requests = []

def on_request_captured(request):
    all_requests.append({
        "url": request.url,
        "method": request.method,
        "post_data": request.post_data,
        "resource_type": request.resource_type,
        "frame_url": request.frame.url if request.frame else None,
        "timestamp": time.time(),
    })

page.on("request", on_request_captured)

# Tentativa de bloquear monkey-patch interceptor:
page.add_init_script("""
    // Override location.assign + location.replace (parcial — não funciona em href= setter)
    const origAssign = window.location.assign.bind(window.location);
    const origReplace = window.location.replace.bind(window.location);
    window.location.assign = function(url) {
        console.log('[AUDIT] location.assign:', url);
        return origAssign(url);
    };
    window.location.replace = function(url) {
        console.log('[AUDIT] location.replace:', url);
        return origReplace(url);
    };
    // window.location.href = ... setter NÃO é trapável via defineProperty (Chrome security restriction)
""")

# ⭐ MESMO ASSIM, redundância: page.on('request') captura tudo
```

### §9.1 Cross-origin redirect handling

```python
def follow_redirect_chain(page, target_url):
    """
    Se URL alvo redireciona pra outro domínio (cross-origin), Playwright segue.
    Audita destino final + reporta redirect chain.
    """
    redirect_chain = []
    
    def on_response(response):
        if response.status in (301, 302, 303, 307, 308):
            redirect_chain.append({
                "from": response.url,
                "to": response.headers.get("location"),
                "status": response.status,
            })
    
    page.on("response", on_response)
    page.goto(target_url, wait_until="networkidle")
    return {
        "final_url": page.url,
        "redirect_chain": redirect_chain,
    }
```

---

## §10 — Cookie scanner LGPD categorizado (`--lgpd-scan=true`)

```python
def lgpd_cookie_scan(page):
    """
    Cookie scanner profundo categorizado (OneTrust scanner pattern).
    """
    cookies = page.context.cookies()
    categorized = {
        "estritamente_necessarios": [],
        "funcionais": [],
        "analytics": [],
        "marketing": [],
        "desconhecidos": [],
    }
    
    KNOWN_COOKIES = {
        # Estritamente necessários
        "session": "estritamente_necessarios",
        "csrf": "estritamente_necessarios",
        "PHPSESSID": "estritamente_necessarios",
        "JSESSIONID": "estritamente_necessarios",
        "auth_token": "estritamente_necessarios",
        # Funcionais
        "language": "funcionais",
        "currency": "funcionais",
        "theme": "funcionais",
        # Analytics
        "_ga": "analytics",
        "_gid": "analytics",
        "_gat": "analytics",
        "_ga_": "analytics",  # GA4 prefix
        # Marketing
        "_gcl_aw": "marketing",
        "_gcl_dc": "marketing",
        "_gcl_au": "marketing",
        "_fbp": "marketing",
        "_fbc": "marketing",
        "fr": "marketing",
        "_uetsid": "marketing",
        "_uetvid": "marketing",
        "_ttp": "marketing",
        "tt_appInfo": "marketing",
        "li_at": "marketing",
        "li_sugr": "marketing",
        "li_fat_id": "marketing",
        "lidc": "marketing",
    }
    
    for c in cookies:
        category = "desconhecidos"
        for known, cat in KNOWN_COOKIES.items():
            if c["name"] == known or c["name"].startswith(known):
                category = cat
                break
        categorized[category].append({
            "name": c["name"],
            "domain": c["domain"],
            "expires": c.get("expires"),
            "secure": c.get("secure"),
            "httpOnly": c.get("httpOnly"),
            "sameSite": c.get("sameSite"),
        })
    
    return categorized
```

---

## §11 — Workflow completo Playwright (skill audita Nível 1)

```python
def audit_lp_nivel_1(target_url, output_path):
    """
    Auditoria modo `lp` Nível 1 (só URL) — workflow completo.
    """
    from playwright.sync_api import sync_playwright
    
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        context = browser.new_context()
        page = context.new_page()
        
        # 1. Setup ALL interceptors (request + response + console)
        all_requests = []
        all_responses = []
        console_errors = []
        
        page.on("request", lambda r: all_requests.append(r))
        page.on("response", lambda r: all_responses.append(r))
        page.on("console", lambda m: console_errors.append(m) if m.type == "error" else None)
        
        # 2. Inject monkey-patches BEFORE load
        page.add_init_script("""
            window._auditPushes = [];
            window._consentCalls = [];
            // ... (full setup)
        """)
        
        # 3. Navega + jornada
        page.goto(target_url, wait_until="networkidle")
        
        # 4. Aplica todos os módulos:
        stack_detected = detect_stack(page)
        lcp_value = measure_lcp_correctly(page)
        cls_value = measure_cls(page)
        click_ids = test_click_ids_capture(page, target_url)
        consent_audit = audit_consent_mode(page, target_url)
        
        # 5. Jornada simulada
        simulate_lp_journey(page)
        
        # 6. Captura pushes + payloads
        dl_pushes = page.evaluate("() => window._auditPushes")
        meta_payloads = [r for r in all_requests if "facebook.com/tr" in r.url]
        
        # 7. EMQ estimation
        emq_scores = [estimate_emq(parse_meta_payload(p.url)) for p in meta_payloads]
        
        # 8. Server-side inference
        server_side = detect_server_side_dedup(all_requests, None)
        
        browser.close()
        
        # 9. Compõe relatório (próximos passos — templates de output)
        return {
            "stack": stack_detected,
            "cwv": {"lcp": lcp_value, "cls": cls_value},
            "click_ids": click_ids,
            "consent": consent_audit,
            "dl_pushes": dl_pushes,
            "meta_payloads_count": len(meta_payloads),
            "emq_scores": emq_scores,
            "server_side": server_side,
            "console_errors_count": len(console_errors),
        }
```

---

## §12 — Pré-requisitos operacionais

| Pré-req | Detalhe |
|---|---|
| Playwright Python | `pip install playwright && playwright install chromium webkit firefox` |
| Headless mode | Default — invisível em produção |
| Timeout default | `30s` por navegação; `10s` por wait_for_load_state |
| User agent | Customizável; default `chromium` UA |
| Cookies isolados | Cada execução cria novo `context` (sem cookies persistentes prévios) |
| Network idle threshold | `wait_until="networkidle"` espera 500ms sem requests novos |
| **Browser nativo `browser_network_requests`** | Universal P1 (D-OPERADOR-1) — captura DENTRO do browser engine independente do JS rodar |

---

## §13 — Reference cross-skill

- [`rubrica-tracking-blackbox.md`](rubrica-tracking-blackbox.md) — rubrica das 10 dimensões + scoring + thresholds
- [`emq-formula-meta.md`](emq-formula-meta.md) — fórmula EMQ 2024-2026 detalhada
- [`consent-mode-v2-spec.md`](consent-mode-v2-spec.md) — spec GCM v2 + mudança junho/2026
- [`lgpd-compliance-canonicas.md`](lgpd-compliance-canonicas.md) — LGPD Art. + ANPD guias + severidades
- [`click-ids-taxonomy-2026.md`](click-ids-taxonomy-2026.md) — taxonomia completa + Safari ITP resilience
