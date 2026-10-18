# Skill de Dimensionamento — NTec

Skill do Claude que ajuda coordenadores de negócios (CNs) do Núcleo de Tecnologia a chegar a uma **faixa de semanas** realista (otimista / provável / conservador) antes de mandar a proposta para a área de projetos. A arquitetura segue a skill de dimensionamento do NCiv (Gabrielle Lobo, [repositório](https://github.com/gabriellelobo-pj/skill-dimensionamento)). A régua segue o formulário *Dimensionamento Tec* e foi calibrada nos projetos reais do NTec.

**Entrada padrão: a PRD.** A skill roda logo depois da `documento-requisitos-ntec` ([repositório do Augustus](https://github.com/rivasaugustus/documento-requisitos-ntec)), que gera o documento de requisitos "[Requisitos] {projeto}". Esta skill ocupa o lugar que a skill da PRD chama de `validacao-tecnica-ntec`, mas calcula só o prazo, sem preço. Sem PRD, ela dimensiona mesmo assim, e o resultado sai como ⚠️ estimativa preliminar.

> **Retomando em outra conversa?** Leia a seção [Contexto para retomar](#contexto-para-retomar) no fim deste arquivo.

## Estrutura

```
skill-dimensionamento-ntec/
├── README.md                       ← este arquivo
├── docs/
│   ├── Dimensionamento-NTec-Skill.pdf   ← PDF explicativo (v4, 11 páginas: Proposta Técnica e fator Claude Code)
│   └── gerar_pdf.py                     ← gera o PDF (reportlab + fontes Segoe UI do Windows)
└── .claude/skills/dimensionamento-ntec/
    ├── SKILL.md                    ← orquestrador: procedimento, guardrails, entregável e resposta no chat
    ├── proposta-tecnica.md         ← como preencher as 14 seções da Proposta Técnica (o entregável)
    ├── leitura-prd.md              ← entrada padrão: onde está cada fator na PRD + o que perguntar ao CN
    ├── fathom.md                   ← busca automática das reuniões do cliente no Fathom (conector)
    ├── insumos-e-perguntas.md      ← caminho sem PRD (formulário, ficha, transcrição) + vocabulário
    ├── regua-dimensionamento.md    ← pesos por fase (F1 a F6), faixa final, calibração, fontes
    ├── regra-fronteira-e-faixa.md  ← desempate entre linhas + tabela de cortes ("O que cortar")
    ├── scripts/calcular_faixa.py   ← faz a conta (fonte da verdade dos pesos)
    ├── scripts/gerar_proposta.py   ← gera o PDF "[Proposta Técnica] {projeto}" no template do NTec (reportlab)
    └── exemplos/                   ← precedentes (.md + .json): Projeto A, Projeto B, Projeto C, Projeto D,
                                      Projetos E e F (pendente), PetAgenda (fictício, resposta completa)
                                      PetAgenda via PRD (fictício, leitura da PRD) e PetAgenda proposta (JSON modelo)
```

**Projetos anonimizados.** Os precedentes reais do NTec aparecem como Projeto A a F, só com o tipo de produto e os fatores da régua. Quem é cada um, as fichas originais e os links internos ficam numa pasta `privado/` que não vai para o repositório.

## Como usar

- **Instalada em:** `~\.claude\skills\dimensionamento-ntec\` (pasta pessoal; vale em qualquer projeto do Claude Code).
- **Chamar:** `/dimensionamento-ntec`, ou pedir "dimensiona esse projeto" mandando o PDF da PRD (e, se tiver, o bloco "Inventário para dimensionamento"). Sem PRD, aceita o formulário, a ficha ou a transcrição, mas a estimativa sai como preliminar.
- **Fathom:** a skill busca sozinha as reuniões do cliente (`fathom.md`). No claude.ai, ligar o conector Fathom em *Configurações → Conectores*. No Claude Code, uma vez: `claude mcp add --transport http --scope user fathom https://api.fathom.ai/mcp` e `/mcp` para o login. Sem conector, a skill pede o link ou a transcrição ao CN.
- **claude.ai:** compactar a pasta `dimensionamento-ntec` em .zip e enviar em *Settings → Capabilities → Skills*.
- **Rodar a conta direto:**
  ```bash
  python .claude/skills/dimensionamento-ntec/scripts/calcular_faixa.py .claude/skills/dimensionamento-ntec/exemplos/projeto-a.json
  ```
- **Gerar a Proposta Técnica** (precisa de `pip install reportlab`):
  ```bash
  python .claude/skills/dimensionamento-ntec/scripts/gerar_proposta.py .claude/skills/dimensionamento-ntec/exemplos/ficticio-petagenda-proposta.json --saida .
  ```
- **Regerar o PDF explicativo:**
  ```bash
  python docs/gerar_pdf.py docs/Dimensionamento-NTec-Skill.pdf
  ```

**Instalar a partir do GitHub:** clonar o repositório e copiar a pasta `.claude/skills/dimensionamento-ntec` para `~/.claude/skills/`.

**Atenção:** esta pasta é a cópia de trabalho. A cópia instalada em `~/.claude/skills` **não se atualiza sozinha**: depois de editar aqui, copiar de novo para lá.

---

## Contexto para retomar

Resumo das conversas que criaram e evoluíram a skill (16 e 17/10/2026), para continuar em outra conversa sem precisar do histórico. **Comece por "Onde paramos".**

### Onde paramos (17/10/2026)
- **Implementado:** o dimensionamento final sai no **template "Proposta Técnica" do NTec** (14 seções + checklist + anexo com a memória do dimensionamento), gerado por `scripts/gerar_proposta.py`. O bloco para colar no formulário Dimensionamento Tec saiu. Guia em `proposta-tecnica.md`. O PDF explicativo em `docs/` (v4) já mostra a Proposta Técnica e o fator Claude Code.
- **Implementado:** **fator Claude Code** na régua (todos os projetos do NTec agora são codados com Claude Code por uma dupla com ~20 h/semana somadas): núcleo do F3 x0,6 / 0,75 / 0,9 e F2 x0,7 / 0,85 / 1; imersão, extras do F3, testes, deploy e lojas ficam iguais. É um meio-termo: um x0,5 acertava o Projeto B (vender 12) mas dava 20 semanas para um app + painel de 8 fluxos com offline que a área de projetos espera em ~12 sprints. Projetos antigos rodam com `claude_code: false`.
- **Implementado:** integrações com **peso fixo, sem perguntar ao CN** (ver [Integrações: peso fixo](#integrações-peso-fixo-sem-perguntar-ao-cn-implementado)).
- **Implementado:** o que a PRD não responde é tirado da **transcrição do Fathom**; o que não estiver lá volta ao CN como **lacuna** (`leitura-prd.md`, Passo 4).
- **Implementado:** a venda mostra com **"+"** quanto depende das lacunas do Fathom: *"vender 14 semanas = 12 + 2 que dependem do Fathom (disponibilidade do cliente, referência de produto)"*. Vende com as lacunas no valor mais pesado (o erro do NTec é vender para menos) e mostra a parte delas em sprints inteiras. Vale também quando há Fathom, mas ele não fala do assunto ("dependem de confirmar com o cliente"). No JSON: `lacunas_fathom` e `fathom`. A página "Régua de Dimensionamento NTec" faz igual.
- **Removido (17/10/2026):** o buffer de risco percentual (desprezível; provável = soma dos centrais, conservador = soma dos máximos) e a negociação "quando o cliente pede menos prazo". A seção "O que cortar" continua. Só o Projeto B mudou na calibração (11 / 14 / 21).
- **Implementado (17/10/2026):** busca automática das reuniões do cliente no Fathom pelo conector (`fathom.md`). A skill acha as reuniões pelo radical do nome da empresa (gravações de qualquer CN), confere que são do cliente, lê os resumos de todas e a transcrição de até 3, e lista no chat as reuniões lidas. O CN só cola transcrição ou link se a busca falhar.
- **Próximo passo:** pendência 2 (avisar o Augustus) ou 3 (resto do levantamento), conforme a lista de pendências.
- **Como trabalhar com o usuário:** quando faltar informação ou houver decisão a tomar, **perguntar antes de assumir**, com uma opção recomendada.

### Linha do tempo
| Data | O que foi feito |
|---|---|
| 16/10/2026 | Criação da skill (v1) a partir da skill do NCiv, do formulário Dimensionamento Tec e do levantamento do Notion; calibração com 4 projetos |
| 16/10/2026 | A PRD da skill `documento-requisitos-ntec` virou a entrada padrão (`leitura-prd.md`) |
| 16/10/2026 | Projetos anonimizados (Projeto A a F) e repositório publicado (público) no GitHub |
| 16/10/2026 | PDF explicativo v2 (9 páginas), já com a PRD |
| 17/10/2026 | Explicação de onde vem o "Inventário para dimensionamento"; discussão sobre integrações sem documentação |
| 17/10/2026 | Integrações com peso fixo, sem perguntar ao CN; PDF v3 (10 páginas) |
| 17/10/2026 | Buffer de risco e negociação de prazo removidos |
| 17/10/2026 | O que a PRD não responde vem do Fathom; o que faltar volta ao CN como lacuna |
| 17/10/2026 | Entregável passa a ser a Proposta Técnica no template do NTec (PDF) |
| 17/10/2026 | Fator Claude Code (primeiro x0,5, depois o meio-termo x0,75 no núcleo do F3); PDF explicativo v4 |
| 17/10/2026 | Venda com "+" para as lacunas do Fathom ("14 = 12 + 2 que dependem do Fathom"), igual na skill e na página |
| 17/10/2026 | Busca automática das reuniões do cliente no Fathom pelo conector (`fathom.md`), sem o CN colar transcrição |

### O que foi pedido (origem)
Criar, para o NTec, uma skill parecida com a do NCiv da Gabi Lobo, que leve em conta que somos uma **empresa júnior**, use os dados reais dos projetos e chegue a uma **faixa de semanas**. O uso deve ser **prático para CNs**, que em geral são fracos tecnicamente e mandam para a equipe de projetos prazos sem sentido.

### Insumos usados
1. **Skill do NCiv** (GitHub da Gabi): 4 arquivos Markdown. SKILL.md orquestra, os outros 3 trazem as regras (faixas por disciplina, regra de fronteira, leitura do Drive).
2. **Formulário "Dimensionamento Tec"** (`~\Downloads\dimensionamento.pdf`, 5 páginas): seções 1–10. O Baseline (seção 7) tem as fases de 1 a 4 semanas cada e lojas de 2 a 6. O perfil da demanda (seção 6) traz formato, níveis de acesso, multitenancy, integrações, dados e offline. A seção 8 é o buffer adaptável (a régua não usa mais). A subpágina **"Régua de Dimensionamento"** do Notion é citada, mas **não veio no PDF**.
3. **Levantamento do Notion** (colado no chat, **cortado no meio do Projeto D**). Trouxe o inventário de 6 projetos e as fichas completas de Projeto C, Projeto A e Projeto B.
4. **Skill da PRD** (`documento-requisitos-ntec`, [repositório do Augustus](https://github.com/rivasaugustus/documento-requisitos-ntec)): gera o documento de requisitos "[Requisitos] {projeto}" a partir das reuniões (Fathom, Granola, transcrições). Template: Visão Geral, Requisitos Funcionais por módulo (A(cli)D/SID, Superfície, integrações com sentido, dados e meio), Não Funcionais e Exclusões Explícitas.

### Decisões tomadas
**Régua (v1)**
- **Modelo:** fases F1–F6 do formulário, cada fator com mínimo / central / máximo. Faixa: otimista = soma dos mínimos; provável = soma dos centrais; conservador = soma dos máximos. A venda recomendada é o provável arredondado para sprints inteiras (2 semanas), mais as semanas off.
- **Os pesos já embutem o time do NTec:** uma dupla com ~20 h/semana somadas, codando com Claude Code, e o template NTec (fator Claude Code no núcleo do F3 e no F2; ver `regua-dimensionamento.md`). **Sem buffer percentual** (removido em 17/10/2026 por ser desprezível): o risco fica no intervalo mín/central/máx.
- **Fronteira: sempre para o lado mais pesado.** O histórico do NTec é vender para menos (Projeto C).
- **Atípico ≠ regra de negócio específica**, para não contar o mesmo motivo duas vezes (erro achado na calibração do Projeto C).
- **A resposta sempre traz a seção "O que cortar"** (cortes de escopo com a economia em semanas). A negociação "quando o cliente pede menos prazo" foi removida em 17/10/2026.
- **Template NTec** (Google login, e-mail, S3) não soma prazo. Painel admin no mesmo código web não conta como formato extra.
- **Script Python como fonte da verdade** dos pesos, para o Claude não somar de cabeça. Se mudar um peso, mudar no script **e** na régua.

**PRD como entrada padrão**
- A skill lê o PDF da PRD e, se o CN colar, o bloco "Inventário para dimensionamento". Sem o bloco, monta o inventário a partir do PDF.
- **Sem PRD:** dimensiona, mas marca ⚠️ "Estimativa preliminar, sem PRD".
- **O que a PRD não cobre** (disponibilidade do cliente, stack e infra, benchmark, data de início): a skill procura na **transcrição do Fathom** (só vale o que foi dito explicitamente, citando reunião e trecho). O que não achar vira **lacuna**: entra na conta com o valor mais pesado e volta ao CN na seção 5 da resposta, com a pergunta pronta. Lacuna que mexe na faixa deixa o status em ⚠️. A documentação das APIs **saiu** dessa rodada (peso fixo, seção abaixo).
- **1 módulo = 1 fluxo.** 2.1 Autenticação vai para F2, e Disponibilização vai para F5 e F6.
- **Fatores de complexidade da PRD encaixam nas linhas atuais**, sem peso novo. Pagamento, IA e push viram integração. Concorrência, imutabilidade e rotina agendada fazem o módulo virar RN específica. Tempo real por push e relatório configurável viram feature atípica. Tabela completa em `leitura-prd.md`.
- **Papel da skill:** ocupa o lugar que a skill da PRD chama de `validacao-tecnica-ntec`, mas calcula **só o prazo, sem preço**.

**Publicação**
- Repositório **público**: https://github.com/matheusrpgomes-sketch/skill-dimensionamento-ntec
- **Projetos reais anonimizados** como Projeto A a F. Nome real, fichas originais e links do Notion ficam em `privado/` (no `.gitignore`, só no computador local; o mapa está em `privado/mapa-projetos.md`). **Nunca** subir nome de cliente, link do Notion ou detalhe que identifique o projeto.

### De onde vem o "Inventário para dimensionamento"
É gerado pela skill da PRD, mas **não fica no PDF**: aparece só na resposta dela no chat (Passo 7), junto com Perfis, Funcionalidades, Integrações e Pontos para a imersão. Colunas fixas: `Módulo | Superfície | Itens A(cli)D | Itens SID | Integrações | Fluxos de dados | Fatores de complexidade`. O CN copia esse bloco e manda junto com o PDF; se esquecer, a nossa skill reconstrói o inventário a partir do PDF (só os "Pontos para a imersão" se perdem). Ideia para pedir ao Augustus: a skill da PRD salvar o inventário num arquivo ao lado do PDF.

### Integrações: peso fixo, sem perguntar ao CN (implementado)
**Problema:** o documento de validação pedia ao CN a API de cada integração (documentação, tipo, fluxo). Os CNs atuais não têm como responder, e não há técnico para validar antes da proposta.

**Decisão (17/10/2026):** a skill não pergunta sobre a documentação e não pesquisa o fornecedor. Cada integração recebe um peso fixo a partir do que a PRD já traz: o **sentido** (uni ou bi) e **de quem é o sistema**.
- **Serviço de mercado** (pagamento, Pix, mapa, push, IA, WhatsApp Business, NF-e, Google Calendar…) → linha "com documentação" (`uni_doc` / `bi_doc`).
- **Sistema do cliente** (ERP, legado, sob medida), meio `arquivo` ou dúvida → linha "sem documentação" (`uni_sem_doc` / `bi_sem_doc`), mais **cláusula de contrato** ("depende de o cliente fornecer acesso à API até a sprint {n}; sem acesso, vira planilha, sem alteração de prazo") e o **plano B** (planilha) em "O que cortar".
- `sem_api` (🚩) só quando a PRD disser explicitamente que o sistema não tem API.

**Pesos inalterados:** a regra só fixa qual linha usar. Os exemplos rodados depois da mudança dão os mesmos números de antes. **Descartado da proposta anterior:** catálogo em arquivo separado, pesquisa na web, mensagem para o cliente repassar ao TI e a categoria "a confirmar" no script.

**Arquivos mexidos:** `regua-dimensionamento.md` (regra + registro das fontes), `leitura-prd.md`, `insumos-e-perguntas.md`, `SKILL.md`, `regra-fronteira-e-faixa.md`, mensagens de alerta do `scripts/calcular_faixa.py`, exemplos PetAgenda, `docs/gerar_pdf.py` e o PDF.

### Calibração atual
| Projeto | Faixa da régua (otim. / prov. / cons.) | Vendido | Situação |
|---|---|---|---|
| Projeto A | 9 / 12 / 16 | 12 | No prazo ✅ |
| Projeto D | 13 / 17 → vender 18 / 23 | 18 | Sem atraso ✅ (fatores com palpite) |
| Projeto C | 13 / 18 / 23 | 11 | Atrasou (~14). Régua ~3 semanas acima do real (viés conhecido) |
| Projeto B | 11 / 14 / 21 | 12 | Em execução; fim previsto em 23/11/2026 |
| Projetos E e F | não calibrado | 34 / 38 | Sem ficha. Alerta automático acima de 26 semanas |

Todos são **anteriores à PRD**: as regras de leitura da PRD ainda não foram conferidas contra um projeto real.

### Pendências (próximos passos)
1. **Conferir a regra de peso fixo** no primeiro projeto com integração com sistema do cliente que terminar (se `bi_sem_doc` ficou longe do real, ajustar o peso).
2. **Avisar o Augustus:** trocar `validacao-tecnica-ntec` por `dimensionamento-ntec` na skill da PRD (a nossa não calcula preço) e, se possível, salvar o inventário num arquivo.
3. **Mandar o resto do levantamento** (Projeto D completo, Projetos E e F) para calibrar os projetos grandes e tirar os palpites do Projeto D.
4. **Conseguir a subpágina "Régua de Dimensionamento"** do Notion. Se tiver números, eles valem mais que os da skill.
5. **Atualizar a calibração** quando o Projeto A (19/10/2026) e os Projetos B e D (23/11/2026) terminarem. Passo a passo em `exemplos/README.md`; atualizar também a ficha original em `privado/`.
6. **Conferir as regras de leitura da PRD** no primeiro projeto com PRD que terminar (principalmente "rotina agendada → RN específica").
7. Testar a skill com um CN num lead real, conferindo se a busca do Fathom acha as reuniões certas.
8. **Conferir o fator Claude Code** com o realizado do Projeto B (23/11/2026) e do primeiro projeto com login que terminar (o F2 com fator não foi medido).

Feito: PDF v2 com a PRD; publicação no GitHub com projetos anonimizados; integrações com peso fixo e PDF v3; Proposta Técnica como entregável, fator Claude Code e PDF v4.

### Onde está cada coisa
| O quê | Caminho |
|---|---|
| Cópia de trabalho (esta pasta, repositório git) | `~\OneDrive\Área de Trabalho\Projetos\skill-dimensionamento-ntec\` |
| Repositório no GitHub (público) | https://github.com/matheusrpgomes-sketch/skill-dimensionamento-ntec |
| Skill instalada (não sincroniza sozinha) | `~\.claude\skills\dimensionamento-ntec\` |
| Mapa dos projetos reais (fora do git) | `privado\mapa-projetos.md` e `privado\exemplos-originais\` |
| PDF | `docs\Dimensionamento-NTec-Skill.pdf` (cópia também em `~\Downloads\`) |
| Formulário original | `~\Downloads\dimensionamento.pdf` |
| Template da Proposta Técnica (original) | `~\Downloads	emplate-proposta-comercial.pdf` |
| Skill da PRD (Augustus) | https://github.com/rivasaugustus/documento-requisitos-ntec |

**Fluxo de atualização:** editar nesta pasta → rodar os exemplos com o script se mexeu em peso → copiar `.claude/skills/dimensionamento-ntec` para `~/.claude/skills/` → commit e push.
