# Enhanced Ecommerce GA4 — 11 eventos canônicos + schema items array

> **Reference da skill `tracking-blackbox-auditor`** (modo `ecommerce` exclusivo — D9 = 8% peso) — 11 eventos canônicos GA4 Enhanced Ecommerce + items array schema + currency consistency + transaction_id uniqueness + procedures de validação Playwright.

---

## 1. 11 eventos canônicos GA4 Enhanced Ecommerce

| # | Evento | Quando dispara | Fields required (items array + outros) |
|---|--------|----------------|----------------------------------------|
| 1 | `view_item_list` | Listing/categoria carregada | `items` + `item_list_id` + `item_list_name` |
| 2 | `select_item` | Click em item do listing | `items` + `item_list_id` + `item_list_name` |
| 3 | `view_item` | PDP (product detail page) carregada | `items` + `currency` + `value` |
| 4 | `add_to_cart` | Click "Add to cart" | `items` + `currency` + `value` |
| 5 | `remove_from_cart` | Click "Remove" no cart | `items` + `currency` + `value` |
| 6 | `view_cart` | Cart page aberta | `items` + `currency` + `value` |
| 7 | `begin_checkout` | Entrada no checkout flow | `items` + `currency` + `value` + `coupon` (cond.) |
| 8 | `add_shipping_info` | Etapa shipping no checkout | `items` + `currency` + `value` + `shipping_tier` |
| 9 | `add_payment_info` | Etapa payment no checkout | `items` + `currency` + `value` + `payment_type` |
| 10 | `purchase` | Confirmation page | `items` + `transaction_id` único + `currency` + `value` + `tax` + `shipping` + `coupon` (cond.) |
| 11 | `refund` | Refund processado (server-side typically) | `transaction_id` (same as purchase) + `items` (cond.) |

**Total 11 eventos.** Modo `ecommerce` audita todos.

---

## 2. Items array — schema canônico

Items array é estrutura **compartilhada** entre todos 11 eventos. Cada item objeto suporta até 27 parâmetros:

### 2.1 Required fields

| Field | Tipo | Exemplo |
|---|---|---|
| `item_id` | string (SKU) | `"SKU-12345"` |
| `item_name` | string | `"Tênis Esportivo Premium"` |

### 2.2 Recommended fields

| Field | Tipo | Exemplo | Notas |
|---|---|---|---|
| `affiliation` | string | `"Loja Oficial BR"` | Afiliação/loja |
| `coupon` | string | `"SUMMER2026"` | Cupom aplicado |
| `currency` | string ISO 4217 | `"BRL"` | Moeda |
| `discount` | number | `15.50` | Desconto item-level |
| `index` | integer | `1` | Posição no listing |
| `item_brand` | string | `"Nike"` | Marca |
| `item_category` | string | `"Calçados"` | Categoria nível 1 |
| `item_category2` | string | `"Esportivo"` | Categoria nível 2 |
| `item_category3` | string | `"Running"` | Categoria nível 3 |
| `item_category4` | string | — | Categoria nível 4 |
| `item_category5` | string | — | Categoria nível 5 |
| `item_list_id` | string | `"listing_search"` | ID lista origem |
| `item_list_name` | string | `"Resultado Busca"` | Nome lista origem |
| `item_variant` | string | `"Vermelho_42"` | Variante (cor_tamanho) |
| `location_id` | string | `"loja-rj-001"` | ID localização |
| `price` | number | `299.90` | Preço unitário |
| `quantity` | integer | `1` | Quantidade |

### 2.3 Custom parameters (até 27 extras)

GA4 suporta até **27 custom parameters** por item. Comum:
- `item_size` (XS/S/M/L/XL/XXL)
- `item_color`
- `item_material`
- `is_promotion` (boolean)
- `seller_id` (marketplace)

---

## 3. Exemplo canônico — payload `purchase` completo

```javascript
dataLayer.push({
  event: "purchase",
  ecommerce: {
    transaction_id: "T_12345_2026_05_11",
    affiliation: "Loja Oficial BR",
    value: 599.80,
    tax: 64.78,
    shipping: 15.00,
    currency: "BRL",
    coupon: "PROMO20",
    items: [
      {
        item_id: "SKU-12345",
        item_name: "Tênis Esportivo Premium",
        affiliation: "Loja Oficial BR",
        coupon: "PROMO20",
        currency: "BRL",
        discount: 15.50,
        index: 0,
        item_brand: "Nike",
        item_category: "Calçados",
        item_category2: "Esportivo",
        item_category3: "Running",
        item_list_id: "listing_search",
        item_list_name: "Resultado Busca",
        item_variant: "Vermelho_42",
        location_id: "loja-online",
        price: 299.90,
        quantity: 2
      }
    ]
  }
});
```

---

## 4. Sub-critérios canônicos D9 (auditoria)

### 4.1 Cobertura eventos

| Sub-critério | Threshold |
|---|---|
| `view_item_list` capturado em listing/categoria | Sim/Não |
| `select_item` capturado em click item do listing | Sim/Não |
| `view_item` capturado em PDP | Sim/Não (CRÍTICO — sem isso, conversion path incompleto) |
| `add_to_cart` capturado em click ATC | Sim/Não (CRÍTICO — Meta `AddToCart` paralelo esperado) |
| `view_cart` capturado em cart open | Sim/Não |
| `begin_checkout` capturado em entrada checkout | Sim/Não (CRÍTICO — Meta `InitiateCheckout` paralelo) |
| `add_shipping_info` capturado | Sim/Não |
| `add_payment_info` capturado | Sim/Não |
| `purchase` capturado em confirmation | Sim/Não (BLOCKER — Meta `Purchase` paralelo CRÍTICO) |
| `refund` capturado server-side ou via Customer Service flow | Sim/Não/N-A |

### 4.2 Items array integrity

| Sub-critério | Threshold |
|---|---|
| `item_id` presente em todos items | Required |
| `item_name` presente em todos items | Required |
| `price` numérico em todos items | Required |
| `quantity` integer em todos items | Required |
| `currency` consistente cross-events | "BRL" sempre (não mix BRL+USD) |
| `value` = soma(items.price × items.quantity) - discount | Consistência matemática |
| `transaction_id` único per-transaction | Não-duplicado em refresh página |

### 4.3 Currency consistency

```python
def check_currency_consistency(ecommerce_events):
    """
    Currency deve ser consistente cross-events (sempre BRL pra BR).
    """
    currencies_used = set()
    for evt in ecommerce_events:
        currency = evt.get("ecommerce", {}).get("currency")
        if currency:
            currencies_used.add(currency)
        for item in evt.get("ecommerce", {}).get("items", []):
            if "currency" in item:
                currencies_used.add(item["currency"])
    return {
        "currencies_detected": list(currencies_used),
        "consistent": len(currencies_used) == 1,
        "issue": "Multiple currencies cross-events" if len(currencies_used) > 1 else None,
    }
```

### 4.4 transaction_id uniqueness

```python
def check_transaction_id_uniqueness(purchase_events):
    """
    transaction_id deve ser único per-transaction.
    Duplicação típica: user refresh confirmation page → purchase event 2× com mesmo transaction_id.
    """
    transaction_ids = [e["ecommerce"]["transaction_id"] for e in purchase_events]
    duplicates = [t for t in transaction_ids if transaction_ids.count(t) > 1]
    return {
        "total_purchase_events": len(transaction_ids),
        "unique_transaction_ids": len(set(transaction_ids)),
        "duplicates_detected": list(set(duplicates)),
        "consistent": len(duplicates) == 0,
    }
```

### 4.5 value coherence

```python
def check_value_coherence(purchase_event):
    """
    value = soma(items.price × items.quantity) - discount + shipping + tax
    (depende do modelo — algumas plataformas incluem shipping/tax no value, outras não)
    """
    items = purchase_event["ecommerce"]["items"]
    items_total = sum(i["price"] * i["quantity"] for i in items)
    declared_value = purchase_event["ecommerce"]["value"]
    discount = purchase_event["ecommerce"].get("discount", 0)
    
    # Modelo canônico GA4: value = items.price × quantity - discount item-level
    # shipping + tax são fields separados
    expected_value = items_total - sum(i.get("discount", 0) for i in items)
    
    return {
        "items_total": items_total,
        "declared_value": declared_value,
        "expected_value": expected_value,
        "diff_pct": abs(declared_value - expected_value) / expected_value if expected_value > 0 else 0,
        "coherent": abs(declared_value - expected_value) < 0.01,
    }
```

---

## 5. Cross-platform paralelos canônicos

### 5.1 Meta `Purchase` paralelo

Cliente com Meta Pixel + Google Ads ecommerce **deve** disparar Meta `Purchase` em paralelo ao GA4 `purchase`:

```javascript
fbq('track', 'Purchase', {
  value: 599.80,
  currency: 'BRL',
  content_ids: ['SKU-12345'],
  content_type: 'product',
  num_items: 2,
  contents: [{id: 'SKU-12345', quantity: 2, item_price: 299.90}]
}, {eventID: 'T_12345_2026_05_11'});  // eventID = transaction_id pra dedup
```

### 5.2 Google Ads `purchase` value-based bidding

Google Ads tag deve receber `transaction_id` + `value` pra Enhanced Conversions:

```javascript
gtag('event', 'conversion', {
  send_to: 'AW-XXXXXX/YYYYYY',
  transaction_id: 'T_12345_2026_05_11',
  value: 599.80,
  currency: 'BRL'
});
```

### 5.3 TikTok / Pinterest (cond. vertical)

Ecommerce de moda/beleza/lifestyle pode ter TikTok Pixel + Pinterest Pixel paralelos:
```javascript
// TikTok
ttq.track('CompletePayment', {
  value: 599.80, currency: 'BRL',
  contents: [{content_id: 'SKU-12345', quantity: 2, price: 299.90}]
});

// Pinterest
pintrk('track', 'checkout', {
  value: 599.80, currency: 'BRL',
  line_items: [{product_id: 'SKU-12345', product_quantity: 2, product_price: 299.90}]
});
```

---

## 6. Procedure Playwright — jornada ecommerce

```python
def simulate_ecommerce_journey(page, target_url):
    """
    Modo `ecommerce` — simulação não-destrutiva (NÃO completa purchase real).
    Captura D9 11 eventos.
    """
    # 1. Listing
    page.goto(target_url, wait_until="networkidle")
    listing_url = target_url + "/categoria/calcados" if not target_url.endswith("/") else target_url + "categoria/calcados"
    try:
        page.goto(listing_url, wait_until="networkidle")
    except:
        # Listing pode ser homepage com grid produtos
        pass
    
    # 2. Click primeiro item do listing → PDP
    item_link = page.locator("a[href*='/produto'], a[href*='/p/'], a[href*='/sku'], .product-card a, .product-item a").first
    if item_link.count() > 0:
        item_link.click()
        page.wait_for_load_state("networkidle")
    
    # 3. Add to cart
    add_to_cart_btn = page.locator("button:has-text('Adicionar'), button:has-text('Comprar'), button.add-to-cart, button[data-action='add-to-cart']").first
    if add_to_cart_btn.count() > 0:
        add_to_cart_btn.click()
        page.wait_for_timeout(2000)
    
    # 4. View cart
    cart_link = page.locator("a[href*='cart'], a[href*='carrinho'], button.view-cart").first
    if cart_link.count() > 0:
        cart_link.click()
        page.wait_for_load_state("networkidle")
    
    # 5. Begin checkout
    checkout_btn = page.locator("button:has-text('Finalizar'), button:has-text('Checkout'), button.checkout").first
    if checkout_btn.count() > 0:
        checkout_btn.click()
        page.wait_for_load_state("networkidle")
    
    # 6. NÃO PROSSEGUIR PRA PAYMENT — skill é não-destrutiva
    # Auditoria captura eventos até begin_checkout
    
    return {
        "events_captured": page.evaluate("() => window._auditPushes || []"),
    }
```

---

## 7. Anti-patterns canônicos D9

| Anti-pattern | Severidade |
|---|---|
| `purchase` sem `transaction_id` | **BLOCKER técnico** — quebra dedup |
| `transaction_id` duplicado em refresh | **Alta** — infla revenue 2× |
| `currency` inconsistente cross-events | **Alta** — quebra reports |
| `value` ≠ soma items | **Média** — discrepância downstream |
| Meta `Purchase` paralelo ausente | **Alta** — Meta Ads otimização sub-ótima |
| Google Ads `purchase` sem value-based bidding setup | **Média** — perde Smart Bidding optimization |
| Items array vazio em `purchase` | **BLOCKER técnico** — Enhanced Ecommerce incompleto |
| `item_id` ausente em items | **BLOCKER técnico** — sem SKU = não match com Merchant Center |
| Refund event ausente em refund flow | **Média** — revenue inflada |
| Currency hardcoded "USD" em loja BR | **Alta** — reports zonzos |

---

## 8. Scoring D9 (rubrica modo `ecommerce`)

| Score | Critério |
|---|---|
| 10 | 11 eventos canônicos cobertos + items array completo + currency consistente + transaction_id único + Meta Purchase paralelo + Google Ads value-based bidding setup |
| 7-9 | ≥9/11 eventos capturados + items array integrity + transaction_id único; gaps em refund OR Meta paralelo OR Google Ads VBB |
| 4-6 | ≥6/11 eventos + items array com gaps (item_id/item_name presente; outros campos missing) OR transaction_id duplicado eventual |
| 1-3 | <6/11 eventos OR items array vazio em purchase OR transaction_id ausente |
| 0 | Enhanced Ecommerce ausente — só pageview e custom events |

---

## 9. References externas

- [Google — Measure Ecommerce GA4](https://developers.google.com/analytics/devguides/collection/ga4/ecommerce)
- [Google — Recommended Events GA4](https://developers.google.com/analytics/devguides/collection/ga4/reference/events)
- [Google — Set up Ecommerce Events](https://support.google.com/analytics/answer/12200568)
- [Simo Ahava — GA4 Ecommerce Guide for GTM](https://www.simoahava.com/analytics/google-analytics-4-ecommerce-guide-google-tag-manager/)
- [Propellernet — Data Layers for GA4 Ecommerce](https://www.propellernet.co.uk/insights/data-layers-for-enhanced-ecommerce-guide/)
- [Napkyn — GA4 Ecommerce Tracking Setup Without Data Loss](https://www.napkyn.com/blog/how-to-set-up-ga4-ecommerce-tracking)
