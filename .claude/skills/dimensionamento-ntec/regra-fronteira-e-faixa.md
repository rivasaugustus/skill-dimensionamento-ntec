# Regra de fronteira e de negociação da faixa

Vale quando um fator não cai claramente numa linha da régua, ou quando o cliente pede um prazo que não cabe na faixa. Não substitui a régua: resolve só o empate e sempre deixa rastro para a revisão da área de projetos.

---

## 1. Direção padrão: sempre para o lado mais pesado

Diferente do NCiv (onde a direção muda por disciplina), no NTec a direção é **uma só: para cima.** O histórico mostra que o erro do núcleo é vender para menos:
- Projeto C: vendido 11, realizado ~14 e com testes por terminar.
- Projeto A: a proposta original tinha 14 sprints e virou um MVP de 6. Deu certo porque o **escopo** foi cortado, não o prazo.
- Projeto B: o prazo encolheu de 4 para 2–3 meses enquanto o escopo crescia. O hand-off registrou que "não se sustenta com só 1 dev".

## 2. Ordem de decisão

Sempre nesta ordem. Só passa ao passo seguinte se o anterior não resolver.

**Passo 0 — O que o cliente disse.** Procurar no mapa das falas (`leitura-prd.md`, Passo 4) a fala do cliente sobre o ponto em dúvida: "precisa estar na App Store" decide PWA × app; "é o nosso ERP" decide serviço de mercado × sistema do cliente; "o gestor só aprova" decide poucas × grandes diferenças entre perfis; uma referência citada decide atípico × setor incomum. Fala explícita do cliente decide, e o comentário cita a reunião e o minuto. Com PRD, a fala só desempata o que a PRD deixa em dúvida: não muda o escopo.

**Passo 1 — Precedente.** Procurar em `exemplos/` um projeto com o mesmo formato e um número parecido de fluxos e níveis de acesso. Se achar, usar a classificação que ele usou para o fator em dúvida e registrar qual foi.

**Passo 2 — Linha mais pesada.** Sem precedente, escolher a linha de cima da régua. Registrar.

**Passo 3 — Meio-termo de 0,5 semana.** É **alternativa** ao passo 2, nunca acréscimo. Usar quando as duas condições valem:
- subir de linha custa 1 semana ou mais no central, **e**
- o projeto encosta na linha de cima por um único critério e por pouco (ex.: 2 perfis com poucas diferenças, mas um deles tem uma única tela de aprovação).

**Passo 4 — Parar e marcar para revisão**, nos casos dos Guardrails (seção 5).

## 3. Tipos de fronteira conhecidos

| # | Situação | Exemplo | O que fazer |
|---|---|---|---|
| 1 | Fluxo comum ou RN específica? | Cadastro com validação de CPF e CNPJ | Comum. RN específica é regra **do cliente** que precisa de documento (fórmula, tabela, norma). Validação padrão de mercado é comum |
| 2 | Poucas ou grandes diferenças entre perfis? | Admin e usuário, em que o admin só vê um menu a mais | Poucas. Telas próprias, regras próprias ou aprovação de cadastro = grandes |
| 3 | PWA ou app? | "Quero que instale no celular" | Se não precisa de loja nem de recurso nativo pesado (Bluetooth, segundo plano) → PWA. Na dúvida → app |
| 4 | Serviço de mercado ou sistema do cliente? | "Tem API, o TI de lá passa" | Produto vendido para qualquer empresa, com API pública = mercado. Sistema que só o cliente usa, ou dúvida = sistema do cliente (peso fixo, sem pedir link) |
| 5 | Atípico ou só setor incomum? | App de agenda para criadores de abelhas | Setor incomum com produto comum = **não** atípico. Atípico é o *produto* sem benchmark |
| 6 | Mesmo formato ou formato extra? | Painel admin | Mesmo código web = mesmo formato. Código separado (ex.: app nativo + painel web) = extra |
| 7 | Dependência de dados ou integração? | "Importar a tabela nutricional pública" | Dado estático importado uma vez ou atualizado à mão = dependência de dados. Consulta automática a um serviço = integração |

## 4. Comentário obrigatório

Toda vez que esta regra mexer no dimensionamento, sai um comentário de **duas linhas** em *Observações*:

```
[Fase] [fator]: [linha de origem] → [linha final] ([+X] semana no central)
Motivo: [qual fronteira e por que a regra decidiu assim]
```

Exemplos:

```
F2 níveis de acesso: 2-6 poucas diferenças → 2-6 grandes diferenças (+1)
Motivo: são 3 perfis, mas o profissional tem cadastro com aprovação que bloqueia rotas; precedente Projeto C.

F3 formato: PWA → app nativo (+x0,1 no núcleo, +2,5 de lojas)
Motivo: cliente quer "baixar na loja"; na dúvida entre PWA e app, a régua vai para o mais pesado.

F3 integração ERP: parada, sem valor
Motivo: a PRD diz que o sistema do cliente não tem API e ele quer enviar e receber dados; potencial inviabilidade, a skill não decide.
```

Se a regra não mexeu em nada: `Sem ajustes de fronteira.`

## 5. Guardrails

- **Mover de linha nunca gera "inviável".** Inviabilidade só vem dos 🚩 da régua e sempre vai para a área de projetos.
- **Máximo de 3 ajustes de fronteira por projeto.** No quarto, parar e marcar o projeto inteiro para revisão (é sinal de que a demanda está mal levantada, não de que o projeto está na fronteira).
- **Não contar duas vezes.** O mesmo motivo não entra em dois fatores (ex.: fórmula clínica → RN específica, não também "atípico"). A RN crítica em F4 é permitida porque mede outra coisa: o tempo de teste com o cliente.

---

## 6. Cortes que mais economizam (para a seção "O que cortar")

Ordem de impacto típica:

| Corte | Economia típica (central) |
|---|---|
| Publicar como PWA em vez de app nas lojas | F6 inteira (2,5 a 3) + fator de formato |
| Tirar o segundo formato (só app ou só web no MVP) | ~10% do desenvolvimento + parte do deploy |
| Juntar perfis de usuário (ex.: um "profissional" no lugar de "nutricionista" e "psicólogo") | 0,5 a 1 em F2 |
| Mover fluxos não essenciais para uma fase 2 | 1,25 por fluxo comum, 3 por fluxo com RN específica |
| Trocar integração automática por importação manual (planilha) no MVP — é o plano B obrigatório de toda integração com sistema do cliente | 0,5 a 2,5 |

Sempre recalcular com o script depois do corte. Não subtrair de cabeça.

**Prioridade do cliente.** Antes de ordenar os cortes, conferir no mapa das falas o que o cliente chamou de essencial e o que ele disse que pode ficar para depois. Corte de algo essencial vai por último e leva o aviso "o cliente disse que é essencial (data, minuto)"; o que ele aceitou adiar vai primeiro.
