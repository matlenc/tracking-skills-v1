# Playbook — Validação E2E de Tags Server-Side
> Carregado sob demanda na Fase 6 (QA) quando há tag CAPI Forwarder, beacon próprio ou qualquer tag custom que envia ao N8N/SGTM/endpoint próprio.

---

## Por que existe esse playbook

GTM Tag Assistant Preview Mode confirma que a tag DISPAROU mas NÃO confirma que o destino RECEBEU. Pior: Preview Mode polui o destino com eventos `gtm.*` que NÃO existem em produção real. Validar tag server-side só pelo Tag Assistant gera 2 falsos positivos:

1. **Falso negativo:** tag dispara no Preview, sendBeacon "OK" no console, mas destino nunca recebe (CORS preflight, payload mal formado, endpoint errado, etc).
2. **Falso positivo:** destino recebe eventos `gtm.init/dom/load` que poluem analytics — operador acha que tag tá disparando errado em produção, quando na verdade é só Preview Mode aberto.

Esse playbook elimina os dois.

---

## Regra zero — fechar o Tag Assistant Preview ANTES de validar

GTM Tag Assistant carrega o site dentro de iframe `https://gtm-msr.appspot.com/render?id=<container>` e dispara eventos internos do GTM nesse contexto. Tags ativas no workspace de Preview disparam ali e mandam dados pra produção (webhook N8N, etc) — que NÃO correspondem ao comportamento real.

**Heurística de debug:** se o destino recebe execuções com `event_source_url = https://gtm-msr.appspot.com/render?...`, é Preview rodando — não é bug da tag.

Antes de cada validação:
1. Fecha aba do GTM ou clica em "Leave preview mode"
2. Confirma que `chrome://extensions` não tem GTM Preview/Tag Assistant Helper ativo na página de produção

---

## Setup do interceptor

Cole no DevTools Console da página de produção (depois de fechar Preview):

```js
// Intercepta sendBeacon — captura url + body sem bloquear o request
const __origBeacon = navigator.sendBeacon.bind(navigator);
navigator.sendBeacon = function(url, data) {
  let preview = data;
  if (typeof data === 'string') {
    try { preview = JSON.parse(data); } catch(e) {}
  } else if (data instanceof Blob) {
    data.text().then(t => console.log('🔵 sendBeacon →', url, '\n', t.substring(0, 500)));
    return __origBeacon(url, data);
  }
  console.log('🔵 sendBeacon →', url, '\n', preview);
  return __origBeacon(url, data);
};

// Intercepta fetch para endpoints conhecidos (ajustar lista por projeto)
const __origFetch = window.fetch;
window.fetch = function(url, opts) {
  if (typeof url === 'string' && /n8n|graph\.facebook|googleads|clarity\.ms|google-analytics|sgtm/.test(url)) {
    let body = opts && opts.body;
    if (typeof body === 'string') {
      try { body = JSON.parse(body); } catch(e) {}
    }
    console.log('🟢 fetch →', url, '\n', body || '(sem body)');
  }
  return __origFetch.apply(this, arguments);
};

console.log('✅ Interceptor instalado. Dispara o evento agora.');
```

> **Pegadinha sendBeacon com Blob:** se o body for `Blob` com type `application/json`, navegador tenta CORS preflight que sendBeacon não suporta — request é dropado silenciosamente (sem erro JS). Se vir Blob no log, **a tag tá com bug** (deve passar string, não Blob). Documentado em `gtm-config-reference.md` §2.2.5.

---

## Roteiro de validação E2E

### 1. Limpar estado
- Fechar Preview Mode (regra zero)
- Browser limpo: incognito ou Playwright (sem cookies de sessão anterior)

### 2. Instalar interceptor
- Cola o snippet do bloco acima no DevTools Console

### 3. Disparar o evento real
- Form submit, checkout, click no CTA — qualquer ação que aciona o evento canônico (Lead/MQL/NOICP/etc)
- NÃO usar `dataLayer.push({event: 'Lead'})` direto — isso pula a lógica do snippet (qualificação, atribuição, etc) e vira teste sintético

### 4. Validar no Console (front-end)
Conferir nos logs:
- ✅ Quantidade de chamadas é a esperada (ex: PJ no popup → 2 sendBeacons: Lead + MQL)
- ✅ URL bate com o endpoint configurado
- ✅ `body_type` é string (não Blob)
- ✅ `event_id` único por chamada
- ✅ `event_name` é canônico (não `gtm.init/dom/load`)
- ✅ `user_data` populado conforme handoff

### 5. Cruzar com Executions do destino (server-side)
Anote os `event_id`s do passo 4 e cheque no destino:
- **N8N:** workflow → Executions → procurar pelos event_ids no body
- **SGTM:** preview do server container
- **Endpoint próprio:** logs do servidor

Para cada event_id capturado no front, deve haver uma execução correspondente no destino. Se NÃO há:
- Possível CORS preflight bloqueando (sendBeacon com Blob — ver pegadinha acima)
- Possível Settings do workflow N8N com "Save successful production executions = None" (executa mas não grava)
- Possível CDN/proxy entre browser e destino dropando o request

### 6. Validar pipeline completo no destino
Para cada execução no N8N/SGTM:
- Pipeline 100% verde (todos os nodes Succeeded)
- Output do último node (Log Response, etc) confirma que a plataforma final aceitou: `meta_status: 200`, `meta_events_received: 1`, `success: true`

### 7. Validar na plataforma final (não só no destino intermediário)
N8N/SGTM aceitar o request NÃO significa que Meta/GA4/Google Ads contabilizou. Confirmar:
- **Meta:** Events Manager → Test Events (com `test_event_code` setado durante QA) ou Visão Geral 10-20min depois → eventos com source "Server" + flag "Deduplicated"
- **GA4:** DebugView com `debug_mode=1`
- **Google Ads:** Tools → Conversions → diagnóstico da ação

### 8. Remover Test Event Code
Após confirmar funcionando, REMOVER `test_event_code` do workflow/tag. Eventos com test code NÃO contam pra otimização de campanhas.

---

## Checklist resumido

- [ ] Tag Assistant Preview FECHADO
- [ ] Browser limpo (incognito/Playwright)
- [ ] Interceptor instalado no DevTools Console
- [ ] Evento real disparado (não `dataLayer.push` sintético)
- [ ] Console mostra N chamadas esperadas, com event_id único
- [ ] `body_type` é string (não Blob — bug se for Blob)
- [ ] `event_name` é canônico (não `gtm.*` — bug de trigger se for `gtm.*`)
- [ ] N execuções correspondentes no N8N/SGTM com mesmos event_ids
- [ ] Pipeline 100% verde nos nodes do workflow
- [ ] Log Response com `success: true`
- [ ] Evento aparece na plataforma final (Meta Test Events, GA4 DebugView, etc)
- [ ] Test Event Code removido após validação

---

## Sintomas comuns + diagnóstico

| Sintoma | Diagnóstico provável |
|---|---|
| Console mostra `sendBeacon` mas N8N nada recebe | Body é Blob `application/json` → CORS preflight → request dropado. Trocar pra string. |
| Destino recebe `gtm.init/gtm.dom/gtm.js/gtm.load` | Tag Assistant Preview aberto OU trigger Custom Event configurado como "All Custom Events" (ver `checklist-auditoria-container-herdado.md` Item 1) |
| Workflow N8N executa mas nada aparece em Executions | Settings → "Save successful production executions" está como `None`. Trocar pra `All`. |
| Pipeline verde mas Meta retorna `events_received: 0` | Token CAPI errado/expirado, ou pixel_id errado, ou payload com campo obrigatório vazio (event_name, event_time, action_source) |
| `event_id` no Pixel browser ≠ `event_id` no CAPI | Variável `{{DLV \| event_id}}` referencia path diferente, ou snippet do front gera 2 IDs (um pra Pixel, outro pro CAPI). Conferir que ambos lêem do MESMO `event_id` no dataLayer. |
| Tag dispara em página de produção mas evento `evt = undefined` no payload | Variável built-in `{{Event}}` não habilitada. Variables → Built-In Variables → Configure → marcar Event |
