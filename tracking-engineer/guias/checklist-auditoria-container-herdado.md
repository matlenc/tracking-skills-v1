# Checklist — Auditoria de Container GTM Herdado
> Carregado sob demanda no Modo 3 (REPAIR/COMPLEMENT) ou no Modo 4 (SERVER-SIDE EVOLUTION) antes de adicionar tags novas que dependam de triggers existentes filtrarem corretamente.

---

## Por que existe esse checklist

Containers configurados por agências anteriores costumam ter **dívidas de configuração silenciosas** que não quebram nada hoje (porque tags GA4/Clarity filtram internamente), mas explodem quando se adiciona tag mais sensível como Meta CAPI Forwarder. O sintoma típico: tag nova dispara em `gtm.init`, `gtm.dom`, `gtm.js`, `gtm.load` e polui o destino (CAPI, GA4, etc).

**Princípio:** auditar antes de adicionar. 5 minutos de auditoria poupam horas de debug.

---

## Item 1 — Triggers Custom Event devem ter filtro de nome explícito

### Anti-pattern detectado
Trigger Custom Event configurado como **"Este acionador é disparado em: Todos os eventos personalizados"** sem condição de filtro.

### Por que é problema
GTM trata `gtm.init`, `gtm.init_consent`, `gtm.dom`, `gtm.js`, `gtm.load` como custom events. Trigger sem filtro dispara em TODOS — incluindo esses eventos internos. Em tags GA4/Clarity é tolerável (elas têm filtros internos próprios), mas em tags Custom HTML (CAPI Forwarder, beacons próprios) gera ruído crítico.

### Como detectar
Para cada trigger Custom Event no workspace:
1. Abrir o trigger
2. Verificar campo "Este acionador é disparado em:"
3. Se for **"Todos os eventos personalizados"** → dívida
4. Se for **"Alguns eventos personalizados"** + condição `{{Event}} é igual a <NOME>` → OK

### Correção
Trocar pra "Alguns eventos personalizados" + condição `{{Event}} é igual a <NomeExatoDoEvento>`.

**Atenção a regressão:** outras tags que já usam o trigger podem depender do comportamento "All Custom Events" (ex: tag GA4 com lógica de filtro interna que assume que todos os custom events passam). Antes de corrigir, mapear quais tags usam o trigger e validar caso a caso.

### Migração segura (passo a passo)
1. Listar referências do trigger (GTM mostra na própria tela do trigger em "Referências a este acionador")
2. Para cada tag que usa o trigger, conferir se ela funciona corretamente quando o trigger só dispara no evento esperado
3. Se alguma tag depende de disparar em outro evento, criar um trigger novo dedicado pra essa tag antes de corrigir
4. Aplicar a correção do filtro
5. QA com Preview Mode FECHADO + interceptor (ver `playbook-validacao-e2e-server-side.md`)

---

## Item 2 — Tags Custom HTML sensíveis devem ter guard JS interno

### Anti-pattern detectado
Tag Custom HTML (CAPI Forwarder, webhook próprio) sem `if (evt !== 'X' && evt !== 'Y') return;` no início.

### Por que é problema
Defense in depth: se o trigger for revertido por engano (ou estiver com bug — Item 1), o guard JS é a segunda camada que segura. Custo zero, blindagem completa.

### Como detectar
Abrir cada tag Custom HTML. Procurar guard de allowlist nos primeiros 5-10 linhas:
```js
var evt = {{Event}};
if (evt !== 'Lead' && evt !== 'MQL' && evt !== 'NOICP') return;
```

### Correção
Adicionar guard logo após o `(function(){`. Lista de eventos permitidos = mesma do `firingTriggerId` da tag.

---

## Item 3 — Variáveis Constant `[EDIT] Perma | ...` que carregam endpoint sensível devem ter valor placeholder seguro

### Anti-pattern detectado
Variável `[EDIT] Perma | Meta CAPI Endpoint` (ou similar) com valor vazio ou hardcoded de um ambiente errado.

### Por que é problema
Se o operador esquecer de configurar a URL real após import, e a tag não tiver guard de endpoint, requests vão pra `''` ou pra ambiente errado — gera 404 nos logs ou pior, polui ambiente errado.

### Como detectar
Listar todas as variáveis `[EDIT] Perma |` que carregam URLs/endpoints. Verificar valor.

### Correção
Valor default deve ser placeholder identificável (ex: `__SET_ME__`). Tag que consome deve ter guard:
```js
var endpoint = {{[EDIT] Perma | Meta CAPI Endpoint}};
if (!endpoint || endpoint === '__SET_ME__') return;
```

---

## Item 4 — Tags Meta de conversão devem ter `eventID` para deduplicação CAPI

### Anti-pattern detectado
`fbq('track', 'Lead')` ou `fbq('trackCustom', 'MQL', { lead_status: ... })` SEM o terceiro parâmetro `eventID`.

### Por que é problema
Sem `eventID`, Meta não consegue deduplicar Pixel browser ↔ CAPI server-side. Cada lead vira 2 conversões. Smart Bidding otimiza por volume inflado.

### Como detectar
Para cada tag Meta de evento de conversão: procurar `eventID:` no payload do `fbq(...)`.

### Correção
Adicionar terceiro arg com `eventID: '{{DLV | event_id}}'` (ou similar). O `event_id` no DLV deve ser o MESMO enviado pelo CAPI server-side — esse é o link da deduplicação.

---

## Item 5 — Variáveis Built-in necessárias devem estar habilitadas

### Anti-pattern detectado
Tag custom usa `{{Event}}`, `{{Page URL}}`, `{{Page Path}}` mas a built-in correspondente não está habilitada em "Variables → Built-In Variables → Configure".

### Por que é problema
Bloqueia publicação. Erro: "A variável desconhecida 'Event' foi encontrada em uma tag." Discoverable só no momento de Submit + Publish.

### Como detectar
Antes de importar JSON com tag custom que referencia `{{Event}}` ou outras built-ins, verificar Variables → Built-In Variables se estão habilitadas. Se não, habilitar.

### Built-ins comumente necessárias
- `Event` (Utilitários) — pra `{{Event}}` em tags custom multi-evento
- `Page URL`, `Page Path`, `Page Hostname`, `Referrer` (Pages) — derivadas comuns

---

## Item 6 — Plugins legacy de WordPress podem injetar tracking fantasma

### Anti-pattern detectado
Container GTM novo configurado, mas eventos antigos (Pixels velhos, GA Universal, GTM antigo) continuam disparando junto. Causa típica: plugin "Insert Headers and Footers" (IHAF) foi desinstalado mas opções `wp_options.ihaf_insert_header/body/footer` permaneceram, e plugins atuais (WPCode Lite) preservam backwards-compat lendo essas opções.

### Por que é problema
Eventos duplicados, atribuição errada, EMQ baixo no Meta porque há 2 pixels disputando o mesmo evento.

### Como detectar
1. View source do site em produção (`Ctrl+U` no browser)
2. Procurar por: `GTM-` (deve haver APENAS o container atual), `fbq('init'` (apenas o pixel correto), `googletagmanager.com/gtag/js` (apenas o GA4 atual)
3. Se houver containers/pixels desconhecidos, investigar fonte: tema, plugins ativos, **`wp_options` legacy** (especialmente `ihaf_*`)

### Correção
Snippet PHP one-shot via Code Snippets plugin:
```php
delete_option('ihaf_insert_header');
delete_option('ihaf_insert_body');
delete_option('ihaf_insert_footer');
```

Adaptar pra outros plugins legacy descobertos.

---

## Output da auditoria

Para cada item auditado, registre no diagnóstico do Modo 3:

```markdown
## Auditoria de Container GTM — [Cliente]
Data: YYYY-MM-DD
Container: GTM-XXXXXXX

### Item 1 — Triggers Custom Event filtrados corretamente?
- [ ] OK / [ ] Dívida → [lista de triggers com problema]

### Item 2 — Tags Custom HTML com guard JS interno?
- [ ] OK / [ ] Dívida → [lista de tags sem guard]

### Item 3 — Variáveis [EDIT] Perma com valores seguros?
- [ ] OK / [ ] Dívida → [lista de variáveis com valor inadequado]

### Item 4 — Tags Meta de conversão com eventID?
- [ ] OK / [ ] Dívida → [lista de tags sem eventID]

### Item 5 — Built-in Variables necessárias habilitadas?
- [ ] OK / [ ] Habilitar antes de import: [lista]

### Item 6 — Tracking fantasma de plugins legacy?
- [ ] OK / [ ] Encontrado: [GTM-XXX/Pixel-XXX em <fonte>]

## Plano de correção priorizado
[ordem de risco vs. esforço — do mais crítico ao menos]
```

Confirme o plano com o operador antes de executar correções.
