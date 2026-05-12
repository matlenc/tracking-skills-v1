# Guia de Configuração GTM — Referência Operacional
> Carregado sob demanda pelo Tracking Engineer nas Fases 2 e 4.

---

## Seção 1 — Variáveis GTM

### Nomenclatura padrão por grupo

| Grupo | Prefixo | Exemplo |
|---|---|---|
| Permanentes editáveis | `[EDIT] Perma |` | `[EDIT] Perma | GA4 ID` |
| DataLayer | `DLV |` | `DLV | form_id` |
| Derivadas (ambiente) | `Derived |` | `Derived | Page URL` |
| User Data (hash) | `UD |` | `UD | Email Hash` |
| Cookie / Storage | `CKV |` | `CKV | fbclid` |

---

### Grupo 1 — Permanentes Editáveis `[EDIT] Perma |`

Tipo GTM: **Constante**. O operador substitui os valores `X` pelos IDs reais do projeto.

| Nome da Variável | Valor a preencher |
|---|---|
| `[EDIT] Perma | GA4 ID` | `G-XXXXXXXX` |
| `[EDIT] Perma | Meta Pixel ID` | `XXXXXXXXXX` |
| `[EDIT] Perma | GAds Conversion ID` | `AW-XXXXXXXXX` |
| `[EDIT] Perma | Clarity ID` | `XXXXXXXXXX` |
| `[EDIT] Perma | GAds Label | Lead` | label de conversão Lead |
| `[EDIT] Perma | GAds Label | MQL` | label de conversão MQL |
| `[EDIT] Perma | GAds Label | NOICP` | label de conversão NOICP (se no scope) |

> Adicione apenas labels para eventos que estão no scope do projeto. Não crie labels especulativos.

---

### Grupo 2 — DataLayer `DLV |`

Tipo GTM: **Data Layer Variable**. O nome no campo "Data Layer Variable Name" deve ser idêntico ao do `dataLayer.push()` — case-sensitive.

**Identidade do evento:**
| Nome da Variável | Data Layer Variable Name |
|---|---|
| `DLV | event_id` | `event_id` |
| `DLV | event_time` | `event_time` |

**Formulário:**
| Nome da Variável | Data Layer Variable Name |
|---|---|
| `DLV | form_id` | `form_id` |
| `DLV | form_name` | `form_name` |
| `DLV | form_type` | `form_type` |

**Qualificação:**
| Nome da Variável | Data Layer Variable Name |
|---|---|
| `DLV | lead_status` | `lead_status` |
| `DLV | qualification_rule` | `qualification_rule` |
| `DLV | qualification_reason` | `qualification_reason` |

**User data** (inclua somente se o handoff declara user data no evento):
| Nome da Variável | Data Layer Variable Name |
|---|---|
| `DLV | user.email` | `user.email` |
| `DLV | user.phone` | `user.phone` |
| `DLV | user.first_name` | `user.first_name` |

**Atribuição:**
| Nome da Variável | Data Layer Variable Name |
|---|---|
| `DLV | attribution.utm_source` | `attribution.utm_source` |
| `DLV | attribution.utm_medium` | `attribution.utm_medium` |
| `DLV | attribution.utm_campaign` | `attribution.utm_campaign` |
| `DLV | attribution.utm_content` | `attribution.utm_content` |
| `DLV | attribution.fbclid` | `attribution.fbclid` |
| `DLV | attribution.gclid` | `attribution.gclid` |

---

### Grupo 3 — Derivadas `Derived |`

Tipo GTM: **URL** ou **JavaScript Variable**

| Nome da Variável | Tipo | Componente |
|---|---|---|
| `Derived | Page URL` | URL | Full URL |
| `Derived | Page Path` | URL | Path |
| `Derived | Page Hostname` | URL | Hostname |
| `Derived | Referrer` | JS Variable | `document.referrer` |

---

### Grupo 4 — User Data Hasheado `UD |`

Tipo GTM: **Custom JavaScript Variable**

Necessário para: Meta CAPI (Advanced Matching), Google Ads Enhanced Conversions.

**Regras de normalização obrigatória antes do hash:**
- Email: converter para lowercase, remover espaços
- Phone: formato E.164 com código do país (`+5511999999999`)
- Executar SHA256 após normalização

```javascript
// UD | Email Hash
function() {
  var email = {{DLV | user.email}};
  if (!email) return undefined;
  email = email.toLowerCase().trim();
  // SHA256 via CryptoJS ou implementação nativa
  return CryptoJS.SHA256(email).toString();
}
```

> Se o projeto usa um template de hash já instalado no GTM (ex: template oficial do Google para Enhanced Conversions), use-o e referencie a variável gerada ao invés de criar Custom JS.

---

## Seção 2 — Configuração de Tags por Plataforma

### 2.1 GA4 — Event Tag

**Tipo GTM:** Google Analytics: GA4 Event (ou via Google tag se configuração unificada)

Para cada evento canônico, crie uma tag:

```
Tag: TAG | GA4 | [EventoCanônico]
Measurement ID: {{[EDIT] Perma | GA4 ID}}
Event Name: [EventoCanônico]   ← exato, case-sensitive
Parameters:
  event_id    → {{DLV | event_id}}
  form_id     → {{DLV | form_id}}         (quando aplicável)
  lead_status → {{DLV | lead_status}}     (quando aplicável)
  qualification_rule → {{DLV | qualification_rule}}  (MQL/NOICP)
  utm_source  → {{DLV | attribution.utm_source}}
  utm_campaign → {{DLV | attribution.utm_campaign}}
Trigger: TRG | Custom Event | [EventoCanônico]
```

> QA: valide no GA4 DebugView (ative via GTM Preview — o debug_mode é setado automaticamente).
> Se Internal Traffic filter estiver ativo, eventos do DebugView não aparecem nos relatórios padrão — isso é comportamento esperado.

---

### 2.2 Meta Pixel — Browser Tag

**Tipo GTM:** HTML Customizado

```html
<script>
!function(f,b,e,v,n,t,s){...}(window, document,'script',
'https://connect.facebook.net/en_US/fbevents.js');
fbq('init', '{{[EDIT] Perma | Meta Pixel ID}}', {
  em: '{{UD | Email Hash}}',
  ph: '{{UD | Phone Hash}}'
});
</script>
```

**Tag de evento (Lead):**
```html
<script>
fbq('track', 'Lead', {
  eventID: '{{DLV | event_id}}'
});
</script>
```

**Tag de evento customizado (MQL):**
```html
<script>
fbq('trackCustom', 'MQL', {
  eventID: '{{DLV | event_id}}',
  lead_status: '{{DLV | lead_status}}'
});
</script>
```

> `eventID` é obrigatório para deduplicação com CAPI. Deve ser idêntico ao `event_id` enviado pelo server. Ausência de `eventID` é causa de 80% dos erros de deduplicação Meta.
>
> QA: use Meta Events Manager → Test Events com o Test Event Code inserido na tag (campo adicional no HTML).

### 2.2.5 CAPI server-side via N8N (estado da arte sem SGTM)

Quando não houver SGTM (Server-Side GTM), Meta CAPI deve ser implementado via N8N.

**Templates canônicos (use sempre como base — não reescreva do zero):**
- HTML da tag GTM: `templates/capi-forwarder-tag.html`
- Workflow N8N importável: `templates/n8n-workflow-meta-capi.json`

Os templates já contêm guards JS, allowlist server-side, SHA-256 puro JS, normalização phone Meta, parse de body string e log de resposta.

**Arquitetura:**
```
GTM Web (Custom HTML CAPI Forwarder) ──sendBeacon (string)──► N8N webhook ──parse + hash SHA256 + Switch──► graph.facebook.com/v23.0/{PIXEL}/events
```

**Tag GTM "TAG | Meta | CAPI Forwarder":**
- HTML Customizado (template em `templates/capi-forwarder-tag.html`)
- Trigger: múltiplos triggers Custom Event (Lead, MQL, NOICP — os mesmos que disparam Pixel browser)
- Lê `{{Event}}` pra determinar event_name dinamicamente
- Envia via `navigator.sendBeacon(endpoint, body)` com STRING (não Blob — ver pegadinha abaixo) com fallback `fetch` keepalive
- Payload contém: `event_name`, `event_id` (DLV — MESMO do Pixel browser pra dedup), `event_time`, `event_source_url`, `user_data` (em/ph/fn plain + fbp/fbc/client_user_agent), `custom_data`

**Defense in depth (3 camadas) — obrigatório pra CAPI Forwarder:**
1. Trigger no GTM com `Some Custom Events` + filter `Event = Lead/MQL/NOICP` (ver Pegadinha "All Custom Events" abaixo)
2. Guard JS no início da tag: `var evt = {{Event}}; if (evt !== 'Lead' && evt !== 'MQL' && evt !== 'NOICP') return;`
3. Allowlist no Code node N8N: `if (!ALLOWED_EVENTS.includes(input.event_name)) return { json: { skipped: true, ... } };` + Switch node com 3 outputs (Lead/MQL/NOICP) convergindo no HTTP Request

**Workflow N8N:**
1. **Webhook node** recebe POST
2. **Function node** normaliza phone E.164 + hashea SHA256 todos os campos PII (`em`, `ph`, `fn`)
3. **HTTP Request node** POST → `https://graph.facebook.com/v19.0/{{$env.META_PIXEL_ID}}/events?access_token={{$env.META_CAPI_ACCESS_TOKEN}}`

**Variáveis de ambiente N8N (server-side, NUNCA no GTM/front):**
- `META_PIXEL_ID`
- `META_CAPI_ACCESS_TOKEN` (sensível — env var only)
- `META_TEST_EVENT_CODE` (opcional, debug)

**Anti-pattern:** colocar token CAPI em variável GTM (mesmo "[EDIT] Perma"). Token CAPI é credencial de write na conta Meta — vaza no front, qualquer um pode injetar conversões falsas.

**Validação:** Meta Events Manager → Test Events deve mostrar cada evento chegando 2× (Pixel + CAPI) com badge **"Deduplicated"** quando event_id é idêntico.

**Pegadinha CRÍTICA — sendBeacon: passar STRING, NUNCA Blob `application/json`:**

`navigator.sendBeacon(url, data)` aceita string ou Blob. A escolha tem consequência grave em request cross-origin:

- ✅ **`navigator.sendBeacon(url, body)` com STRING** → Content-Type vira `text/plain;charset=UTF-8` (simple Content-Type) → SEM CORS preflight → request chega ao destino.
- ❌ **`navigator.sendBeacon(url, new Blob([body], {type: 'application/json'}))`** → Content-Type vira `application/json` (non-simple) → browser tenta CORS preflight → **sendBeacon NÃO suporta preflight** → request DESCARTADO SILENCIOSAMENTE, sem erro JS, sem log no console. O destino nunca recebe.

Sintoma: tag dispara no Tag Assistant Preview (interceptor confirma 700+ bytes saindo do browser), mas N8N nunca recebe execução. Diagnóstico fácil: troca pra string, funciona imediatamente.

**Pattern correto (já aplicado no template em `templates/capi-forwarder-tag.html`):**
```js
var body = JSON.stringify(payload);
if (navigator.sendBeacon) {
  navigator.sendBeacon(endpoint, body);  // string, sem preflight
} else {
  fetch(endpoint, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: body,
    keepalive: true
  });
}
```

**Consequência no servidor:** body chega como STRING (não objeto JSON parsed). Solução obrigatória no primeiro Code node após o webhook:
```js
let input = $input.item.json.body || $input.item.json;
if (typeof input === 'string') {
  try { input = JSON.parse(input); } catch (e) { input = {}; }
}
```

Mesmo padrão se aplica a qualquer webhook que recebe `sendBeacon`. `fetch` com `Content-Type: application/json` explícito é parseado automaticamente pelo N8N, mas dispara preflight se cross-origin — usar `keepalive: true` apenas como fallback de browsers sem `sendBeacon`.

**Por que usar `sendBeacon` mesmo assim:** é otimizado pra envios "fire-and-forget" no unload da página (form submit + redirect imediato). `fetch keepalive` pode perder request se browser fechar antes de completar.

**Pegadinha — Triggers Custom Event "Todos os eventos personalizados" (anti-pattern):**

Trigger Custom Event configurado como "Este acionador é disparado em: **Todos os eventos personalizados**" (sem condição de filtro de nome) DISPARA em eventos internos do GTM como `gtm.init`, `gtm.init_consent`, `gtm.dom`, `gtm.js`, `gtm.load` — porque GTM trata esses como custom events.

Em tags GA4/Clarity é tolerável (filtros internos próprios). Em **tag CAPI Forwarder é CRÍTICO** — gera lixo no Meta Graph API com event_name = "gtm.dom" e polui Pixel Diagnostics.

**Sempre usar:** "Alguns eventos personalizados" + condição `{{Event}} é igual a Lead` (e MQL, NOICP).

Containers herdados costumam ter esse anti-pattern. Antes de adicionar tag CAPI Forwarder a um container existente, rodar a auditoria documentada em `guias/checklist-auditoria-container-herdado.md` Item 1.

**Pegadinha — Tag Assistant Preview Mode polui o destino:**

GTM Tag Assistant Preview Mode carrega o site dentro de iframe `https://gtm-msr.appspot.com/render?id=<container>` e dispara eventos internos do GTM lá. Tags ativas no workspace de Preview disparam ali e mandam dados pra produção (webhook N8N, etc) — que NÃO correspondem ao comportamento real. Heurística de debug: se o destino recebe execuções com `event_source_url = gtm-msr.appspot.com/render?...`, é Preview Mode rodando — não é bug da tag.

**Validação E2E correta (com Preview FECHADO):** ver `guias/playbook-validacao-e2e-server-side.md`.

**Outra pegadinha — hash em N8N Code node:**
N8N Cloud E muitos self-host bloqueiam `require()`, `crypto.subtle`, e até `TextEncoder` no sandbox VM2. **Solução universal:** SHA256 em puro JavaScript (~50 linhas inline, síncrono, zero deps). Funciona em qualquer ambiente. Template no doc do projeto Aquatro: `docs/tracking/n8n-meta-capi-aquatro.json` Code node "Hash & Build Payload". Decision tree:

1. Tenta `crypto.subtle.digest()` (Web Crypto API) — funciona em N8N Cloud + Node 18+
2. Se "TextEncoder is not defined" ou "crypto.subtle is undefined" → fallback pra SHA256 puro JS inline
3. NUNCA tente `require('crypto')` — bloqueado em todos os N8N modernos por sandbox

A v3 do workflow Meta CAPI no projeto Aquatro usa SHA256 puro JS por causa dessa restrição.

**Pegadinha — phone normalization difere entre Google e Meta:**
- **Google Ads Enhanced Conversions:** E.164 com `+` (ex: `+5544999887766`)
- **Meta CAPI:** dígitos puros sem `+` (ex: `5544999887766`)

Mesmo telefone, hash diferente. Não compartilhe a função `normalizePhone()` entre os 2 — crie `normalizePhoneForGoogle()` e `normalizePhoneForMeta()` separadas. O front pode normalizar pra E.164 (Google), e o N8N Code node strippa o `+` antes de hashear pra Meta CAPI.

**Pegadinha — `client_user_agent` em CAPI vem do front, não do header N8N:**
`sendBeacon` no GTM mantém o user-agent original do navegador, mas request via reverse proxy (Cloudflare, Nginx) muitas vezes adiciona/altera `User-Agent` header. Pra match correto Meta-Pixel ↔ Meta-CAPI, o `client_user_agent` deve ser **`navigator.userAgent` capturado no front** e enviado no payload do CAPI Forwarder, NÃO `headers['user-agent']` no N8N. Fallback: tenta primeiro `ud.client_user_agent` (do front), depois `headers['user-agent']`.

**Pegadinha — proteção `__SET_ME__` no CAPI Forwarder GTM:**
Antes de enviar `fetch()` pro endpoint N8N, **valide se a variável `[EDIT] Perma | Meta CAPI Endpoint` foi configurada**:
```js
var endpoint = {{[EDIT] Perma | Meta CAPI Endpoint}};
if (!endpoint || endpoint === '__SET_ME__') return;
```
Sem essa proteção, se o operador esquecer de atualizar a variável após import, o GTM dispara `fetch('__SET_ME__', ...)` em todos os eventos — gera 404 nos servers do operador (poluindo logs) e nada chega no Meta.

**Pegadinha — builders com AJAX próprio (GreatPages, alguns ClickFunnels) ignoram hidden fields criados via JS:**
Hidden fields criados dinamicamente via `document.createElement('input')` + `appendChild` NÃO são enviados quando o builder usa AJAX próprio que serializa apenas campos cadastrados no editor (formato `{campos: [{id, titulo, valor}]}`). **Solução:** cadastrar manualmente os 15 campos `aq_*` (user_id, gclid, utm_*, fbp, fbc, etc.) no editor do builder como "Campo oculto"/"Hidden field". O snippet JS só popula `input.value` dos fields que JÁ EXISTEM no DOM. Validação: `Ctrl+U` no source ou inspecionar payload do POST do form (Network tab).

**Correspondência eventos canônicos → Meta (recomendação 2025-2026):**

| Evento canônico | Meta event_name | Tipo Meta | Por quê |
|---|---|---|---|
| `PageView` | `PageView` | Standard (`fbq('track', ...)`) | Standard event nativo |
| `Lead` | `Lead` | Standard (`fbq('track', ...)`) | Conversão de funil meio (entry) |
| `MQL` | **`MQL`** | **Custom (`fbq('trackCustom', ...)`)** | Mantém nomenclatura canônica, evita dupla contagem se também tiver `Lead` |
| `NOICP` | **`NOICP`** | **Custom (`fbq('trackCustom', ...)`)** | Custom Audience de exclusão direto pelo nome do evento |
| `DealWon` | `Purchase` | Standard (`fbq('track', ...)` com `value`, `currency`) | Standard event de compra |

**Anti-pattern:** mapear `MQL` como `Lead` Standard com `custom_data.lead_status=mql` causa Meta contar 2× conversões "Lead" por usuário (Lead canônico + MQL canônico ambos viram `Lead` Meta). Use Custom Event pra MQL/NOICP, sempre.

**Configuração Custom Conversion no Meta Ads Manager:**
- Events Manager → Custom Conversions → Create
- Source: Pixel
- Event: nome do Custom Event (`MQL`, `NOICP`)
- Category: `Lead` (afeta Smart Bidding como conversão padrão)

**Custom Audiences via Custom Events:**
- Lookalike base: Custom Event `MQL` últimos 180 dias
- Exclusion audience: Custom Event `NOICP` últimos 180 dias

---

### 2.3 Google Ads — Conversion Tracking Tag

**Tipo GTM:** Google Ads Conversion Tracking (`awct`)

**Realidade técnica:** Enhanced Conversions depende da variável "User-Provided Data" (UPD), que é **Community Template** instalado da Gallery — NÃO tipo built-in. Por isso parte da configuração precisa ser manual no UI (3-5 min).

**No JSON principal (parte automatizável):**
```
Tag: TAG | GAds | Conversion | [EventoCanônico]
Conversion ID: {{[EDIT] Perma | GAds Conversion ID}}
Conversion Label: {{[EDIT] Perma | GAds Label | [EventoCanônico]}}
Currency Code: BRL
Conversion Linker: enabled
Trigger: TRG | Custom Event | [EventoCanônico]
```

```json
{ "type": "TEMPLATE", "key": "conversionId", "value": "{{[EDIT] Perma | GAds Conversion ID}}" },
{ "type": "TEMPLATE", "key": "conversionLabel", "value": "{{[EDIT] Perma | GAds Label | [Evento]}}" },
{ "type": "TEMPLATE", "key": "currencyCode", "value": "BRL" },
{ "type": "BOOLEAN", "key": "enableConversionLinker", "value": "true" }
```

**No setup manual pós-import (Enhanced Conversions — exige UI):**
1. Instalar Community Template "Google User-Provided Data" da Gallery
2. Criar variável `UD | <Project> User Data` (tipo "User-Provided Data") com:
   - Email = `{{DLV | user.email}}`
   - Phone Number = `{{DLV | user.phone}}`
   - First Name = `{{DLV | user.first_name}}`
3. Editar cada tag Google Ads → marcar **"Include user-provided data from your website"** → selecionar a variável
4. Save + Submit + Publish

> **NÃO tente incluir `enableEnhancedConversions: true` + `userDataVariable: {{UD | ...}}` no JSON.** A variável referenciada (`awsl`) não é importável. Sempre falha com "Tipo de entidade desconhecido". Documentar o setup manual é o único caminho honesto.
>
> QA: verifique em Google Ads → Conversões → aba "Enhanced Conversions" — status "Recording user-provided data" ou similar.

---

### 2.4 Clarity — Inicialização + Custom Tags (estado da arte)

**Tag de inicialização (All Pages):**
```
Tag: TAG | Clarity | Init
Tipo: HTML Customizado
```
```html
<script type="text/javascript">
    (function(c,l,a,r,i,t,y){
        c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
        t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
        y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
    })(window, document, "clarity", "script", "{{[EDIT] Perma | Clarity ID}}");
</script>
```
Trigger: All Pages

**Custom Tags — OBRIGATÓRIO quando há eventos de qualificação no scope.** Permite filtrar sessões no Clarity por dimensão de qualificação (lead/mql/noicp), debugar UX por persona.

```
TAG | Clarity | Custom Tags | Lead   (trigger: Lead)
TAG | Clarity | Custom Tags | MQL    (trigger: MQL)
TAG | Clarity | Custom Tags | NOICP  (trigger: NOICP)
```

```html
<script>
if (window.clarity) {
  clarity('set', 'lead_status', {{DLV | lead_status}});
  clarity('set', 'qualification_reason', {{DLV | qualification_reason}});
  clarity('set', 'is_icp', '{{DLV | is_icp}}');
}
</script>
```

> Custom Tags do Clarity é trivial de implementar e tem alto valor pra debug. NÃO deixe pro roadmap se há eventos de qualificação no scope.

### 2.5 Consent Mode v2 — Default Tag (OBRIGATÓRIO em todo deploy inicial)

**Tag de Consent Default (All Pages, prioridade 100 pra disparar antes das outras):**

```
Tag: TAG | Consent | Default
Tipo: HTML Customizado
Tag firing priority: 100  (importante: dispara antes das outras tags)
Trigger: All Pages
```

**Quando NÃO há cookie banner ainda (modo conservador, default `granted`):**

```html
<script>
window.dataLayer = window.dataLayer || [];
function gtag(){dataLayer.push(arguments);}

gtag('consent', 'default', {
  'ad_storage': 'granted',
  'ad_user_data': 'granted',
  'ad_personalization': 'granted',
  'analytics_storage': 'granted',
  'functionality_storage': 'granted',
  'personalization_storage': 'granted',
  'security_storage': 'granted',
  'wait_for_update': 500
});

gtag('set', 'ads_data_redaction', false);
gtag('set', 'url_passthrough', true);
</script>
```

**Quando há cookie banner LGPD-compliant instalado (default `denied`):**
- Trocar todos `'granted'` por `'denied'` (exceto `'security_storage'` que pode ficar granted)
- Configurar o banner pra fazer `gtag('consent', 'update', {...})` quando usuário aceitar

> Consent Mode v2 é estado da arte mesmo sem banner. Sinaliza ao Google que consent foi obtido, e prepara o terreno pra Conversion Modeling quando ligar `denied` default. NÃO omita esta tag do deploy inicial. Roadmap: ligar banner LGPD + transicionar pra denied default quando volume justificar.

---

## Seção 3 — Padrão de Nomenclatura de Tags e Gatilhos

| Elemento | Padrão | Exemplo |
|---|---|---|
| Tag GA4 | `TAG \| GA4 \| [Evento]` | `TAG \| GA4 \| Lead` |
| Tag Meta | `TAG \| Meta \| [Evento]` | `TAG \| Meta \| MQL` |
| Tag Google Ads | `TAG \| GAds \| Conversion \| [Evento]` | `TAG \| GAds \| Conversion \| Lead` |
| Tag Clarity init | `TAG \| Clarity \| Init` | — |
| Gatilho evento | `TRG \| Custom Event \| [Evento]` | `TRG \| Custom Event \| Lead` |
| Gatilho All Pages | `TRG \| All Pages` | — |

---

## Seção 4 — Checklist Pré-Publicação GTM

Antes de publicar o container, verifique:
- [ ] Todas as variáveis `[EDIT]` foram preenchidas com valores reais
- [ ] Nenhuma variável DLV tem nome divergente do dataLayer (case-sensitive)
- [ ] Todos os gatilhos usam nomes canônicos de evento
- [ ] **Triggers Custom Event configurados como "Some Custom Events" + filter `Event = X` (NUNCA "All Custom Events")**
- [ ] **Variáveis built-in usadas em tags custom (`{{Event}}`, `{{Page URL}}`, etc.) estão habilitadas em Variables → Built-In Variables → Configure**
- [ ] Tags de Meta têm `eventID` mapeado para `DLV | event_id`
- [ ] Tags de Google Ads com Enhanced Conversions têm hash configurado
- [ ] Tag de inicialização Clarity está em All Pages
- [ ] **Tags Custom HTML com sendBeacon usam STRING (não Blob) — pra evitar CORS preflight**
- [ ] QA foi executado e aprovado nas plataformas (com Tag Assistant Preview FECHADO — ver `playbook-validacao-e2e-server-side.md`)
- [ ] Não há lógica de qualificação em nenhum gatilho ou tag

---

## Seção 5 — Template JSON para Importação de Container GTM

### Quando gerar JSON importável

**SEMPRE gere JSON importável.** O operador espera um arquivo `.json` pronto para importar no GTM, não instruções manuais. O JSON é o artefato principal do Tracking Engineer.

O JSON deve ser importável via GTM → Admin → Import Container.

### Premissa: User Data é obrigatório

Todo JSON do Tracking Engineer deve incluir **user data nas tags de conversão** quando o handoff declara `user.*` nos eventos. Isso é uma premissa operacional — não opcional.

Checklist de user data por plataforma:
- **GA4 Lead/MQL tags:** incluir `user_email`, `user_phone`, `user_name` como event parameters
- **Meta Init tag:** incluir Advanced Matching (`em`, `ph`, `fn` no `fbq('init', ...)`)
- **Google Ads Conversion tags:** documentar que Enhanced Conversions deve ser ativado manualmente no GTM (não suportado via JSON)

Se o handoff não declarar user data, não inclua. Se declarar, sempre inclua.

### Estrutura obrigatória (exportFormatVersion 2)

```json
{
  "exportFormatVersion": 2,
  "exportTime": "YYYY-MM-DD HH:MM:SS",
  "containerVersion": {
    "path": "accounts/000000/containers/000000/versions/0",
    "accountId": "000000",
    "containerId": "000000",
    "containerVersionId": "0",
    "container": {
      "path": "accounts/000000/containers/000000",
      "accountId": "000000",
      "containerId": "000000",
      "name": "Nome do Container",
      "publicId": "GTM-XXXXXXX",
      "usageContext": ["WEB"],
      "fingerprint": "1700000000000"
    },
    "fingerprint": "1700000000001",
    "tag": [],
    "trigger": [],
    "variable": []
  }
}
```

> Os valores de `accountId`, `containerId` e `path` são placeholders — o GTM ignora e usa os do container de destino na importação. O importante é que estejam presentes como strings numéricas válidas.

### Regras invioláveis por elemento

**Tags:**
```json
{
  "accountId": "000000",
  "containerId": "000000",
  "tagId": "1",
  "name": "TAG | GA4 | Config",
  "type": "gaawc",
  "parameter": [],
  "firingTriggerId": ["ID_DO_TRIGGER"],
  "tagFiringOption": "ONCE_PER_EVENT",
  "fingerprint": "1700000000010",
  "consentSettings": { "consentStatus": "NOT_SET" }
}
```

- `tagId`: string numérica, único por tag
- `firingTriggerId`: array de strings — deve corresponder a um `triggerId` definido ou `"2147479553"` (All Pages built-in)
- `fingerprint`: qualquer timestamp Unix em ms como string — deve ser único por elemento
- `consentSettings`: obrigatório em formato v2

**Triggers:**
```json
{
  "accountId": "000000",
  "containerId": "000000",
  "triggerId": "100",
  "name": "TRG | Custom Event | Lead",
  "type": "CUSTOM_EVENT",
  "customEventFilter": [{
    "type": "EQUALS",
    "parameter": [
      { "type": "TEMPLATE", "key": "arg0", "value": "{{_event}}" },
      { "type": "TEMPLATE", "key": "arg1", "value": "Lead" }
    ]
  }],
  "fingerprint": "1700000000100"
}
```

- `triggerId`: string numérica, único por trigger
- ID `"2147479553"` é reservado para All Pages — NÃO declare no array de triggers

**Variables (Data Layer):**
```json
{
  "accountId": "000000",
  "containerId": "000000",
  "variableId": "1",
  "name": "DLV | event_id",
  "type": "v",
  "parameter": [
    { "type": "INTEGER", "key": "dataLayerVersion", "value": "2" },
    { "type": "TEMPLATE", "key": "name", "value": "event_id" }
  ],
  "fingerprint": "1700000000201"
}
```

- `variableId`: string numérica, único por variável
- `type: "v"` = Data Layer Variable
- `type: "c"` = Constant

### TAG_REFERENCE para GA4 Event tags

GA4 Event tags (`gaawe`) referenciam a tag Config pelo **nome exato**:

```json
{ "type": "TAG_REFERENCE", "key": "measurementId", "value": "TAG | GA4 | Config" }
```

O `value` deve ser o `name` exato da tag `gaawc`. O GTM resolve pelo nome durante import.

### Regras de integridade referencial (CRÍTICO)

O GTM valida referências cruzadas no JSON **antes** de importar. Mesmo em JSONs parciais (updates), todas as dependências devem estar presentes:

1. **Triggers referenciados devem existir no JSON.** Se uma tag usa `"firingTriggerId": ["105"]`, o trigger com `"triggerId": "105"` DEVE estar no array `trigger[]` — mesmo que já exista no container de destino. A única exceção é `"2147479553"` (All Pages built-in).

2. **TAG_REFERENCE deve apontar para tag presente no JSON.** Se uma tag GA4 Event usa `TAG_REFERENCE` para `"TAG | GA4 | Config"`, essa tag Config DEVE estar no JSON — ou já existir no container e o import deve ser feito com "Combinar → Substituir conflitantes".

3. **Variáveis referenciadas em tags HTML** (via `{{DLV | xxx}}`) NÃO precisam estar no JSON — o GTM não valida referências dentro de strings HTML. Mas é boa prática incluí-las.

> **Regra prática:** para JSONs parciais (updates), inclua SEMPRE os triggers e a tag Config referenciados, mesmo que já existam no container. O custo de incluir é zero; o custo de não incluir é erro de importação.

### Erros comuns que causam falha na importação

| Erro | Causa | Fix |
|---|---|---|
| "Não encontrado" | `path` ausente no `containerVersion` ou `container` | Adicionar `path` com formato `accounts/ID/containers/ID/versions/0` |
| "O formato do arquivo é inválido" | Campo obrigatório ausente (`fingerprint`, `consentSettings`, `tagId`) | Verificar que todo elemento tem todos os campos |
| "O valor precisa ser um número inteiro positivo" | `conversionId` do Google Ads com prefixo "AW-" | Usar apenas o número: `"698149535"` sem "AW-" |
| "A tag faz referência a um acionador desconhecido" | `firingTriggerId` aponta para trigger que não está no JSON | Incluir o trigger no array `trigger[]` — mesmo em JSONs parciais |
| "Unknown variable name" / "A variável desconhecida 'Event' foi encontrada em uma tag" | Variável built-in (`{{Event}}`, `{{Page URL}}`, etc.) usada em tag custom mas NÃO habilitada no workspace | Variables → Built-In Variables → Configure → marcar a built-in (ex: Event na categoria Utilitários). Acontece tipicamente após import de container que adiciona tag CAPI Forwarder usando `{{Event}}` |
| "Unknown variable name" (variável custom) | Variável `{{DLV \| xxx}}` referenciada em tag mas não definida em `variable[]` | Adicionar a variável no array |
| TAG_REFERENCE não resolve | `value` não bate com o `name` da tag config | Conferir case-sensitive: nome exato |

### Regra de conversionId Google Ads

No JSON importável, o `conversionId` da tag `awct` (Google Ads Conversion Tracking) e `sp` (Remarketing) deve ser **apenas o número**, sem prefixo:

- Correto: `"698149535"`
- Errado: `"AW-698149535"`

O prefixo "AW-" é usado no painel do Google Ads e em gtag.js, mas no JSON do GTM o campo aceita apenas inteiro positivo.

### Instrução de importação para o operador

Ao entregar o JSON, instrua:
1. GTM → Admin → Import Container
2. Selecione o arquivo JSON
3. Escolha **"Combinar"** (não "Substituir" — preserva config existente)
4. Se houver conflitos, escolha "Renomear conflitantes"
5. Revise as mudanças no workspace antes de publicar
