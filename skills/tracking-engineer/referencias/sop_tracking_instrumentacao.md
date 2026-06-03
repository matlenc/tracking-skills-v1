# SOP Operacional — Tracking e Instrumentação
Versão: 0.2  
Status: Revisado  
Owner: Robson / Dante  
Base normativa: `taxonomia_tracking_instrumentacao_v0.2_final.md`

---

## 1. Objetivo

Este SOP descreve o processo operacional padrão para planejar, preparar, instrumentar, configurar, validar e documentar projetos de tracking e instrumentação.

Ele existe para garantir que:
- o evento nasce com semântica correta
- o GTM recebe dados consistentes
- as plataformas de mídia recebem sinais confiáveis
- o n8n e o CRM preservam a lógica do funil
- o projeto termina com documentação suficiente para manutenção e evolução

Este SOP deve ser usado como base operacional para as skills:
- `Instrumentation Engineer`
- `Tracking Engineer`

---

## 2. Escopo

Este processo cobre:
- planejamento de medição
- definição de eventos e contexto
- definição de regras de qualificação
- preparação da origem dos dados
- configuração de GTM Web
- configuração de integrações com plataformas
- preparação de fluxo server-side com n8n quando necessário
- QA técnico e funcional
- handoff documental

Este processo não pressupõe:
- um único tipo de formulário
- um único tipo de página
- um único CRM
- uma única regra de qualificação
- um único webhook
- um único stack além do core

---

## 3. Stack Padrão

### 3.1. Core
- GTM Web
- GA4
- Meta
- Google Ads
- Clarity

### 3.2. Sob demanda
- TikTok Ads
- LinkedIn
- CRM específico
- n8n
- GTM Server-Side
- integrações adicionais

---

## 4. Papéis Lógicos do Processo

### 4.1. Instrumentation Engineer
Responsável por preparar a origem do dado.

Entrega:
- contrato de instrumentação
- form HTML ou script de instrumentação
- payload para n8n
- configuração compatível com GTM
- documentação do que foi implementado nas páginas/ativos

### 4.2. Tracking Engineer
Responsável por preparar o consumo e distribuição do dado.

Entrega:
- GTM parametrizado
- variáveis, gatilhos e tags
- integrações com plataformas
- QA
- documentação de completude

---

## 5. Fronteira entre as Skills

### 5.1. Onde termina o Instrumentation Engineer
O `Instrumentation Engineer` termina quando:
- a(s) página(s) ou ativo(s) foram configurados
- a origem do evento foi preparada
- o form, script, builder ou payload necessário foi configurado
- a lógica de qualificação aplicável foi implementada na origem correta
- o webhook e/ou payload para n8n foi preparado quando aplicável
- a documentação do que foi feito foi entregue

A entrega do `Instrumentation Engineer` funciona como handoff para o `Tracking Engineer`.

---

### 5.2. Onde começa o Tracking Engineer
O `Tracking Engineer` começa recebendo o handoff da instrumentação e então:
- entende o que já foi implementado na origem
- verifica o contrato do dado
- complementa o que for necessário no GTM
- conecta plataformas
- ajusta variáveis, gatilhos e tags
- executa QA
- fecha a completude do tracking

---

## 6. Resultado Esperado por Projeto

Ao final do processo, o projeto deve ter:
- plano de medição definido
- taxonomia aplicada
- eventos canônicos mapeados
- variáveis e IDs documentados
- origem de dados preparada
- GTM configurado
- integrações principais ativas
- QA executado
- documentação final entregue

---

## 7. Macrofluxo Operacional

1. Planejamento
2. Provisionamento de stack
3. Registro técnico
4. Preparação da origem do dado
5. Parametrização do GTM
6. Configuração de tags e integrações
7. QA
8. Publicação
9. Handoff

---

## 8. Etapa 1 — Planejamento

### 8.1. Objetivo
Definir o que será medido, onde será medido, como será qualificado e quais sinais precisam chegar às plataformas e ao CRM.

### 8.2. Perguntas obrigatórias
- Quais páginas ou ativos serão rastreados?
- Quais eventos precisam existir?
- Quais eventos são padrão e quais têm contexto específico?
- Haverá formulário?
- O formulário será custom HTML, builder, embed ou nativo?
- Haverá qualificação no front?
- Haverá integração com CRM?
- Haverá roteamento de jornada por status do lead?
- O projeto terá apenas tracking web ou full funnel?
- Quais plataformas fazem parte do escopo?

### 8.3. Saída obrigatória
- lista de páginas/ativos
- lista de eventos
- definição de contexto por evento
- regra de qualificação por projeto
- definição de stack obrigatório e opcional

---

## 9. Etapa 2 — Definição do Measurement Plan

### 9.1. Objetivo
Transformar o planejamento em um plano formal de medição.

### 9.2. O measurement plan deve conter
- evento canônico
- descrição do evento
- origem do evento
- condição de disparo
- parâmetros obrigatórios
- parâmetros condicionais
- destino do evento
- observações operacionais

### 9.3. Eventos-base padrão
- `PageView`
- `CTAInteract`
- `VideoProgress`
- `FormStart`
- `Lead`
- `MQL`
- `NOICP`

### 9.4. Eventos full funnel / CRM
- `LeadQualified`
- `LeadSQL`
- `Opportunity`
- `DealWon`

---

## 10. Etapa 3 — Definição da Regra de Qualificação

### 10.1. Objetivo
Definir como o projeto decide se um lead é apenas `Lead`, `MQL` ou `NOICP`.

### 10.2. O que precisa ser definido
- perguntas de qualificação
- opções de resposta
- critérios de corte
- regra de decisão
- versão da regra
- razão semântica da classificação

### 10.3. Saída obrigatória
- `qualification_rule`
- `qualification_version`
- critérios objetivos
- possíveis `qualification_reason`

### 10.4. Regra crítica
A lógica de qualificação **não deve nascer no GTM**.

Ela deve nascer:
- no form/script
- no construtor
- no n8n
- no CRM

Dependendo do cenário operacional.

---

## 11. Etapa 4 — Provisionamento de Stack

### 11.1. Objetivo
Criar ou confirmar todas as contas e propriedades necessárias.

### 11.2. Itens que normalmente entram
- GA4
- Meta Ads / Pixel
- Google Ads
- Clarity
- GTM Web
- GTM Server-Side, quando necessário
- CRM
- n8n
- integrações opcionais

### 11.3. Saída obrigatória
IDs, chaves e endpoints disponíveis para uso.

---

## 12. Etapa 5 — Registro Técnico do Projeto

### 12.1. Objetivo
Centralizar os dados técnicos necessários para configuração e manutenção.

### 12.2. O inventário técnico deve conter
- `project`
- `product`
- domínios rastreados
- URLs/páginas monitoradas
- GA4 ID
- Google Ads Conversion ID
- conversion labels
- Meta Pixel ID
- Clarity ID
- TikTok Pixel ID, se houver
- LinkedIn Partner ID, se houver
- webhook URL
- transport URL
- CRM
- funnel coberto
- eventos ativos
- regra de qualificação
- redirects de jornada
- observações

### 12.3. Formato aceito
- markdown
- json
- estrutura equivalente

Planilha não é obrigatória se a documentação cumprir a mesma função.

---

## 13. Etapa 6 — Preparação da Origem do Dado

### 13.1. Objetivo
Garantir que o evento nasce corretamente antes do GTM consumir.

### 13.2. Cenários possíveis
- form custom HTML
- script em construtor de páginas
- form embed
- form nativo
- evento vindo do CRM
- evento vindo do n8n

### 13.3. Regras
- a origem deve respeitar a taxonomia
- o payload deve respeitar o contrato
- a origem não precisa ser idêntica entre projetos
- a saída precisa ser consistente entre projetos

### 13.4. Entregáveis possíveis
- template HTML do form
- script de instrumentação
- payload JSON para n8n
- instruções de inserção em builder
- instruções para webhook / CRM

---

## 14. Etapa 7 — Contrato do dataLayer

### 14.1. Objetivo
Fazer o front ou origem do dado falar a mesma língua que o GTM espera.

### 14.2. Regras
- usar eventos canônicos
- usar parâmetros conforme a taxonomia
- gerar `event_id` único
- gerar `event_time`
- incluir `form_id` quando aplicável
- incluir `qualification_rule` para `MQL` e `NOICP`
- incluir `lead_status` quando aplicável

### 14.3. Regra crítica
Campos de layout e CSS não fazem parte do contrato.

---

## 15. Etapa 8 — Parametrização do GTM

### 15.1. Objetivo
Preparar o container para consumir o contrato do projeto.

### 15.2. Fluxo operacional
1. Criar ou selecionar o GTM do projeto
2. Importar template base, se aplicável
3. Instalar o container nas páginas/ativos
4. Atualizar variáveis permanentes `[EDIT]`
5. Atualizar nomes operacionais quando necessário
6. Validar variáveis derivadas
7. Ajustar gatilhos
8. Ajustar ou criar tags específicas

### 15.3. O que normalmente é atualizado no template
- IDs de plataforma
- labels de conversão
- transport URL
- URLs de disparo
- contexto do projeto
- nomes de gatilhos
- eventos personalizados de destino, se houver necessidade operacional

---

## 16. Etapa 9 — Configuração das Variáveis

### 16.1. Objetivo
Fazer o GTM ler corretamente o que vem do `dataLayer` e do ambiente.

### 16.2. Grupos de variáveis
- variáveis permanentes editáveis
- variáveis de `dataLayer`
- variáveis derivadas
- variáveis de cookie/storage
- variáveis de atribuição
- variáveis de user data
- variáveis técnicas

### 16.3. Regra
Variáveis devem ser nomeadas de forma clara e consistente.

Exemplos:
- `DLV | form_id`
- `DLV | lead_status`
- `DLV | qualification_rule`
- `[EDIT] Perma | GA4 ID`

---

## 17. Etapa 10 — Configuração dos Gatilhos

### 17.1. Objetivo
Definir quando as tags disparam.

### 17.2. Regra principal
O gatilho responde ao evento; ele não cria o evento.

### 17.3. Boas práticas
- usar eventos canônicos
- evitar duplicação de gatilhos sem necessidade
- filtrar por contexto só quando necessário
- manter legibilidade operacional

Exemplos:
- `TRG | Custom Event | Lead`
- `TRG | Custom Event | MQL`

---

## 18. Etapa 11 — Configuração das Tags

### 18.1. Objetivo
Enviar os eventos e parâmetros para as plataformas corretas.

### 18.2. Plataformas core
- GA4
- Meta
- Google Ads
- Clarity

### 18.3. Regras
- a tag não inventa lógica de qualificação
- a tag não muda a semântica do evento sem motivo forte
- a tag deve mapear parâmetros de forma coerente
- customização por plataforma deve ser controlada

### 18.4. Casos comuns
- Meta pode exigir tags customizadas quando o evento não for padrão
- Google Ads pode depender de labels específicos por conversão
- GA4 pode receber eventos mais completos e servir de ponte para server-side

---

## 19. Etapa 12 — Integração com n8n e CRM

### 19.1. Quando usar
- quando houver envio para CRM
- quando houver avanço de funnel fora do front
- quando houver necessidade de orquestração server-side

### 19.2. Papel do n8n
- receber payload
- transformar payload
- enviar ao CRM
- enviar ao SGTM, quando necessário
- enriquecer o contexto
- manter coerência semântica

### 19.3. Regras
- o n8n não renomeia arbitrariamente o evento
- o n8n pode enriquecer o payload
- o CRM é a verdade comercial para etapas posteriores ao lead

---

## 20. Distinção entre Eventos de Qualificação

### 20.1. MQL
`MQL` representa o lead qualificado no front, no momento da captura ou imediatamente após o envio do formulário.

Ele depende da regra de qualificação definida para a página, form ou fluxo de entrada.

---

### 20.2. LeadQualified
`LeadQualified` representa uma qualificação posterior no CRM, n8n ou processo comercial.

Ele **não é equivalente** ao `MQL` da página.

---

### 20.3. Regra operacional
`MQL` e `LeadQualified` podem coexistir no mesmo projeto.

Exemplo:
- o lead pode ser `MQL` no front
- e depois virar `LeadQualified` no CRM em outro momento do processo

Essa distinção deve ser mantida no measurement plan, no GTM, no n8n e na documentação final.

---

## 21. Etapa 13 — QA

### 21.1. Definição
QA significa **Quality Assurance**.

No contexto deste processo, QA é a etapa de validação técnica e funcional que confirma que a implementação está correta antes de ser considerada pronta.

Em termos práticos, QA é o teste final de qualidade da implementação.

---

### 21.2. Objetivo
Confirmar que:
- o evento certo dispara na hora certa
- o evento não duplica indevidamente
- os parâmetros corretos existem
- o GTM consome corretamente
- as plataformas recebem corretamente
- o webhook funciona
- o redirect funciona
- o CRM registra corretamente quando aplicável

---

### 21.3. Ferramentas usuais
- GTM Preview
- Chrome DevTools
- console
- Network
- DebugView do GA4
- Meta Test Events
- diagnósticos das plataformas

---

## 22. Etapa 14 — QA Técnico

### 22.1. Checklist mínimo
- o evento certo dispara na hora certa?
- há duplicação?
- `event_id` existe?
- `event_time` existe?
- `form_id` existe quando deveria?
- `lead_status` existe quando deveria?
- `qualification_rule` existe em `MQL` e `NOICP`?
- UTMs foram preservadas?
- user data foi enviado quando aplicável?
- a plataforma recebeu o evento esperado?
- o redirect de jornada está correto?
- o webhook respondeu corretamente?
- o CRM recebeu o payload corretamente?

---

## 23. Etapa 15 — QA Funcional por Evento

### 23.1. PageView
- disparou na página correta?
- contexto da página está preenchido?

### 23.2. CTAInteract
- dispara apenas no CTA correto?
- `cta_text` e `cta_position` foram preservados quando aplicável?

### 23.3. FormStart
- dispara apenas na primeira interação?
- não duplica a cada input?

### 23.4. Lead
- só dispara após envio válido?
- não dispara em erro de validação?

### 23.5. MQL
- dispara apenas quando a regra de qualificação é satisfeita?
- leva `qualification_rule`?
- leva `qualification_reason`, quando houver?

### 23.6. NOICP
- dispara apenas quando não atende o corte?
- leva a regra correta?

### 23.7. LeadQualified
- veio do CRM/n8n no momento correto?
- não está sendo confundido com `MQL`?

### 23.8. CRM events
- payload chegou ao n8n?
- n8n transformou corretamente?
- CRM registrou corretamente?
- evento server-side foi emitido com a semântica esperada?

---

## 24. Etapa 16 — Publicação

### 24.1. Objetivo
Subir a configuração validada para produção.

### 24.2. Regra
Publicar apenas após QA mínimo aprovado.

### 24.3. Checklist antes de publicar
- variáveis revisadas
- gatilhos revisados
- tags revisadas
- ambientes corretos
- URLs corretas
- redirects corretos
- webhook correto
- regra de qualificação correta

---

## 25. Etapa 17 — Handoff e Documentação Final

### 25.1. Objetivo
Encerrar o projeto com informação suficiente para manutenção, leitura e evolução.

### 25.2. Entregáveis mínimos
- inventário técnico final
- measurement plan final
- observações de implementação
- limitações conhecidas
- status das integrações
- funnel coberto
- eventos ativos
- grau de completude

### 25.3. Campo de completude
Registrar o nível de cobertura do projeto:
- `platform_only`
- `platform_crm`

### 25.4. Regra
- `platform_only`: tracking cobre até lead / MQL / NOICP no ambiente web
- `platform_crm`: tracking cobre também etapas posteriores via CRM / n8n / server-side

---

## 26. Critério de Conclusão

O projeto só é considerado concluído quando:
1. O measurement plan foi definido
2. A taxonomia foi aplicada
3. A origem do evento foi preparada
4. O GTM foi parametrizado
5. As integrações foram configuradas
6. O QA foi executado
7. O nível de completude foi registrado
8. A documentação final foi entregue

---

## 27. Anti-patterns Operacionais

- Começar pelo GTM sem measurement plan
- Criar evento fora da taxonomia
- Colocar contexto no nome do evento
- Fazer o GTM decidir a qualificação
- Publicar sem QA
- Não registrar IDs e labels
- Não documentar webhook e redirects
- Não documentar a regra de qualificação
- Assumir que todos os forms terão nome, telefone e email
- Assumir um único formato de formulário
- Misturar setup de origem com setup de consumo sem clareza

---

## 28. Estrutura Recomendada de Entregáveis por Projeto

- `measurement-plan.md`
- `inventario-tecnico.md`
- `instrumentation-notes.md`
- `gtm-notes.md`
- `qa-checklist.md`

Opcionalmente:
- `form-template.html`
- `page-script.js`
- `n8n-payload.json`

---

## 29. Próximos Passos

1. Revisar este SOP
2. Fechar versão final
3. Revisar briefing da skill `Instrumentation Engineer`
4. Revisar briefing da skill `Tracking Engineer`
5. Evoluir depois para SOP de server-side

---
