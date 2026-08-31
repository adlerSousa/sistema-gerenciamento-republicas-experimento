# T2 — Reputação mensal do morador

| Campo | Valor |
|---|---|
| Identificador | T2 |
| Tipo de tarefa | Implementação de regra de negócio |
| Regras de negócio | BR02.2 e seus subitens BR02.2.1, BR02.2.2 e BR02.2.3 |
| Casos de uso | UC7, UC16 (exibição), UC5 (consulta pelo representante) |

---

## Estado inicial no repositório-base

Já existem, persistidos e consultáveis, todos os dados de que o cálculo depende:

- reclamações e sugestões, com autor, envolvidos, data de registro, data de
  solução e marcação de resolvida;
- tarefas domésticas, com responsáveis, data de término e data de conclusão;
- lançamentos com as participações no rateio, contendo a data de vencimento do
  lançamento e a data de pagamento de cada participação.

Não existe nenhum cálculo de reputação nem qualquer forma de consultá-la.

---

## Requisito canônico

> Implementar o cálculo da reputação mensal do morador e disponibilizá-lo para
> consulta, dado um morador e um mês de referência.
>
> A reputação mensal é a soma de três índices: o índice de solução de
> reclamações, o índice de realização de tarefas e o índice de compromisso com
> pagamentos.
>
> **Índice de solução de reclamações.** Sendo `NR` o número de reclamações
> registradas sobre o morador no mês e `NRR` o número dessas reclamações
> resolvidas no mês, o índice é `NRR / NR` quando `NR` for maior que zero, e
> vale um quando `NR` for igual a zero.
>
> **Índice de realização de tarefas.** Sendo `TAM` o número de tarefas
> atribuídas ao morador no mês e `TCM` o número dessas tarefas concluídas pelo
> morador dentro do prazo estipulado, o índice é `TCM / TAM` quando `TAM` for
> maior que zero, e vale um quando `TAM` for igual a zero.
>
> **Índice de compromisso com pagamentos.** É a razão entre o somatório dos dias
> agendados para pagamento e o somatório dos dias em que os pagamentos foram
> efetivamente realizados, considerando as participações do morador em
> lançamentos com vencimento no mês. Quando o pagamento ocorre em mês posterior
> ao do vencimento, somam-se aos dias os dias dos meses decorridos. Quando não
> houver participação com vencimento no mês, o índice vale um.

### Definições que integram o requisito

- **Reclamação registrada sobre o morador.** Reclamação, não excluída, com data
  de registro dentro do mês de referência, em que o morador consta entre os
  envolvidos.
- **Reclamação resolvida.** Reclamação marcada como resolvida, com data de
  solução dentro do mês de referência.
- **Tarefa atribuída no mês.** Tarefa em que o morador consta como responsável e
  cuja data de término esteja dentro do mês de referência.
- **Tarefa concluída no prazo.** Tarefa finalizada cuja data de conclusão não
  seja posterior à data de término.
- **Dia agendado para pagamento.** Dia do mês da data de vencimento do
  lançamento.
- **Dia do pagamento realizado.** Dia do mês da data de pagamento da
  participação, acrescido dos dias dos meses decorridos entre o mês do
  vencimento e o mês do pagamento. Participações ainda não pagas são
  consideradas pagas no último dia do mês corrente para efeito do somatório.
- **Precisão.** Os índices e a reputação são expressos com quatro casas
  decimais, arredondamento `HALF_UP`.

---

## Critérios de aceitação

1. É possível consultar a reputação de um morador em um mês de referência,
   obtendo os três índices e a soma.
2. Com `NR` igual a zero, o índice de solução de reclamações vale um.
3. Com `NR` maior que zero, o índice é a razão entre resolvidas e registradas,
   valendo zero quando nenhuma foi resolvida.
4. Com `TAM` igual a zero, o índice de realização de tarefas vale um.
5. Uma tarefa concluída após a data de término **não** conta como concluída no
   prazo, e portanto reduz o índice.
6. O índice de compromisso com pagamentos é maior que um quando os pagamentos
   ocorrem antes do dia do vencimento, e menor que um quando ocorrem depois.
7. Um pagamento realizado em mês posterior ao do vencimento produz índice menor
   do que o de um pagamento realizado no mesmo mês, em igualdade das demais
   condições.
8. A reputação é a soma exata dos três índices.
9. A consulta de um morador inexistente responde com código `404`.
10. Reclamações excluídas não são consideradas em nenhum dos índices.
11. O cálculo considera apenas os dados do mês de referência, sem contaminação
    de meses vizinhos.
12. As regras arquiteturais do projeto permanecem satisfeitas.

---

## Decisões de interpretação — pendentes de validação

A especificação de requisitos apresenta três pontos ambíguos, resolvidos acima
de forma explícita para tornar o resultado verificável. Devem ser confirmados
antes do congelamento.

**1. A terceira alternativa das fórmulas de `ISS` e `IRT`.**
A especificação apresenta três alternativas para cada índice: a razão, um valor
correspondente a "caso tenha resolvido reclamações de outros" (ou "concluído
tarefa para outros"), e o valor um. A segunda alternativa pressupõe distinguir
o morador que resolve uma reclamação de terceiro daquele sobre quem a reclamação
foi registrada, distinção que o modelo de dados da especificação não sustenta,
pois o registro guarda apenas o autor e os envolvidos.

*Resolução adotada:* a fórmula foi reduzida a duas alternativas, mantendo a
razão e o valor um. *Alternativa:* acrescentar ao modelo de dados a figura do
morador que soluciona, o que amplia o escopo da tarefa.

**2. Divisão por zero no índice de compromisso com pagamentos.**
A especificação não define o índice quando não há pagamentos no mês.

*Resolução adotada:* o índice vale um, por analogia com os outros dois índices.

**3. Participações vencidas e não pagas.**
A especificação não define como tratar a participação cujo vencimento ocorreu no
mês mas que permanece em aberto.

*Resolução adotada:* considera-se paga no último dia do mês corrente, o que
penaliza o atraso de forma crescente. *Alternativa:* excluí-la do somatório, o
que faria o atraso não produzir efeito algum sobre o índice.

---

## Observações para a suíte-oráculo

- Verificar cada índice isoladamente antes de verificar a soma, de modo que uma
  falha indique qual parcela do cálculo está incorreta.
- Incluir um caso em que os três índices resultam em valores distintos entre si,
  evitando que uma implementação que devolva o mesmo número três vezes seja
  aprovada.
- Incluir o caso de pagamento em mês posterior, que uma implementação incompleta
  tende a tratar como se fosse no mesmo mês.
- Verificar o mês de referência nas fronteiras, com registros no primeiro e no
  último dia do mês.
