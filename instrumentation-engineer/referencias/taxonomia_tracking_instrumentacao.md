# Taxonomia de Tracking e Instrumentação
Versão: 0.2  
Status: Final  
Owner: Robson / Dante

---

## 1. Objetivo

Este documento define o padrão de taxonomia para tracking, instrumentação de formulários, eventos de navegação, qualificação de leads e payloads enviados para GTM, plataformas de mídia e fluxos via n8n.

O objetivo é padronizar a **saída do sistema**, mesmo quando a **entrada varia** entre projetos, páginas, formulários, construtores e CRMs.

Este documento não padroniza:
- CSS
- layout do formulário
- número de campos
- perguntas de qualificação
- opções de resposta
- regra de qualificação
- webhook específico do projeto

Este documento padroniza:
- nomes canônicos de eventos
- contrato do `dataLayer`
- contrato do payload enviado ao n8n
- convenção de nomenclatura
- níveis de obrigatoriedade dos campos
- regras de contexto
- critérios de consistência entre front, GTM e server-side

---

## 2. Princípios do Sistema

### 2.1. Evento canônico, contexto em parâmetros
O nome do evento deve representar **o que aconteceu**, não todos os contextos possíveis.

**Certo**
- `Lead`
- `MQL`
- `NOICP`

**Errado**
- `Lead_Bootcamp`
- `MQL_BUX_JornadaX`
- `form_submit_bootcamp`

O contexto deve ir em parâmetros.

---

### 2.2. Entrada flexível, saída rígida
Cada projeto pode ter:
- campos diferentes
- formulários diferentes
- critérios de qualificação diferentes
- webhooks diferentes
- páginas e jornadas diferentes

Mas a saída deve respeitar o mesmo contrato.

---

### 2.3. Machine-readable
Os campos devem ser consistentes e fáceis de mapear no GTM, n8n, GA4, Meta, Google Ads e análises futuras.

---

### 2.4. Separação de responsabilidades
- **Página / Form / Script** = origem do dado
- **dataLayer** = contrato de transporte no front
- **GTM** = leitura, mapeamento e envio
- **n8n** = orquestração server-side
- **CRM** = verdade comercial do avanço no pipeline

---

### 2.5. Taxonomia de mídia compatível
Sempre que possível, os campos de campanha devem preservar a lógica de taxonomia já adotada para campanhas, conjuntos e anúncios, que usa estrutura machine-readable e decomposição por separadores.

---

## 3. Convenções de Nomenclatura

### 3.1. Eventos
Os nomes dos eventos canônicos devem usar **PascalCase**.

Exemplos:
- `PageView`
- `CTAInteract`
- `VideoProgress`
- `FormStart`
- `Lead`
- `MQL`
- `NOICP`
- `LeadQualified`
- `LeadSQL`
- `Opportunity`
- `DealWon`

---

### 3.2. Parâmetros
Os parâmetros devem usar **snake_case**.

Exemplos:
- `event_id`
- `event_time`
- `page_type`
- `form_id`
- `lead_status`
- `qualification_rule`

---

### 3.3. Convenções semânticas
- `project` = cliente/projeto
- `product` = oferta/produto
- `journey` = jornada
- `funnel` = estágio macro do funil
- `business_unit` = unidade de negócio
- `segment` = segmento relevante para leitura do contexto

---

### 3.4. Valores enumerados
Sempre que possível, valores recorrentes devem seguir listas fechadas.

Exemplos:
- `page_type`: `landing_page`, `site`, `blog`, `checkout`, `thank_you`
- `asset_type`: `form_custom`, `form_embed`, `form_native`, `page_builder`, `site`
- `lead_status`: `lead`, `mql`, `noicp`, `sql`, `opportunity`, `won`

---

## 4. Eventos Canônicos

### 4.1. Eventos de navegação e interação
- `PageView`
- `CTAInteract`
- `VideoProgress`
- `FormStart`

---

### 4.2. Eventos de captura e qualificação no front
- `Lead`
- `MQL`
- `NOICP`

---

### 4.3. Eventos de avanço comercial via CRM / n8n / server-side
- `LeadQualified`
- `LeadSQL`
- `Opportunity`
- `DealWon`

---

## 5. Regras Gerais dos Eventos

### 5.1. Um evento canônico não deve ser renomeado por projeto
Exemplo:
- `MQL` continua sendo `MQL`, independentemente de produto, BU, jornada ou campanha.

O contexto entra em parâmetros como:
- `product`
- `journey`
- `business_unit`
- `segment`
- `project`

---

### 5.2. Um mesmo evento pode ter diferentes origens
Exemplo:
- `Lead` pode nascer de form custom HTML
- `Lead` pode nascer de script em construtor
- `Lead` pode nascer de integração indireta

Mas o nome e o contrato permanecem os mesmos.

---

### 5.3. Qualificação não muda o nome-base do evento
A lógica de qualificação pode variar por projeto, mas deve ser informada em parâmetros.

Exemplo:
- `qualification_rule`
- `qualification_version`
- `qualification_reason`

---

## 6. Níveis de Obrigatoriedade dos Parâmetros

### 6.1. Obrigatórios
Devem existir em qualquer evento do sistema.

- `event`
- `event_id`
- `event_time`

---

### 6.2. Obrigatórios por contexto
Devem existir quando o tipo de evento exigir.

Exemplos:
- `form_id` em eventos de formulário
- `lead_status` em eventos de qualificação
- `value` em `DealWon`
- `qualification_rule` em `MQL` e `NOICP`

---

### 6.3. Opcionais
São desejáveis, mas não bloqueiam o evento.

Exemplos:
- `business_unit`
- `segment`
- `utm_term`
- `utm_content`
- `ad_id`
- `campaign_name`

---

## 7. Contrato Base do dataLayer

### 7.1. Estrutura mínima obrigatória
```js
window.dataLayer.push({
  event: "Lead",
  event_id: "uuid-ou-id-unico",
  event_time: 1712600000
});
```

---

### 7.2. Estrutura recomendada
```js
window.dataLayer.push({
  event: "Lead",
  event_id: "uuid-ou-id-unico",
  event_time: 1712600000,

  page_type: "landing_page",
  asset_type: "form_custom",
  form_id: "lead_form_main",
  form_name: "Lead Form Main",

  project: "ClienteXPTO",
  product: "BootcampVendas",
  journey: "Webinar",
  funnel: "mid",
  business_unit: "bux",
  segment: "educacao",

  lead_status: "lead",

  attribution: {
    utm_source: "meta",
    utm_medium: "paid_social",
    utm_campaign: "TEST_META_LEAD_BootcampVendas_FormNativo",
    utm_term: "",
    utm_content: "ad-015",
    utm_id: "123456",
    gclid: "",
    fbclid: "xxx",
    ad_id: "987654"
  },

  user: {
    email: "lead@email.com",
    phone: "5511999999999",
    first_name: "joao",
    last_name: "silva",
    full_name: "joao silva"
  }
});
```

---

## 8. Contrato de Parâmetros

### 8.1. Identidade do evento

#### Obrigatórios
- `event`
- `event_id`
- `event_time`

#### Regras
- `event` deve usar nome canônico em PascalCase
- `event_id` deve ser único por ocorrência
- `event_time` deve ser timestamp unix em segundos

---

### 8.2. Contexto da página / ativo

#### Condicionais
- `page_type`
- `page_name`
- `page_path`
- `asset_type`
- `asset_name`

#### Valores recomendados
- `page_type`: `landing_page`, `site`, `blog`, `checkout`, `thank_you`
- `asset_type`: `form_custom`, `form_embed`, `form_native`, `page_builder`, `site`

---

### 8.3. Contexto do projeto

#### Opcionais recomendados
- `project`
- `product`
- `journey`
- `funnel`
- `business_unit`
- `segment`

#### Regra
- `project` representa o cliente/projeto
- `product` representa a oferta/produto

Esses campos não alteram o nome do evento. Eles contextualizam o evento.

---

### 8.4. Contexto do formulário

#### Obrigatórios para eventos de formulário
- `form_id`

#### Condicionais
- `form_name`
- `form_type`
- `form_step`
- `form_variant`

#### Valores recomendados
- `form_type`: `custom_html`, `native`, `embed`, `builder`

---

### 8.5. Contexto de interação

#### Condicionais
- `cta_text`
- `cta_position`
- `cta_type`
- `video_name`
- `video_percent`

---

### 8.6. Contexto de qualificação

#### Obrigatórios para `MQL` e `NOICP`
- `lead_status`
- `qualification_rule`

#### Condicionais
- `qualification_version`
- `qualification_reason`
- `is_icp`

#### Valores recomendados para `lead_status`
- `lead`
- `mql`
- `noicp`
- `sql`
- `opportunity`
- `won`

---

### 8.7. Contexto do usuário

#### Opcionais / condicionais
- `user.email`
- `user.phone`
- `user.first_name`
- `user.last_name`
- `user.full_name`

#### Regra
Esses campos só devem ser enviados quando existirem no formulário ou no sistema de origem.

Não é obrigatório que todo formulário tenha nome, telefone e email.

---

### 8.8. Contexto de atribuição

#### Recomendados
- `utm_source`
- `utm_medium`
- `utm_campaign`
- `utm_term`
- `utm_content`
- `utm_id`
- `gclid`
- `fbclid`
- `ad_id`
- `campaign_id`
- `campaign_name`
- `landing_page_url`
- `original_location`
- `referrer`

#### Regra
Sempre que possível, os parâmetros de mídia devem preservar a taxonomia padrão da operação.

---

## 9. Regras por Evento

### 9.1. PageView
Evento de visualização de página.

#### Obrigatórios
- `event`
- `event_id`
- `event_time`

#### Recomendados
- `page_type`
- `page_name`
- `page_path`
- `asset_type`
- `project`
- `product`
- `journey`
- `attribution.*`

---

### 9.2. CTAInteract
Evento de clique ou interação com CTA relevante.

#### Obrigatórios
- `event`
- `event_id`
- `event_time`

#### Recomendados
- `cta_text`
- `cta_position`
- `cta_type`
- `page_type`
- `project`

---

### 9.3. VideoProgress
Evento de progresso em vídeo.

#### Obrigatórios
- `event`
- `event_id`
- `event_time`
- `video_percent`

#### Recomendados
- `video_name`
- `page_type`
- `project`

---

### 9.4. FormStart
Primeira interação relevante com o formulário.

#### Obrigatórios
- `event`
- `event_id`
- `event_time`
- `form_id`

#### Recomendados
- `form_name`
- `form_type`
- `page_type`
- `project`

---

### 9.5. Lead
Evento de envio válido do formulário.

#### Obrigatórios
- `event`
- `event_id`
- `event_time`
- `form_id`
- `lead_status`

#### Valor recomendado
- `lead_status = "lead"`

#### Recomendados
- `user.*`
- `project`
- `product`
- `journey`
- `attribution.*`

---

### 9.6. MQL
Evento de lead qualificado no front.

#### Obrigatórios
- `event`
- `event_id`
- `event_time`
- `form_id`
- `lead_status`
- `qualification_rule`

#### Valor recomendado
- `lead_status = "mql"`

#### Condicionais
- `qualification_reason`
- `qualification_version`
- `user.*`

---

### 9.7. NOICP
Evento de lead não qualificado.

#### Obrigatórios
- `event`
- `event_id`
- `event_time`
- `form_id`
- `lead_status`
- `qualification_rule`

#### Valor recomendado
- `lead_status = "noicp"`

#### Condicionais
- `qualification_reason`
- `qualification_version`
- `user.*`

---

### 9.8. LeadQualified
Evento vindo de CRM / n8n indicando lead qualificado em etapa posterior.

#### Obrigatórios
- `event`
- `event_id`
- `event_time`
- `lead_status`

#### Valor recomendado
- `lead_status = "mql"`

---

### 9.9. LeadSQL
Evento vindo de CRM / n8n indicando SQL.

#### Obrigatórios
- `event`
- `event_id`
- `event_time`
- `lead_status`

#### Valor recomendado
- `lead_status = "sql"`

---

### 9.10. Opportunity
Evento vindo de CRM / n8n indicando oportunidade aberta.

#### Obrigatórios
- `event`
- `event_id`
- `event_time`
- `lead_status`

#### Valor recomendado
- `lead_status = "opportunity"`

---

### 9.11. DealWon
Evento vindo de CRM / n8n indicando venda concluída.

#### Obrigatórios
- `event`
- `event_id`
- `event_time`
- `lead_status`
- `value`
- `currency`

#### Valores recomendados
- `lead_status = "won"`
- `currency = "BRL"`

---

## 10. Contrato de Payload para n8n

O payload enviado ao n8n deve seguir a mesma lógica semântica do dataLayer, ainda que o formato técnico varie.

### 10.1. Estrutura recomendada
```json
{
  "event": "MQL",
  "event_id": "uuid-ou-id-unico",
  "event_time": 1712600000,

  "page_type": "landing_page",
  "asset_type": "form_custom",
  "form_id": "lead_form_main",
  "form_name": "Lead Form Main",

  "project": "ClienteXPTO",
  "product": "BootcampVendas",
  "journey": "Webinar",
  "funnel": "mid",
  "business_unit": "bux",
  "segment": "educacao",

  "lead_status": "mql",
  "qualification_rule": "BootcampVendas_v1",
  "qualification_version": "1",
  "qualification_reason": "faturamento_1m_plus",

  "user": {
    "email": "lead@email.com",
    "phone": "5511999999999",
    "first_name": "joao",
    "last_name": "silva",
    "full_name": "joao silva"
  },

  "attribution": {
    "utm_source": "meta",
    "utm_medium": "paid_social",
    "utm_campaign": "TEST_META_LEAD_BootcampVendas_FormNativo",
    "utm_term": "",
    "utm_content": "ad-015",
    "utm_id": "123456",
    "gclid": "",
    "fbclid": "xxx",
    "ad_id": "987654"
  }
}
```

---

## 11. Regras para n8n como intermediador

### 11.1. O n8n é o orquestrador server-side
Quando houver eventos vindos de CRM ou necessidade de integração server-side, o n8n atua como intermediador entre:
- front
- CRM
- GTM Server-Side
- APIs externas

---

### 11.2. O n8n não deve quebrar a semântica do evento
Se o evento no contrato é `MQL`, o n8n não deve renomeá-lo para outro padrão arbitrário.

---

### 11.3. O n8n pode enriquecer o payload
Exemplos:
- adicionar `crm_stage`
- adicionar `owner_name`
- adicionar `deal_id`
- adicionar `pipeline_name`

Desde que preserve o núcleo canônico.

---

## 12. Regras para GTM

### 12.1. O GTM não é a origem da lógica de negócio
A lógica de qualificação não deve nascer no GTM.

O GTM deve:
- ler os dados
- mapear variáveis
- disparar tags
- enviar para plataformas

---

### 12.2. O GTM deve usar eventos canônicos
O GTM deve escutar:
- `FormStart`
- `Lead`
- `MQL`
- `NOICP`
- etc.

E não múltiplas variações por projeto.

---

### 12.3. O GTM deve mapear contexto via variáveis
Exemplos:
- `DLV | form_id`
- `DLV | lead_status`
- `DLV | qualification_rule`
- `DLV | user.email`

---

## 13. Regras para Formulários Instrumentados

### 13.1. O formulário não precisa ter sempre os mesmos campos
O sistema deve funcionar mesmo quando:
- não houver nome
- não houver telefone
- não houver email
- houver menos perguntas
- houver mais perguntas
- houver lógica de qualificação diferente

---

### 13.2. O formulário deve produzir payload consistente
Mesmo com campos variáveis, a saída deve respeitar:
- nome canônico do evento
- `event_id`
- `event_time`
- parâmetros obrigatórios do contexto

---

### 13.3. Campos ocultos não são a origem da verdade
Campos ocultos podem ser usados para transporte técnico, mas a semântica do evento deve ser definida pelo contrato lógico do payload.

---

## 14. Anti-patterns

- Criar nomes de eventos diferentes por projeto
- Colocar BU, produto ou jornada no nome do evento
- Fazer o GTM inventar a lógica de qualificação
- Misturar nomenclatura de eventos entre `snake_case`, `camelCase` e `PascalCase`
- Fazer o n8n renomear eventos arbitrariamente
- Tornar `email`, `telefone` e `nome` obrigatórios em todos os formulários
- Assumir que todo formulário terá a mesma estrutura
- Acoplar a taxonomia ao CSS ou layout
- Usar hidden fields como única fonte de verdade sem objeto lógico padronizado
- Duplicar sem necessidade o mesmo contexto no nome do evento e nos parâmetros

---

## 15. Exemplos Práticos

### 15.1. FormStart
```js
window.dataLayer.push({
  event: "FormStart",
  event_id: "fs_001",
  event_time: 1712600000,
  form_id: "lead_form_main",
  form_name: "Lead Form Main",
  form_type: "custom_html",
  page_type: "landing_page",
  project: "ClienteXPTO"
});
```

---

### 15.2. Lead
```js
window.dataLayer.push({
  event: "Lead",
  event_id: "lead_001",
  event_time: 1712600100,
  form_id: "lead_form_main",
  lead_status: "lead",
  project: "ClienteXPTO",
  product: "BootcampVendas",
  user: {
    email: "lead@email.com",
    phone: "5511999999999"
  }
});
```

---

### 15.3. MQL
```js
window.dataLayer.push({
  event: "MQL",
  event_id: "mql_001",
  event_time: 1712600110,
  form_id: "lead_form_main",
  lead_status: "mql",
  qualification_rule: "BootcampVendas_v1",
  qualification_reason: "faturamento_1m_plus",
  project: "ClienteXPTO",
  product: "BootcampVendas",
  user: {
    email: "lead@email.com",
    phone: "5511999999999"
  }
});
```

---

### 15.4. NOICP
```js
window.dataLayer.push({
  event: "NOICP",
  event_id: "noicp_001",
  event_time: 1712600110,
  form_id: "lead_form_main",
  lead_status: "noicp",
  qualification_rule: "BootcampVendas_v1",
  qualification_reason: "faturamento_abaixo_do_corte",
  project: "ClienteXPTO"
});
```

---

### 15.5. LeadSQL via n8n
```json
{
  "event": "LeadSQL",
  "event_id": "sql_001",
  "event_time": 1712605000,
  "lead_status": "sql",
  "project": "ClienteXPTO",
  "crm_stage": "SQL Confirmado",
  "deal_id": "12345"
}
```

---

## 16. Próximos Passos

1. Usar este documento como base do SOP
2. Usar este documento como referência para as skills:
   - `Instrumentation Engineer`
   - `Tracking Engineer`
3. Fechar o SOP operacional com base nesta taxonomia
4. Revisar o briefing final das duas skills

---
