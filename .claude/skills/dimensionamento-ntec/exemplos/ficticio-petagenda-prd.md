# Exemplo de leitura da PRD (caso fictício)

Este caso **não é um projeto real**. É o mesmo PetAgenda de `ficticio-petagenda.md`, só que agora a entrada é uma **PRD** gerada pela skill `documento-requisitos-ntec`. O objetivo é mostrar como `leitura-prd.md` transforma o documento nos fatores da régua. A resposta completa ao CN segue o formato do `SKILL.md`, igual à do outro exemplo. Entrada do script: `ficticio-petagenda-prd.json`.

---

## O que chegou

**PDF "[Requisitos] PetAgenda"** (resumo das seções que importam):

- **1. Visão Geral:** app mobile para tutores, publicado nas lojas, e painel web para as clínicas de uma rede de 3 unidades. Cada clínica vê só os próprios dados, e as regras são as mesmas para todas.
- **2.1 Autenticação e Perfis de Acesso:** 4 perfis. Tutor (agenda, paga, vê a carteirinha), Recepção (gerencia a agenda da sua clínica), Veterinário (registra atendimento e vacina) e Admin da rede (cadastra clínicas e usuários).
- **2.2 Agendamento e Pagamento.** Superfície: app mobile e painel web. Inclui `SID integrar com o Mercado Pago` (Sentido: envia e recebe · Dados: cobrança e confirmação · Meio: API do fornecedor).
- **2.3 Carteirinha de Vacina e Lembretes.** Superfície: app mobile. Lembrete disparado por agendamento na data da próxima dose, por push no app.
- **2.4 Registro de Atendimento.** Superfície: painel web.
- **2.5 Gestão da Agenda da Clínica.** Superfície: painel web.
- **2.6 Disponibilização do Produto:** App Store e Play Store, painel na web.
- **3.1:** sem stack exigida, sem infra do cliente. Sem 3.3.
- **4. Exclusões:** nota fiscal, conciliação financeira, migração do histórico das clínicas, prontuário completo.

**Inventário para dimensionamento** (colado pelo CN):

| Módulo | Superfície | Itens A(cli)D | Itens SID | Integrações | Fluxos de dados | Fatores de complexidade |
|---|---|---|---|---|---|---|
| 2.1 Autenticação e Perfis | app mobile e painel web | 3 | 3 | — | — | multiempresa: dados por clínica |
| 2.2 Agendamento e Pagamento | app mobile e painel web | 5 | 4 | Mercado Pago (envia e recebe) | — | pagamento |
| 2.3 Carteirinha e Lembretes | app mobile | 3 | 3 | push no app | — | notificação: push · rotina agendada |
| 2.4 Registro de Atendimento | painel web | 4 | 2 | — | — | — |
| 2.5 Gestão da Agenda | painel web | 4 | 3 | — | — | — |

**Pontos para a imersão:** duração padrão de cada tipo de consulta; campos da carteirinha; texto dos lembretes.

**Transcrições do Fathom** (2 reuniões, de 12/03 e 19/03, puxadas pela própria skill no conector, sem o CN colar nada; `leitura-prd.md`, Passo 4). O que a PRD não responde foi procurado nelas (`leitura-prd.md`, Passo 4):

| Fator | Achado no Fathom? | Valor | Origem |
|---|---|---|---|
| Disponibilidade do cliente | Sim | boa (`false`) | Fathom · 12/03 — "a Carla, da recepção, testa tudo toda sexta de manhã" |
| Stack e infra | Sim | template, infra NTec (`false`) | Fathom · 12/03 — "a parte técnica vocês que decidem" |
| Atípico | Sim | não (`false`) | Fathom · 19/03 — "tipo aquele app de agendar banho e tosa" |
| Data de início | **Não** | lacuna: semanas off não somadas | Nenhuma das reuniões fala de início |

**5. Lacunas para o CN** (como sai na resposta):

| Fator | Procurado no Fathom e não achado | Pergunta para o cliente | Valor provisório na conta |
|---|---|---|---|
| Semanas off | Data ou mês de início, prazo de lançamento | "Qual é a data prevista de início? (para contar férias, provas e recesso)" | Semanas off não somadas |

A única lacuna é a data de início, que não muda a faixa. Por isso o status não cai para ⚠️.

---

## Tradução para a régua

| Fator | Valor | Origem |
|---|---|---|
| Formatos | app + web → x1,3 | PRD 2.2 a 2.5 (superfícies) e 2.6 |
| Lojas | normal | PRD 2.6. Dado de pet não é sensível |
| Níveis de acesso | 2 a 6, grandes diferenças | PRD 2.1: Recepção e Veterinário têm módulos próprios (2.4 e 2.5) |
| Multitenancy | dados separados | PRD 1: "cada clínica vê só os próprios dados, regras iguais" |
| Fluxos comuns | 3 (2.2, 2.4, 2.5) | PRD seção 2. 2.1 vai para F2 e 2.6 vai para F5 e F6 |
| Fluxos com RN específica | 1 (2.3) | Rotina agendada (lembrete na data da dose) → o módulo vira RN específica (`leitura-prd.md`, Passo 3) |
| Integrações | Mercado Pago `bi_doc`; push `uni_doc`; login e e-mail `template` | PRD 2.2 e 2.3 + regra de peso fixo: Mercado Pago e push são serviços de mercado (sem pergunta ao CN) |
| Demanda pouco detalhada | não | Tem PRD |
| Disponibilidade do cliente | boa | Fathom · 12/03 |

**Exclusões conferidas:** nota fiscal, conciliação e migração não foram contadas. Também não podem aparecer em "O que cortar", porque já estão fora.

## Resultado do script

| Cenário | Semanas | Sprints |
|---|---|---|
| Otimista (piso) | 15 | 8 |
| **Provável (recomendado)** | **20** | **10** |
| Conservador (teto de risco) | 28 | 14 |

## Comparação com o mesmo caso sem PRD

| | Sem PRD (`ficticio-petagenda.md`) | Com PRD |
|---|---|---|
| F1 Imersão | 3 (demanda sem documento) | 2 (tem PRD) |
| F3 Desenvolvimento | 4 fluxos comuns | 3 comuns + 1 com RN (lembrete por rotina agendada) |
| Provável | 19 → vender 20 | 20 → vender 20 |
| Status | ⚠️ preliminar, sem PRD | ✅ pronto para proposta |

A PRD tirou uma semana da imersão, mas mostrou que o lembrete é disparado por agendamento, coisa que a call não deixava claro. Com a regra "rotina agendada → RN específica", o módulo da carteirinha pesa mais. O provável quase não mudou, mas agora cada número aponta para uma seção do anexo de contrato.

Se a rotina agendada contasse como fluxo comum, o provável cairia para 17,5 semanas (vender 18). Essa regra ainda **não foi calibrada** com projeto real e é a primeira a conferir quando houver um projeto do NTec com PRD terminado.
