# Proposta Técnica — como preencher o template do NTec

O entregável da skill é a **Proposta Técnica** no template oficial do NTec ("Comercial → Validação de Projeto"). É o documento que o comercial manda para a validação técnica. A skill monta um JSON e roda `scripts/gerar_proposta.py`, que gera o PDF no visual do template: faixa azul-marinho no topo, selo laranja OBRIGATÓRIO, as 14 seções, o checklist final e um **Anexo — Memória do dimensionamento**.

Modelo completo do JSON: `exemplos/ficticio-petagenda-proposta.json`.

## Regras do preenchimento

- **Fonte de tudo é a PRD**, mais as transcrições do Fathom (pelo mapa das falas, `leitura-prd.md`, Passo 4) e o CN, como no resto da skill. A proposta não acrescenta módulo, perfil, integração, log ou permissão que a PRD não tem. Se o template pede algo que a PRD não traz, escrever "não previsto no PRD". Se isso muda o prazo, vira ponto em aberto (seção 11).
- **As semanas não são digitadas.** O script recalcula a faixa a partir de `dimensionamento.entrada`, que é o mesmo JSON da régua (`calcular_faixa.py`). Caixa de status, cronograma e anexo saem do mesmo cálculo.
- **Lacunas do Fathom na caixa de status.** Com `lacunas_fathom` na entrada, a caixa de status mostra a venda com "+" ("14 semanas = 12 + 2 que dependem do Fathom") (`leitura-prd.md`, Passo 4). O cronograma segue a venda com as lacunas.
- **Nada interno no corpo** (seções 1 a 14): sem `(origem: ...)` nem nome de arquivo da régua. As origens ficam no anexo (critérios, suposições) e no chat.
- **Texto curto e sem adjetivo de venda.** Todo termo técnico é explicado na primeira vez que aparece.

## Cabeçalho (`cabecalho`)

| Campo | De onde vem |
|---|---|
| `cliente_projeto` | Nome da PRD + subtítulo ("PetAgenda · Agenda e carteirinha de vacina para rede de clínicas veterinárias") |
| `responsavel_comercial`, `data_envio` | Do CN. Sem resposta: deixar `""`, e o PDF sai com `______` para preencher à mão |
| `prd_link`, `prd_paginas` | Link ou nome do PDF da PRD e o número de páginas (inteiro). O template exige de 5 a ~10; o script aceita até 12 |
| `prototipo_link` | Do CN. Sem link, fica `""` e o checklist sai PENDENTE |

`status`: `pronto`, `atencao`, `preliminar` (sem PRD) ou `parado`, com os mesmos critérios de antes.

## Seção por seção

| Seção | Campo no JSON | Como preencher a partir da PRD |
|---|---|---|
| 1. Problema | `problema.dor`, `problema.problemas` (2 a 4) | Visão Geral da PRD (dor central e objetivo). Cada problema é uma dor concreta que um módulo resolve |
| 2. Visão geral | `visao_geral.tipo`, `publico`, `proposta_valor`, `principios` | Ambientes de entrega (3.1) e superfícies; proposta de valor em 1 frase; princípios = hierarquia, regras de arquitetura ditas na PRD (offline-first, mobile-first) |
| 3. Entidades | `entidades.centrais`, `entidades.apoio` | Substantivos que os módulos criam ou consultam. Centrais = o que o fluxo principal produz; apoio = cadastros e registros auxiliares |
| 4. Perfis **(obrig.)** | `perfis.tabela` (`[perfil, o que faz, observações]`), `perfis.matriz_permissoes` | PRD 2.1. Observação = superfície, filtros no servidor, Hardcoded. Matriz: dizer se existe ou se está em aberto (se a PRD exclui permissões granulares, dizer isso) |
| 5. Fluxo | `fluxo` (lista de etapas `etapa, quem, dados, validacoes, pendente`) | A jornada principal da Visão Geral, quebrada pelos módulos na ordem de uso. `pendente` marca a etapa que depende de decisão futura do cliente e precisa ter o mesmo assunto na seção 11 |
| 6. Regras **(obrig.)** | `regras` (`[regra, o que garante]`) | Todas as RNs da PRD, os itens Hardcoded e os SID que são regra (filtro por perfil, validação no servidor). Regra crítica (duplicidade, conflito, limite) não pode faltar |
| 7. Arquitetura | `arquitetura` (`[rótulo, texto]`) | PRD 3.1: frontend por superfície, backend, banco, autenticação, integrações externas (cada `SID integrar com...`). Sem stack aqui: stack é a seção 12 |
| 8. Segurança **(obrig.)** | `seguranca` (`[rótulo, texto]`) | PRD 3.4 + 2.1. Sempre cobrir: tipos de dado (pessoal? sensível? LGPD?), login e controle por perfil, histórico e logs, dados públicos (links sem login, formulários públicos). O que a PRD não traz: "não previsto no PRD" |
| 9. Módulos críticos | `modulos_criticos` (`nome, detalhe, mvp, futuro`) | Pagamentos, cada integração externa e as automações (offline com sincronização, rotina agendada, geração de documentos, importação). O que entra no MVP × o que está nas Exclusões ou depende de terceiro. Sem nenhum: lista vazia |
| 10. Roadmap **(obrig.)** | `roadmap` (`[fase, entrega, natureza]`) | Cada módulo da PRD = uma entrega *Prioritário*. Item na conta que depende da seção 11 continua *Prioritário*, com "(depende do ponto em aberto N)". *A confirmar* = algo discutido que não está na conta. *Futuro* = Exclusões que o cliente pode querer depois. **Nada A confirmar ou Futuro pode aparecer no cronograma** (o script confere pelo texto da entrega) |
| 11. Pontos em aberto **(obrig.)** | `pontos_abertos` (lista de textos) | Nesta ordem: lacunas que mudam a faixa, lacunas da PRD (termo sem mecanismo, contradição, etapa sem dono), dependências externas (API, fornecedor, conta de loja), e as lacunas que não mudam a faixa (data de início). Cada um diz o valor usado na conta, quando houver. **Nunca vazio** |
| 12. Stack | `stack.itens`, `stack.nao_validado` | O template NTec (Expo, Next.js, MongoDB + Prisma, Better Auth, S3, Vercel, Vitest e Playwright), só as peças que o projeto usa. O primeiro item de `nao_validado` é sempre "Toda a stack é premissa a validar na Sprint de Refino e Prototipação". Exigência do cliente (`stack_fora_template`) entra aqui também como não validada |
| 13. Cronograma | `cronograma.sprints` (`[sprint, entregas, depende_de]`), `cronograma.duracao_obs` | Ver abaixo |
| 14. Protótipo **(obrig.)** | `prototipo.recebido`, `prototipo.tabela` (`[tela, regra, adaptação, justificativa, vale p/ MVP]`), `prototipo.declaracao` | Só com protótipo recebido e validado pelo cliente. Sem adaptação: `declaracao` = "O protótipo segue integralmente as regras do PRD". Sem protótipo: `recebido: false`, tabela vazia; o PDF avisa o comercial |

`dependencia_externa_sem_confirmacao`: `true` se alguma integração ou fornecedor está sendo tratado como certo sem confirmação. Com a regra de peso fixo e a cláusula de contrato, o normal é `false`.

## O que vem das transcrições, seção por seção

A PRD decide o escopo; as transcrições dão o resto. Toda seção consulta o mapa das falas (`leitura-prd.md`, Passo 4) antes de ser escrita. O limite é um só: **nada que esteja só na transcrição entra na conta ou vira entrega *Prioritário***. O que foi discutido e não está na PRD vira *A confirmar* (seção 10) ou ponto em aberto (seção 11).

| Seção | O que tirar das transcrições |
|---|---|
| Cabeçalho | `responsavel_comercial` = dono do card no Pipedrive, quando houver |
| 1. Problema | A dor e o impacto com as palavras do cliente: o que dá errado hoje, com que frequência, quanto custa |
| 2. Visão geral | Quem usa e quantos, como o cliente descreveu; a referência de produto que ele citou |
| 3. Entidades | O vocabulário do cliente para cada entidade da PRD ("fluxo", "solicitação", "chamado") |
| 4. Perfis | Quem faz o quê na operação, como o cliente contou; confirma a matriz da PRD |
| 5. Fluxo | A jornada na ordem em que o cliente contou; `pendente` nas etapas que ele deixou para decidir |
| 6. Regras | Regras que o cliente explicou (alçada de aprovação, SLA, versionamento). Regra dita e ausente da PRD vai para a seção 11 |
| 7. Arquitetura | Sistemas atuais que o cliente citou (ERP, BI, sistemas web próprios) e como eles conversam hoje |
| 8. Segurança | Dados sensíveis, LGPD, acesso público ou privado, quem pode ver o quê, quando o cliente falou disso |
| 9. Módulos críticos | Integração, automação ou documento que o cliente descreveu como crítico, com o sistema do outro lado |
| 10. Roadmap | *A confirmar* = discutido nas transcrições e fora da PRD; *Futuro* = Exclusões e o que o cliente disse que pode ficar para depois |
| 11. Pontos em aberto | Lacunas, divergências entre transcrição e PRD, decisões que o cliente deixou em aberto e reuniões do card sem gravação |
| 12. Stack | Exigência de tecnologia ou de infra dita pelo cliente; sistemas com que a solução precisa conviver |
| 13. Cronograma | Data de início, recesso e calendário citados; quem testa e com que frequência. A próxima reunião combinada não é data de início |
| 14. Protótipo | Se uma transcrição mostra protótipo apresentado, a data e a reação do cliente. O link continua vindo do CN |
| Anexo | `criterios` citam PRD e transcrição; `precedente` se justifica pelo produto que o cliente descreveu; `cortes` respeitam as prioridades dele; `suposicoes` começa pela lista de transcrições lidas e de reuniões do card sem gravação |

## Cronograma sprint a sprint (seção 13)

Usar os valores **centrais** de cada fase do `calcular_faixa.py` e encaixar em sprints de 2 semanas até fechar a recomendação de venda (provável arredondado para sprints inteiras).

1. **F1** (imersão e setup): Sprint 1, ou 1 e 2 se F1 passar de 2 semanas. Inclui validar o protótipo e, se houver app nas lojas, pedir as contas Apple e Google.
2. **F2** (acesso e login): a sprint seguinte. Se sobrar meia sprint, começar os cadastros.
3. **F3** (desenvolvimento): os módulos da PRD na ordem da jornada, de 1 a 2 módulos por sprint conforme o peso. Módulo com RN específica ou offline com sincronização ocupa a sprint inteira ou mais. Integração com sistema do cliente vai depois da sprint em que vence o prazo da cláusula de acesso à API.
4. **F4** (testes com o cliente), **F5** (deploy) e **F6** (lojas) no fim, nessa ordem. Em F6, contar o tempo de aprovação das lojas.
5. Toda sprint cuja entrega depende de um ponto da seção 11 leva `depende_de` = "ponto N".
6. O número de sprints precisa bater com a recomendação. Se a soma dos centrais for quebrada, a folga fica na última sprint ("correções finais e entrega").

## Anexo — Memória do dimensionamento (`dimensionamento`)

É o que a área de projetos revisa. O PDF mostra a faixa e o detalhamento F1 a F6 calculados pelo script, os alertas da régua e os campos abaixo.

| Campo | Conteúdo (o mesmo de antes da proposta) |
|---|---|
| `entrada` | O JSON da régua, igual ao de `calcular_faixa.py` |
| `criterios` | `{"F1": "...", ..., "F6": "..."}`: o critério de cada fase, citando a seção da PRD (ex.: "F3: módulos 2.2 a 2.5 = 4 fluxos comuns; 2.4 tem concorrência → RN específica") |
| `atencao` | Pontos de atenção, do mais grave para o menos grave, cada um com o que fazer. Toda integração com sistema do cliente traz aqui a cláusula de contrato pronta de `regua-dimensionamento.md` |
| `lacunas` | `[fator, procurado e não achado, pergunta para o cliente, valor provisório]` |
| `precedente` | O projeto mais parecido em `exemplos/`, o prazo dele e por que é comparável (ou "sem precedente comparável"). Mais de 30% de distância: explicar |
| `cortes` | 2 a 4 cortes `[corte, economia]`, recalculados com o script. Nunca algo das Exclusões. Com integração de sistema do cliente, um corte é sempre o plano B (planilha) |
| `suposicoes`, `fronteira`, `imersao` | Suposições (inclui o valor provisório de cada lacuna), os comentários de duas linhas de `regra-fronteira-e-faixa.md` e os pontos para a imersão da PRD |

## Onde foi parar cada parte da resposta antiga

| Antes (resposta no chat) | Agora |
|---|---|
| Status, prazo em uma frase, faixa | Caixa de status do PDF + chat |
| Resumo do projeto | Seções 1 e 2 |
| Pontos de atenção | Anexo `atencao` (os que dependem de decisão externa também na seção 11) |
| Lacunas para o CN | Seção 11 + anexo `lacunas` + chat |
| Detalhamento por fase | Anexo (calculado pelo script) + seção 13 (em sprints) |
| Precedente, O que cortar, Observações | Anexo |
| Bloco para colar no Dimensionamento Tec | Saiu: o template de Proposta Técnica substitui o formulário |

## Checklist final (o script confere)

| Item do template | Como o script decide |
|---|---|
| PRD (5 a ~10 páginas) e protótipo anexados | `prd_link` preenchido, `prd_paginas` entre 5 e 12 e `prototipo_link` preenchido |
| Campos OBRIGATÓRIO preenchidos | Seções 4, 6, 8, 10, 11 e 14 não vazias |
| Protótipo segue as regras ou adaptações justificadas | `prototipo.recebido` e tabela ou declaração preenchidas |
| Roadmap e cronograma não se contradizem | Nenhuma entrega *A confirmar* ou *Futuro* aparece no texto do cronograma |
| Pontos em aberto listados | Seção 11 não vazia |
| Nenhuma dependência externa tratada como certa | `dependencia_externa_sem_confirmacao` é `false` |

Item PENDENTE não impede o PDF. Ele sai marcado e vai para o chat com o que o CN precisa fazer.
