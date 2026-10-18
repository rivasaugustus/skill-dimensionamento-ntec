# Leitura da PRD — do documento de requisitos para os fatores da régua

A **PRD** (Documento de Requisitos de produto do NTec) é a entrada padrão desta skill. Ela é gerada pela skill `documento-requisitos-ntec` ([repositório](https://github.com/rivasaugustus/documento-requisitos-ntec)) a partir das reuniões com o cliente e vira anexo de contrato. Esta skill roda **depois** dela.

Este arquivo diz onde, na PRD, está cada fator da régua e como traduzi-lo para o JSON do `scripts/calcular_faixa.py`.

---

## Passo 1 — O que o CN manda

| Peça | De onde vem | Como usar |
|---|---|---|
| **PDF "[Requisitos] {projeto}"** (ou o `.md` que gerou o PDF) | Saída da skill da PRD | **Fonte principal.** Seções 1 (Visão Geral), 2 (Requisitos Funcionais por módulo), 3 (Não Funcionais) e 4 (Exclusões) |
| **Bloco "Inventário para dimensionamento"** | Chat da skill da PRD, logo depois do PDF | Atalho: uma linha por módulo, com `Módulo \| Superfície \| Itens A(cli)D \| Itens SID \| Integrações \| Fluxos de dados \| Fatores de complexidade`. Se o CN não colar, **montar o inventário a partir do PDF** com as mesmas colunas |
| **Bloco "Pontos para a imersão"** | Chat da skill da PRD | Detalhes que não mudam o tamanho. Não somam prazo: entram como atividades da imersão (F1) na resposta |
| Blocos "Perfis de acesso", "Funcionalidades" e "Integrações" | Chat da skill da PRD | Conferência. As fontes (`[Fathom · AT 12/03]`, `[chat]`…) mostram de onde veio cada item |
| **Reuniões do cliente no Fathom** | **A própria skill busca** pelo conector (`fathom.md`): as mesmas reuniões que geraram a PRD. O CN só cola transcrição ou link (ou resumo do Granola) se a busca não funcionar | Responder o Passo 4 sem perguntar ao CN. O que não estiver nelas vira **lacuna** |

**Sem PRD** (só o formulário Dimensionamento Tec, uma transcrição ou um briefing): dimensionar mesmo assim pelo caminho de `insumos-e-perguntas.md`, mas o status nunca é ✅. Marcar ⚠️ **Estimativa preliminar, sem PRD** e dizer ao CN para rodar a skill da PRD antes de fechar a proposta.

**PDF e inventário divergem** (ex.: o inventário tem um módulo que o PDF não tem): vale o PDF, que é o anexo de contrato. Registrar a divergência em *Pontos de atenção*.

## Passo 2 — Onde está cada fator

| Fator da régua (campo do JSON) | Onde ler na PRD | Como traduzir |
|---|---|---|
| `formatos` | Linha `**Superfície:**` de cada módulo + 3.1 *Ambientes de entrega* | `app mobile` → `"app"`. `site` ou `painel web` → `"web"` (ou `"pwa"` se a PRD disser que instala pelo navegador). App mobile + painel web = dois formatos. Site + painel web no mesmo sistema = um formato só |
| `lojas` | Módulo *Disponibilização do Produto* + 3.1 | Publicação na App Store ou Play Store → `"normal"`. Se a 3.4 fala de dado de saúde, financeiro ou de menores → `"sensivel"`. Sem loja → `"nao"` |
| `niveis_acesso` | 2.1 *Autenticação e Perfis de Acesso* | Contar os perfis. Perfil com módulo ou superfície própria, ou com aprovação de cadastro → "grandes diferenças". Só ações a mais no mesmo módulo → "poucas diferenças". Sem 2.1 (sem login) → `"nenhum"` |
| `multitenancy` | Visão Geral, 2.1, RNs e 3.4 (termos: multiempresa, por unidade, por clínica, por filial) | Só o dado separado por empresa → `"separado"` ou `"isolado"` (perguntar qual, se a PRD não disser). Regras ou funções diferentes por empresa → `"instancias"` (🚩) |
| `fluxos_comuns` | Módulos da seção 2 | **1 módulo = 1 fluxo.** Não contam: 2.1 Autenticação (vai para F2) e *Disponibilização do Produto* (vai para F5 e F6) |
| `fluxos_rn_especifica` | Módulos com `### Regras de Negócio` + coluna *Fatores de complexidade* | O módulo sai de `fluxos_comuns` e entra aqui quando tem RN de cálculo, fórmula, regra clínica, legal ou de dinheiro, ou quando tem um dos fatores da tabela do Passo 3 que viram RN específica. Conta **uma vez por módulo**, mesmo com várias RNs |
| `features_atipicas` | Coluna *Fatores de complexidade* | Só pelos fatores da tabela do Passo 3 que viram feature atípica. O "produto sem benchmark" não está na PRD: procurar no Fathom (Passo 4) |
| `integracoes` | Itens `**SID** integrar com…` (Sentido, Dados, Meio) + coluna *Integrações* | **Peso fixo, sem perguntar ao CN** (regra em `regua-dimensionamento.md`, F3). Sentido `envia` ou `recebe` → unidirecional. `envia e recebe` → bidirecional. Serviço de mercado (pagamento, mapa, push, IA, WhatsApp Business, NF-e…) → `_doc`. Sistema do cliente (ERP, legado, sob medida), meio `arquivo` ou dúvida → `_sem_doc`, com cláusula de contrato e plano B. `"sem_api"` (🚩) só se a PRD disser que o sistema não tem API. Google, e-mail e upload de arquivo → `"template"` |
| `dependencia_dados` | Fluxos com `**Origem:**` / `**Processamento:**` / `**Destino:**` | Origem em planilha, base pública, CSV ou sistema antigo → `true` (e chamar o núcleo de Dados). Importação, seja carga única ou recorrente, também. Consulta automática a um serviço é integração, não dependência de dados |
| `offline` | Termo "offline" (a PRD sempre diz o mecanismo) | Só consulta sem internet → `"local"`. Registro com sincronização depois → `"sync"` |
| `volume_alto` | 3.3 *Performance* | Milhares de usuários simultâneos ou picos declarados → `true`. Sem 3.3 → `false` |
| `rn_critica` (F4) | RNs e 3.4 | RN de saúde, dinheiro, cálculo clínico ou legal → `true` |
| `demanda_pouco_detalhada` | — | **`false`.** A PRD é o documento de requisitos |
| `stack_fora_template` | 3.1 (serviços de terceiros definidos com o cliente) | A PRD não fixa stack de propósito. Só `true` se a 3.1 citar uma tecnologia exigida pelo cliente. Se não citar, procurar no Fathom (Passo 4) |
| `deploy_infra_cliente` | 3.1 | `true` se a 3.1 disser que roda nos servidores ou na nuvem do cliente. Se não disser, procurar no Fathom (Passo 4) |
| `atipico`, `cliente_pouca_disponibilidade` | — | Não estão na PRD. Procurar no Fathom; se não estiver, lacuna (Passo 4) |

**Hardcoded** (decisão fixa do cliente) não muda nenhum peso. Ele só confirma que não há tela de configuração daquele item, e a skill não deve somar tela de configuração por conta própria.

**Exclusões Explícitas (seção 4)** não somam nada. Usar para duas coisas: conferir que nenhum item excluído foi contado e montar a seção "O que cortar" (um corte não pode ser algo que já está excluído).

## Passo 3 — Fatores de complexidade da PRD que a régua não tem

A PRD marca fatores que não têm linha própria na régua. Eles **encaixam nas linhas que já existem**, sem peso novo (não há dado de calibração para criar pesos).

| Fator na PRD | Linha da régua | Observação |
|---|---|---|
| Pagamento (gateway, Pix, cartão) | Integração `bi_doc` (cobra e recebe a confirmação) | Gateway é serviço de mercado. `bi_sem_doc` só se o pagamento passar pelo sistema próprio do cliente |
| IA (classificar, extrair, gerar, responder) | Integração `uni_doc` (API do modelo) | Se a PRD disser que a IA decide sem humano no fim, marcar também ⚠️ em *Pontos de atenção* |
| Notificação por push no app | Integração `uni_doc` | |
| Notificação por WhatsApp automático | Integração `uni_doc`, ou `bi_doc` se o sistema lê a resposta | Link para abrir o WhatsApp não é integração (0) |
| Notificação por e-mail | `template` | |
| Aviso dentro do sistema | 0 (faz parte do módulo) | |
| Tempo real por aviso do servidor (push) | +1 em `features_atipicas` | Atualização periódica (polling) = 0 |
| Relatório configurável pelo usuário (construtor) | +1 em `features_atipicas` | Tela fixa ou exportação para PDF ou Excel = 0 (faz parte do módulo) |
| Concorrência (ex.: aceite atômico, "o primeiro que aceitar leva") | O módulo vira `fluxos_rn_especifica` | |
| Imutabilidade ou versionamento (ex.: prontuário que não se edita) | O módulo vira `fluxos_rn_especifica` | |
| Rotina agendada (gatilho por horário, não por evento) | O módulo vira `fluxos_rn_especifica` | Gatilho por evento no sistema = 0 |
| Upload e armazenamento de arquivos | `template` (S3) | |
| Importação ou migração | `dependencia_dados: true` | |
| Multiempresa | `multitenancy` (ver Passo 2) | |
| Offline | `offline` (ver Passo 2) | |

**Não contar duas vezes.** O mesmo módulo vira RN específica uma vez só, mesmo que tenha concorrência **e** imutabilidade. Um fator que já virou integração não vira também feature atípica (a exceção é a IA que decide sozinha, que só ganha ⚠️, sem peso).

## Passo 4 — O que a PRD não responde: transcrição do Fathom primeiro, lacuna para o CN depois

A PRD descreve o produto, não o projeto. Os quatro fatores abaixo ficam de fora dela, mas quase sempre aparecem nas reuniões com o cliente. A skill **busca sozinha as reuniões do cliente no Fathom** (`fathom.md`) e procura cada um nas transcrições antes de perguntar qualquer coisa. O que não estiver lá vira **lacuna** e volta para o CN.

**Busca sem resultado** (sem conector, erro de login ou nenhuma reunião do cliente): pedir uma vez ao CN o link da reunião ou a transcrição colada (`fathom.md`, Passos 1 e 2). Se ele não tiver, os quatro fatores viram lacuna.

| Fator | O que procurar na transcrição | Como traduzir | Pergunta da lacuna (pronta para o CN repassar) |
|---|---|---|---|
| Disponibilidade do cliente (`cliente_pouca_disponibilidade`) | Quem vai testar e aprovar, quantas horas, agenda ("sou plantonista", "a gente tem pouco tempo", "a Ana valida toda sexta") | Pessoa definida com tempo fixo → `false`. Agenda apertada, "depende", ninguém definido → `true` | "Quem do cliente vai testar e aprovar as entregas? Essa pessoa tem umas 2 horas por semana para isso?" |
| Stack e infra (`stack_fora_template`, `deploy_infra_cliente`) | Tecnologia exigida ("tem que ser em…"), "nossos servidores", "a nossa TI hospeda", nuvem própria | Exigência explícita → `true`. "Vocês decidem", "tanto faz" → `false`. Pular se a 3.1 da PRD já responde | "O cliente exige alguma tecnologia, ou que o sistema fique nos servidores dele?" |
| Atípico (`atipico`, `features_atipicas`) | App ou site citado como referência ("tipo o…", "igual ao…", "vi um que…") | Referência citada → `false`. "Não existe nada parecido" → `true` | "Existe algum app ou site parecido que a gente possa usar de referência?" |
| Semanas off | Data ou mês de início, prazo de lançamento | Data → contar férias, provas e recesso | "Qual é a data prevista de início? (para contar férias, provas e recesso)" |

**O que conta como resposta encontrada:**
- O cliente (ou o CN, na reunião) **disse explicitamente**. Registrar com a reunião e um trecho curto: `(origem: Fathom · 12/03 — "a Ana valida toda sexta")`.
- Duas reuniões dizem coisas diferentes → vale a mais recente, e a divergência vai para *Pontos de atenção*. Se não der para saber qual vale, é lacuna.
- **Não é resposta:** dedução a partir do tom da conversa, silêncio sobre o assunto, ou fala vaga ("vamos ver", "acho que sim"). Tudo isso é lacuna.

**O que fazer com a lacuna:**
1. Dimensionar mesmo assim com o **valor mais pesado plausível** (`cliente_pouca_disponibilidade: true`, `stack_fora_template: true` só se houver algum indício, `atipico: true` só se nenhuma referência foi citada). Semanas off sem data de início: não somar, e dizer isso no prazo em uma frase.
2. **Mostrar quanto da venda depende das lacunas.** No JSON da régua, listar os fatores em lacuna em `"lacunas_fathom"` (ex.: `["cliente_pouca_disponibilidade", "atipico"]`) e marcar `"fathom": true` se a busca achou e leu reuniões do cliente (o Fathom existe, mas não fala do assunto). O script refaz a conta com esses fatores no valor leve e diz a diferença em sprints inteiras. O prazo vende o número com as lacunas e mostra a parte delas com "+": *"Recomendado vender 14 semanas (7 sprints) = 12 + 2 que dependem do Fathom (disponibilidade do cliente, referência de produto), mais as semanas off."* Confirmando as lacunas no leve, a venda cai para o primeiro número. Com Fathom que não fala do assunto, a frase diz "dependem de confirmar com o cliente". Se a diferença não muda as sprints, não há "+".
3. Listar na seção **5. Lacunas para o CN** da resposta: o fator, o que foi procurado e não achado, a pergunta pronta e o valor provisório usado na conta.
4. Registrar o valor provisório em *Suposições*, com `(origem: lacuna — não está no Fathom)`.
5. **Status:** lacuna em disponibilidade, stack/infra ou atípico mexe na faixa → ⚠️ *Precisa de revisão* até o CN responder. Lacuna só na data de início não muda a faixa: não rebaixa o status.

Quando o CN responder uma lacuna, trocar o valor provisório, tirar o fator de `lacunas_fathom`, rodar o script de novo e mudar a origem para `(origem: CN)`.

**Não perguntar sobre a documentação das integrações.** O CN não tem como responder: a skill aplica o peso fixo do Passo 2 e registra a classificação em *Suposições* (ex.: `Integração ERP: bidirecional com sistema do cliente (origem: PRD 2.4 + regra de peso fixo)`).

**Lacuna dentro da PRD** (módulo sem superfície, integração sem meio, fluxo de dados sem uma das três partes, "tempo real" sem mecanismo): não deveria acontecer, porque a skill da PRD pergunta antes de gerar o PDF. Se aparecer, tratar como fator desconhecido (perguntar, ou usar o valor mais pesado) e registrar em *Pontos de atenção* que a PRD precisa de revisão antes de virar anexo de contrato.

## Passo 5 — Origem de cada fator

Com PRD, a origem cita a seção:

```
Níveis de acesso: 2 a 6 com grandes diferenças (origem: PRD 2.1 — 4 perfis, recepção e veterinário com módulos próprios)
Fluxos comuns: 4 (origem: PRD 2.2 a 2.5)
Integração Mercado Pago: bidirecional com serviço de mercado (origem: PRD 2.3 + regra de peso fixo)
Disponibilidade do cliente: boa (origem: Fathom · 12/03 — "a Ana valida toda sexta")
Data de início: lacuna (origem: lacuna — não está no Fathom)
```
