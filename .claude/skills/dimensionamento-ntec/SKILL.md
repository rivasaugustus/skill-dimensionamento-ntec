---
name: dimensionamento-ntec
description: Dimensiona propostas de software do Núcleo de Tecnologia (NTec) da Poli Júnior — sites, webapps, PWAs, apps mobile e painéis — e devolve uma faixa de semanas (otimista, provável e conservador) com os riscos que a área de projetos precisa ver, entregue no template "Proposta Técnica" do NTec (PDF com as 14 seções, cronograma sprint a sprint e memória do dimensionamento). Puxa sozinha as reuniões do cliente no Fathom (conector Fathom) para responder o que a PRD não cobre, sem o CN colar transcrição. Use também quando pedirem a proposta técnica para a validação de projeto. Feita para coordenadores de negócios (CNs) sem formação técnica. A entrada padrão é a PRD (Documento de Requisitos "[Requisitos] {projeto}" gerado pela skill documento-requisitos-ntec), e esta skill roda logo depois dela. Use quando alguém pedir para dimensionar, estimar prazo, validar ou "ver quantas sprints dá" um projeto do NTec; quando mandarem a PRD, o PDF de requisitos ou o bloco "Inventário para dimensionamento"; quando colarem o formulário Dimensionamento Tec, uma ficha de qualificação, um briefing ou a transcrição de uma call (nesses casos a estimativa sai como preliminar); ou quando perguntarem "cabe em X semanas?" ou "isso é viável?" — inclusive quando parecer só uma estimativa rápida, porque é aí que o número sai errado.
---

# Skill de Dimensionamento — NTec

Transforma a **PRD** do projeto (o documento de requisitos que vira anexo de contrato) numa **faixa de semanas** defensável, calibrada nos projetos reais do NTec. O público principal são **CNs sem formação técnica**: a skill pergunta em linguagem de negócio, traduz para fatores técnicos e devolve números que a área de projetos reconhece.

**Onde fica no processo:** reuniões com o cliente → skill `documento-requisitos-ntec` gera a PRD → **esta skill dimensiona a partir da PRD** → área de projetos revisa → proposta. Sem PRD a skill ainda dimensiona, mas o resultado é preliminar.

A área de projetos sempre revisa o resultado antes de virar proposta. A skill evita que chegue à equipe um prazo "nada a ver". Ela não substitui a validação.

## Arquivos desta pasta (ler sob demanda)

| Arquivo | Quando ler |
|---|---|
| `leitura-prd.md` | **Sempre primeiro quando houver PRD.** Onde está cada fator na PRD, como traduzir os fatores de complexidade dela e como tirar do Fathom o que a PRD não responde (o resto vira lacuna para o CN) |
| `fathom.md` | **Sempre, antes de perguntar qualquer coisa ao CN.** Como achar sozinha as reuniões do cliente no Fathom (conector), conferir que são dele, quais transcrições ler e como registrar |
| `insumos-e-perguntas.md` | Quando **não** houver PRD (formulário, ficha, briefing, transcrição), e para o vocabulário de negócio → técnico |
| `regua-dimensionamento.md` | Sempre: pesos de cada fase (F1 a F6), faixa final, quando parar |
| `regra-fronteira-e-faixa.md` | Quando um fator encaixar em duas linhas da régua, e para a tabela de cortes de "O que cortar" |
| `exemplos/*.md` | Para achar um **precedente** parecido (mesmo formato, mesmo porte) e comparar o resultado |
| `scripts/calcular_faixa.py` | Sempre, para fazer a conta (não somar de cabeça) |
| `proposta-tecnica.md` | Sempre, no fim: como preencher cada uma das 14 seções da **Proposta Técnica** (o entregável) a partir da PRD e da conta |
| `scripts/gerar_proposta.py` | Sempre, no fim: gera o PDF "[Proposta Técnica] {projeto}" no template do NTec |
| `exemplos/ficticio-petagenda-proposta.json` | Modelo do JSON da proposta, para copiar a estrutura |

## Procedimento

1. **Ler a entrada.**
   - **Com PRD** (PDF ou markdown "[Requisitos] {projeto}", com ou sem o bloco "Inventário para dimensionamento"): seguir `leitura-prd.md`. Se o inventário não veio, montar a partir do PDF.
   - **Sem PRD** (formulário Dimensionamento Tec, ficha, briefing ou transcrição): seguir `insumos-e-perguntas.md` e marcar a resposta como ⚠️ *Estimativa preliminar, sem PRD*.
2. **Preencher os fatores** da régua. Com PRD, o que ela não responde (disponibilidade do cliente, stack e infra, benchmark, data de início) é **procurado nas reuniões do cliente no Fathom, que a skill busca sozinha pelo conector** (`fathom.md`; o que conta como resposta está em `leitura-prd.md`, Passo 4). Sem PRD, a busca no Fathom também vem antes das perguntas. O que não estiver lá vira **lacuna**: entra na conta com o valor mais pesado, vai para `lacunas_fathom` no JSON (o script mostra quantas semanas da venda dependem dela) e volta para o CN em *Lacunas para o CN* (no chat, na seção 11 e no anexo da proposta), com a pergunta pronta. **Integrações não geram pergunta:** recebem peso fixo pela regra de `regua-dimensionamento.md` (serviço de mercado = com documentação; sistema do cliente = sem documentação, com cláusula de contrato e plano B). No máximo 5 perguntas por rodada, as que mais mexem no prazo primeiro. Se o CN não souber, usar o valor mais pesado plausível e registrar em *Suposições*.
3. **Montar o JSON** de entrada (formato dos `exemplos/*.json`) e rodar:
   ```bash
   python scripts/calcular_faixa.py entrada.json
   ```
   Sem Python disponível, fazer a conta pela tabela de `regua-dimensionamento.md` e dizer isso em *Observações*.
4. **Comparar com o precedente** mais parecido em `exemplos/`. Se o resultado ficar muito longe do precedente (mais de 30%), explicar o porquê ou rever os fatores.
5. **Aplicar fronteira** se necessário (`regra-fronteira-e-faixa.md`).
6. **Montar a Proposta Técnica** (`proposta-tecnica.md`): um JSON com as 14 seções do template e o bloco `dimensionamento` (a mesma entrada do passo 3, mais critérios, lacunas, precedente, cortes e observações). Gerar o PDF:
   ```bash
   python scripts/gerar_proposta.py proposta.json --saida <pasta-que-o-CN-vê>
   ```
   O script recalcula a faixa a partir de `dimensionamento.entrada` (nunca digitar as semanas no JSON) e confere o checklist final do template. Sem `reportlab`: `pip install reportlab`. Se não der para instalar, entregar o JSON e dizer isso.
7. **Responder no chat** no formato curto abaixo: o detalhe está no PDF.

## Regras gerais

- **Nunca inventar número nem fator.** Fator desconhecido → buscar no Fathom (`fathom.md`). Não está lá → lacuna para o CN, com o valor mais pesado na conta + *Suposições*. No NTec, o erro histórico é vender para menos (Projeto C: vendido 11, levou ~14).
- **Sempre marcar a origem** do dado: `(origem: PRD 2.3)`, `(origem: Fathom · 12/03 — "trecho")`, `(origem: lacuna — não está no Fathom)`, `(origem: CN)`, `(origem: formulário)`, `(origem: transcrição da call)`, `(origem: palpite do CN)`, `(origem: suposição da skill)`.
- **Restrição técnica → parar, não aumentar prazo.** Multitenancy com instâncias isoladas, integração bidirecional que a PRD diz não ter API e feature sem benchmark nem documentação viram 🚩, não "+3 semanas".
- **Sempre trazer a seção "O que cortar".** Com PRD, cada corte é um módulo ou uma integração da seção 2, e nunca algo que já está nas Exclusões Explícitas.
- **A PRD é anexo de contrato.** Não acrescentar módulo, perfil ou integração que não estão nela. Se a skill achar que falta algo para o produto funcionar, isso vai para *Pontos de atenção* ("a PRD não cobre X"), e não para a conta.
- **Não prometer stack nem tecnologia.** A seção 12 da proposta sugere a stack do template NTec sempre como *premissa a validar na Sprint de Refino*. Se o CN já falou "vai ser em X", registrar também como premissa a validar.
- **Semanas off e garantia ficam fora da faixa**: somar as semanas off do calendário depois; a garantia de 30 dias é gratuita.

## Parar e sinalizar para a área de projetos

- Qualquer 🚩 de `regua-dimensionamento.md` (multitenancy por instâncias, integração sem API, feature atípica sem benchmark nem documentação).
- **Provável acima de 26 semanas**: a régua ainda não foi calibrada para projetos grandes (Projetos E e F, de 34 e 38 semanas, não têm ficha técnica). Dimensionar mesmo assim, marcar ⚠️ e sugerir quebrar em fases.
- Dependência de dados de bases soltas → chamar o núcleo de Dados.
- Cliente Enterprise ou Grande com restrição de stack ou regras de deploy próprias.
- Divergência entre o que o CN declarou e o que aparece na PRD, nos documentos ou na transcrição.
- Lacuna dentro da PRD (módulo sem superfície, integração sem meio, termo como "tempo real" sem mecanismo): a PRD precisa de revisão antes de virar anexo de contrato.
- Teto de fase atingido.

## Entregável: a Proposta Técnica (template do NTec)

O dimensionamento final sai **no template "Proposta Técnica" do NTec** (Comercial → Validação de Projeto), o documento que o comercial envia para a validação técnica. São 14 seções, seis delas OBRIGATÓRIAS (4 Perfis, 6 Regras, 8 Segurança, 10 Roadmap, 11 Pontos em aberto, 14 Protótipo), o checklist final e, no fim, o **Anexo — Memória do dimensionamento**, com o que a área de projetos precisa revisar: faixa, F1 a F6 com critério, alertas, lacunas, precedente, O que cortar, suposições, ajustes de fronteira e pontos para a imersão.

Onde o dimensionamento aparece no corpo do template:
- **Caixa de status** (topo da página 1): status, prazo recomendado e a faixa.
- **10. Roadmap:** tudo que está na conta é *Prioritário*. *A confirmar* e *Futuro* nunca entram no cronograma (o script confere).
- **11. Pontos em aberto:** as lacunas do Fathom, as lacunas da PRD e toda dependência externa. Nunca vazio.
- **13. Cronograma sprint a sprint:** as fases F1 a F6 (valores centrais) distribuídas em sprints de 2 semanas, somando a recomendação de venda. Sprint que depende de um ponto da seção 11 vem marcada.

Como preencher cada seção: `proposta-tecnica.md`. O PDF é lido pelo comercial e pela área de projetos: frases curtas e todo termo técnico explicado na primeira vez.

**O que a skill não inventa no template:** link e conteúdo do protótipo (sem protótipo, a seção 14 sai com o aviso "Protótipo não recebido" e o checklist fica PENDENTE), responsável comercial e data de envio (em branco, para o CN), e histórico, logs ou permissões que a PRD não traz (escrever "não previsto no PRD").

## Resposta no chat (sempre esta, curta)

Público: um CN que talvez nunca tenha programado.

1. `# Projeto [nome do cliente ou produto]`
2. **Status**, uma linha: ✅ Pronto para proposta · ⚠️ Precisa de revisão da área de projetos (inclui "⚠️ Estimativa preliminar, sem PRD") · 🚩 Parado.
3. **Prazo em uma frase.** Ex.: *"Recomendado vender 18 semanas (9 sprints), mais as semanas off. Não vender abaixo de 13."* Com lacuna do Fathom que mexe na venda, usar a frase do script, com "+": *"Recomendado vender 18 semanas (9 sprints) = 16 + 2 que dependem do Fathom (disponibilidade do cliente, referência de produto), mais as semanas off. Não vender abaixo de 13."*
4. **Faixa:** otimista · provável · conservador, em semanas e sprints.
5. **Caminho do PDF** "[Proposta Técnica] {projeto}.pdf".
6. **Fathom**, uma linha: as reuniões lidas, por data e etapa, com link (`fathom.md`, Passo 4), ou o motivo de não ter lido nenhuma.
7. **Lacunas para o CN**, na tabela `Fator | Procurado no Fathom e não achado | Pergunta para o cliente | Valor provisório na conta`, ou `Sem lacunas: tudo veio da PRD e do Fathom.`
8. **Itens PENDENTES do checklist**, com o que o CN precisa fazer (ex.: anexar o protótipo e preencher a seção 14).
9. **Os 2 ou 3 pontos de atenção mais graves**, uma linha cada.

## Status da skill

Versão inicial (16/10/2026). Entrada padrão passou a ser a PRD em 16/10/2026. Integrações com peso fixo, sem perguntar ao CN, desde 17/10/2026. Sem buffer percentual nem negociação de prazo desde 17/10/2026. Fator Claude Code (núcleo do F3 e F2 mais rápidos; dupla com ~20 h/semana) desde 17/10/2026. Entregável no template “Proposta Técnica” do NTec, em PDF gerado por `scripts/gerar_proposta.py`, desde 17/10/2026. O que a PRD não responde vem da transcrição do Fathom, e o que faltar volta ao CN como lacuna, desde 17/10/2026. A venda mostra com "+" quantas semanas dependem das lacunas do Fathom (vende com elas, em sprints inteiras) desde 17/10/2026. As reuniões do cliente são buscadas sozinhas no Fathom pelo conector (`fathom.md`), sem o CN colar transcrição, desde 17/10/2026. Calibrada com Projeto A, Projeto D, Projeto C e Projeto B, que são anteriores ao formato de PRD: a contagem "1 módulo = 1 fluxo" ainda não foi conferida contra um projeto real com PRD. Pendente: (1) fichas técnicas de Projetos E e F para calibrar projetos grandes; (2) conteúdo da subpágina "Régua de Dimensionamento" do Notion, que não veio no PDF; (3) atualizar a calibração quando Projeto B e Projeto D terminarem.
