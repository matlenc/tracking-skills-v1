# EMQ — Event Match Quality Meta 2024-2026

> **Reference da skill `tracking-blackbox-auditor`** — fórmula EMQ Meta + targets canônicos + benchmarks por evento + impacto por parâmetro.
>
> **Cutoff:** estado-da-arte 2026. Score Meta calculado nas últimas 48h de dados (flutua com traffic patterns).

---

## 1. O que é EMQ

**Event Match Quality (EMQ)** é o score 0-10 que Meta atribui a cada evento enviado via Conversions API (CAPI) baseado em quantidade + qualidade dos parâmetros de customer data inclusos no payload.

EMQ alto = matching melhor = atribuição mais precisa = otimização Meta mais eficaz.

---

## 2. Fórmula canônica (estimativa client-side)

```
EMQ_score = base(2.0)
          + email_match: 0.0-2.0    # ud[em] presente + hash SHA-256 válido
          + phone_match: 0.0-2.0    # ud[ph] presente + hash SHA-256 válido
          + name_match (fn+ln): 0.0-1.5    # ud[fn] + ud[ln]
          + address_match (ct+st+zp+country): 0.0-1.0
          + external_id: 0.0-1.5    # ud[external_id]
          + client_ip_address: 0.0-0.5
          + client_user_agent: 0.0-0.5
          + fbc (fbclid persistido): 0.0-1.0
          + fbp (Meta first-party cookie): 0.0-0.5
```

**Total teórico máximo:** 12.5 — mas Meta cap em 10. Score real depende de:
- Quality do hash (normalização correta)
- Cobertura cross-events (mesmo external_id reuso)
- Frescor (dados recentes valem mais)

---

## 3. Impacto por parâmetro (Meta documentation 2024-2026)

| Parâmetro | Impacto canônico | Notas |
|---|---|---|
| **Email hashado (`em`)** | +2.0 (até 4 pontos cumulativos com phone + name + address) | **Highest impact** — adicionar email aumenta EMQ em até 4 pontos |
| **Phone hashado (`ph`)** | +2.0 (~3 pontos típicos) | E.164 obrigatório (`+5511999999999` BR) |
| **Name (fn + ln)** | +1.5 cumulativo | lowercase + remove acentos + trim |
| **Address (ct + st + zp + country)** | +1.0 cumulativo | ct = cidade lowercase; zp = numérico; country = ISO 3166 alpha 2 |
| **external_id** | +1.5 | UUID v4 ou hash CRM ID (mesmo cross-events!) |
| **client_ip_address** | +0.5 | Server-side only — não enviar do browser |
| **client_user_agent** | +0.5 | Server-side preferred |
| **fbc** | +1.0 | fbclid persistido em cookie `_fbc` |
| **fbp** | +0.5 | Meta first-party cookie automaticamente setado |

---

## 4. Targets canônicos por evento (Meta benchmarks 2024-2026)

| Evento | Target EMQ "Good" | Target Excellent |
|---|---|---|
| **Purchase** | 8.8+ | 9.3+ |
| **InitiateCheckout** | 8.5+ | 9.0+ |
| **AddToCart** | 8.0+ | 8.8+ |
| **Lead** | 7.0+ | 8.0+ |
| **CompleteRegistration** | 7.0+ | 8.0+ |
| **ViewContent** | 6.5+ | 7.5+ |
| **PageView** | 6.5-7.5 | 8.0+ |
| **Search** | 6.0+ | 7.0+ |

**Threshold "Good" geral:** ≥7.0 (Meta categoriza: <5.0 Poor / 5.0-6.9 Fair / 7.0+ Good).

---

## 5. Normalização canônica ANTES do hash

Hash SHA-256 hex lowercase. **Normalização na origem, NÃO downstream** (garantir hash bate em qualquer estágio).

| Campo | Normalização canônica | Test sintético |
|---|---|---|
| `email` | lowercase + trim | `test@example.com` → SHA-256 = `973dfe463ec85785f5f95af5ba3906eedb2d931c24e69824a89ea65dba4e813b` |
| `phone` | E.164 (`+55` prefix BR) numeric only | `+5511999999999` → SHA-256 hex |
| `first_name` / `last_name` | lowercase + trim + remove_acentos | `josé` → `jose` |
| `city` | lowercase + remove_acentos | `são paulo` → `sao paulo` |
| `state` | lowercase 2-char ISO | `SP` → `sp` |
| `zipcode` | numeric_only (5 digits BR minimum) | `01310-100` → `01310100` |
| `country` | ISO 3166 alpha 2 lowercase | `Brasil` → `br` |
| `gender` | `m` / `f` lowercase | M/F única letra |
| `date_of_birth` | `YYYYMMDD` | `1990-05-15` → `19900515` |

### 5.1 Test bench canônico

```python
import hashlib

def test_normalization():
    cases = [
        ("test@example.com", "email", "973dfe463ec85785f5f95af5ba3906eedb2d931c24e69824a89ea65dba4e813b"),
        ("Test@Example.com  ", "email", "973dfe463ec85785f5f95af5ba3906eedb2d931c24e69824a89ea65dba4e813b"),  # mesmo hash após trim+lowercase
        ("+5511999999999", "phone", hashlib.sha256("+5511999999999".encode()).hexdigest()),
        ("(11) 99999-9999", "phone", hashlib.sha256("+5511999999999".encode()).hexdigest()),  # mesmo hash após normalização E.164
    ]
    return cases
```

---

## 6. Categorias EMQ Meta

| Faixa | Categoria | Action |
|---|---|---|
| 9.0-10.0 | **Excellent** | Manter; benchmark interno |
| 7.0-8.9 | **Good** | Manter; refinar campos missing |
| 5.0-6.9 | **Fair** | Melhorar — adicionar email/phone hashados |
| <5.0 | **Poor** | Crítico — remediation P1 imediato |

---

## 7. Anti-patterns EMQ canônicos

| Anti-pattern | Por quê |
|---|---|
| ❌ Send fewer fields with high quality vs many low-quality | Princípio: "It's better to send fewer, highly accurate parameters than many inaccurate" |
| ❌ Email não normalizado ANTES do hash | Hash não bate downstream — Meta não consegue match |
| ❌ Phone não-E.164 antes do hash | Mesmo problema — `(11) 99999-9999` ≠ `+5511999999999` em hash |
| ❌ Email plain text em payload Meta CAPI | **BLOCKER LGPD Art. 5º/7º + Meta CAPI spec violation** |
| ❌ external_id diferente por evento | Quebra cross-event matching — usar mesmo external_id sempre |
| ❌ Confiar em Pixel browser sozinho (sem CAPI) | EMQ Pixel browser ~3-4 pontos abaixo de CAPI server-side (iOS 14 ATT + ITP) |
| ❌ Pre-hash em campos que Meta hasheia internamente (`ud[*]` Advanced Matching) | Quebra matching — Meta hasheia internamente Advanced Matching |

---

## 8. Como melhorar EMQ — playbook

### 8.1 Quick wins (+1 a +3 pontos)

1. **Adicionar email hashado** em todos eventos críticos (+2 pontos)
2. **Adicionar phone hashado** (+2 pontos)
3. **Persistir fbclid em cookie `_fbc`** (+1 ponto)
4. **Enriquecer com fn + ln** (+1.5 ponto)
5. **Adicionar external_id consistente cross-events** (+1.5 ponto)

### 8.2 Mudanças estruturais (+2 a +4 pontos)

1. **Migrar pra Meta CAPI server-side** (Stape / Cloud Run / N8N) — EMQ +2-3 pontos
2. **Implementar dedup com event_id UUID v4** consistente browser↔server
3. **Hash policy server-side** (não confiar em Advanced Matching browser-side)
4. **Address fields completos** (city + state + zipcode + country) — +1 ponto

---

## 9. Mensuração EMQ real (Nível 4 Meta Events Manager)

Operador acessa Meta Business → Events Manager → Pixel ID → "Diagnostics" tab → vê EMQ score real por evento últimas 48h.

Auditoria Nível 1 estima via fórmula client-side; confidence `media`. Nível 4 confirma valor real com margem ±1 ponto.

---

## 10. CAPI gateway impact

CAPI gateway (Stape / Cloud Run / N8N) adiciona:
- `client_ip_address` server-side (+0.5)
- `client_user_agent` server-side (+0.5)
- Dedup automático via `event_id`
- Hash normalização server-side garantida

**Lift típico:** +1.5 a +2.5 pontos EMQ vs Pixel browser only.

---

## 11. Limitações estimativa client-side

- ❌ Não vê parâmetros que Meta hasheia internamente após Pixel browser
- ❌ Não vê AEM priority do Pixel ID
- ❌ Não vê eventos rejeitados pelo Meta (data quality issues)
- ❌ Não vê event source URL setup
- ✅ Vê tudo do payload `/tr` browser
- ✅ Vê dedup `event_id` browser↔server (se network monitor captura)
- ✅ Vê hash quality via test sintético

---

## 12. References externas

- [Meta — About Event Match Quality](https://www.facebook.com/business/help/2364892187272978)
- [Meta — Best Practices for Customer Information Parameters](https://developers.facebook.com/docs/marketing-api/conversions-api/parameters/customer-information-parameters)
- [Meta — Improving Your Match Score](https://www.facebook.com/business/help/765081237991954)
- [Stape — Event Deduplication](https://stape.io/helpdesk/documentation/deduplication-config)
