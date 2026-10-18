# Fathom — busca automática das reuniões do cliente

A skill puxa sozinha as reuniões do cliente no Fathom, **antes de perguntar qualquer coisa ao CN**. Com PRD, elas respondem o que a PRD não cobre (`leitura-prd.md`, Passo 4). Sem PRD, elas preenchem os fatores antes das perguntas (`insumos-e-perguntas.md`, Passo 2). O CN só cola transcrição ou link quando a busca não funciona.

O que conta como resposta continua em `leitura-prd.md`, Passo 4: só vale o que foi dito explicitamente, citando a reunião e o trecho. Este arquivo diz **como chegar às reuniões certas**.

---

## Passo 1 — Conector

| Onde a skill roda | Como chegar às ferramentas |
|---|---|
| claude.ai ou app do Claude | Conector **Fathom** ligado em *Configurações → Conectores*. As ferramentas costumam vir diferidas: carregar com `tool_search` ("fathom meetings") antes de chamar |
| Claude Code | Uma vez por computador: `claude mcp add --transport http --scope user fathom https://api.fathom.ai/mcp` e depois `/mcp` para fazer login. As ferramentas aparecem como `mcp__fathom__...` |

Ferramentas usadas: `scan_meeting_summaries`, `find_person`, `list_meetings`, `get_recording_by_url`, `get_meeting_summary` e `get_meeting_transcript`. **Só leitura:** a skill nunca altera nada no Fathom.

**Sem conector ou com erro de login:** dizer em uma linha ("Não consegui acessar o Fathom: [motivo]") e pedir **uma vez** ao CN o link da reunião ou a transcrição colada. Se ele não tiver, os fatores do Passo 4 de `leitura-prd.md` viram lacuna, com `"fathom": false` no JSON.

## Passo 2 — Achar as reuniões do cliente

Nesta ordem, parando no primeiro que achar:

1. **O que já está na conversa.** Link do Fathom colado pelo CN → `get_recording_by_url`. Se a PRD foi feita nesta mesma conversa, as reuniões que a skill da PRD já abriu valem: não buscar de novo. Se o CN colou os blocos de conferência da PRD, eles citam a data de cada reunião (`[Fathom · AT 12/03]`): usar essas datas para fechar a busca com `list_meetings` (`created_after` e `created_before` em volta da data).
2. **Pelo nome da empresa.** `scan_meeting_summaries` com o **radical** do nome e `recorded_by: "anyone"`, porque a gravação pode estar no nome de um colega. Radical é uma palavra só, sem "Ltda", "S.A.", "Engenharia", "Equipamentos" e afins ("SAUR Equipamentos" → `SAUR`), porque a busca exige todas as palavras. O padrão (10 páginas, ~250 reuniões) cobre uns 3 meses no ritmo atual do time. Ciclo de venda mais longo: subir `max_pages`.
3. **Pelo contato.** Nada pelo nome → `find_person` com o nome do contato do cliente e `recorded_by: "anyone"`.
4. **Nada achado:** dizer ao CN em uma linha e pedir o link da reunião (a gravação pode ser de alguém que não a compartilhou com ele). Sem link, seguir com lacuna e `"fathom": false`.

O nome da empresa vem da PRD (Visão Geral ou título), do chat, da ficha ou do formulário. Se a PRD só traz o nome do produto ("[Requisitos] PetAgenda") e nada diz quem é o cliente, perguntar ao CN em uma linha antes de buscar.

**Conferir cada reunião antes de usar.** É aqui que a busca erra:

- **O nome pode estar em qualquer posição do título** e com separadores diferentes: `Garoto & Poli Júnior USP`, `GreenYellow - Poli Júnior USP`, `MPD Engenharia + Poli Júnior`, `Poli Júnior USP & Sambaíba`. Comparar sem acento e sem caixa.
- **Só reunião com o cliente.** Reunião interna (aula, roleplay, reunião de núcleo, alinhamento só do time) não entra, mesmo que cite a empresa: só o cliente cria escopo. Convidado com e-mail do domínio da empresa confirma. A lista de convidados às vezes vem vazia ou só com nomes; aí decide o conteúdo do resumo.
- **Empresa homônima** (dois clientes com o mesmo radical, ou radical que é palavra comum): mostrar título e data das reuniões e perguntar ao CN qual é. Não escolher sozinho.
- **O título não diz a etapa.** `[Aprofundamento]` muitas vezes titula a AT Diagnóstica. A etapa sai do conteúdo do resumo.

## Passo 3 — O que ler

1. **`get_meeting_summary` de todas as reuniões do cliente** (é curto). O resumo do Fathom é de vendas (rapport, dores, próximos passos) e quase nunca traz, com as palavras do cliente, quem testa, servidor próprio ou data de início. Ele serve para **escolher as transcrições**, não como fonte.
2. **`get_meeting_transcript` em no máximo 3 reuniões** (transcrição é longa), passando também o `url` da reunião, para cada trecho vir com o link do minuto. Prioridade:
   - a mais recente, que costuma ter data de início e quem vai aprovar;
   - a AT Diagnóstica, onde aparecem sistema atual, infra e referência de produto;
   - a reunião cujo resumo mais toca nos fatores ainda abertos.

   Com até 3 reuniões, ler todas. Se só o resumo de uma quarta reunião toca num fator aberto, trocar por ela a transcrição menos útil.
3. **Procurar os fatores:** os do Passo 4 de `leitura-prd.md` (com PRD) ou os da tabela do Passo 2 de `insumos-e-perguntas.md` (sem PRD).

**Quem falou importa.** Vale o que o cliente disse, ou o que o time disse e o cliente confirmou na hora. "A gente costuma fazer em React", dito pelo CN, não é exigência de stack do cliente. A transcrição traz o nome de quem fala, mas não diz de que lado ele está: é do cliente quem tem e-mail do domínio da empresa nos convidados, ou quem só aparece nas reuniões deste cliente (quem aparece em reuniões de várias empresas é do time da Poli Júnior). Na dúvida, não conta como resposta.

**Divergência com a PRD.** Se a transcrição mostra algo que muda o tamanho e não está na PRD (ex.: o cliente pediu nota fiscal e a PRD não tem), isso **não entra na conta**, porque a PRD é o anexo de contrato. Vai para *Pontos de atenção*, com a reunião e o trecho.

## Passo 4 — Registrar

- **Origem** de cada fator encontrado: `(origem: Fathom · 12/03 — "a Ana valida toda sexta")`. No chat, o trecho leva o link do minuto que a transcrição devolve.
- **JSON da régua:** `"fathom": true` quando a busca achou e leu pelo menos uma reunião do cliente; `false` quando não havia conector ou nenhuma reunião foi achada. É o que faz o script escrever "dependem de confirmar com o cliente" (o Fathom foi lido e não fala do assunto) ou "dependem do Fathom" (não houve reunião para ler).
- **Resposta no chat:** uma linha *Fathom* com as reuniões lidas, por data e etapa, cada uma com link (ex.: `Fathom: 3 reuniões lidas · AT 02/10 · Aprofundamento 09/10 · Proposta 16/10`), para o CN conferir que a busca pegou o cliente certo. Sem reunião, a linha diz o motivo.
- **Proposta Técnica:** a mesma lista, sem links, como primeiro item de `suposicoes` no anexo.

Não colar transcrição nem resumo no chat ou no PDF: só o trecho curto que sustenta cada fator.
