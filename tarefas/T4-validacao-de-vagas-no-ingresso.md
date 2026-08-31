# T4 — Validação de vagas no ingresso do morador

| Campo | Valor |
|---|---|
| Identificador | T4 |
| Tipo de tarefa | Validação de entrada |
| Regras de negócio | BR07 (Vagas), BR02 (Morador) |
| Casos de uso | UC5 e UC17 — Convidar morador e aceitar solicitação |

---

## Estado inicial no repositório-base

Já existe:

- os campos `total_vagas` e `vagas_ocupadas` na república, e o método de domínio
  que calcula as vagas disponíveis pela diferença entre eles;
- o *endpoint* `POST /api/solicitacoes-moradia/{id}/aceite`, que efetiva o
  ingresso do morador na república;
- o *endpoint* `DELETE /api/moradores/{id}/republica`, que efetua o
  desligamento;
- a restrição de que um morador não reside em mais de uma república ao mesmo
  tempo, já aplicada no domínio.

O ingresso **não** verifica se há vaga disponível e **não** atualiza o número de
vagas ocupadas. O desligamento também não o atualiza. Em consequência, o
quantitativo de vagas ocupadas permanece estático e pode divergir do número real
de moradores.

---

## Requisito canônico

> Condicionar o ingresso de um morador em uma república à existência de vaga
> disponível.
>
> O número de vagas disponíveis de uma república corresponde à diferença entre o
> total de vagas e o número de vagas ocupadas. Um morador só pode ingressar
> quando houver ao menos uma vaga disponível.
>
> Ao ingressar um morador, o número de vagas ocupadas é acrescido de um. Ao
> desligar um morador, é reduzido de um. O número de vagas ocupadas nunca pode
> ser negativo nem superior ao total de vagas.
>
> A tentativa de ingresso em república sem vaga disponível deve ser recusada,
> sem qualquer alteração de estado.

---

## Critérios de aceitação

1. O aceite de uma solicitação em república com vagas disponíveis efetiva o
   ingresso e incrementa em um o número de vagas ocupadas.
2. O aceite de uma solicitação em república sem vagas disponíveis é recusado com
   código `409`.
3. Após uma recusa por ausência de vaga, o morador permanece sem república, a
   solicitação permanece pendente e o número de vagas ocupadas permanece
   inalterado.
4. O desligamento de um morador decrementa em um o número de vagas ocupadas.
5. O número de vagas ocupadas nunca se torna negativo, mesmo diante de
   desligamentos sucessivos.
6. O número de vagas ocupadas nunca excede o total de vagas.
7. Após uma sequência de ingressos e desligamentos, o número de vagas ocupadas
   corresponde ao número de moradores efetivamente vinculados à república.
8. A resposta de recusa traz mensagem que identifica a ausência de vaga como
   motivo, sem expor detalhes internos de persistência.
9. A restrição de residência única permanece válida: um morador que já reside em
   uma república continua impedido de ingressar em outra.
10. As regras arquiteturais do projeto permanecem satisfeitas.

---

## Observações para a suíte-oráculo

A república de identificador `3` da massa de dados inicial encontra-se com todas
as vagas ocupadas, e serve diretamente ao caso de recusa.

Merecem verificação explícita:

- a ausência de efeito colateral na recusa, verificando as três consequências do
  critério 3 e não apenas o código de resposta, pois uma implementação que
  recusa depois de já ter alterado o estado passaria em uma verificação
  superficial;
- a coerência entre o número de vagas ocupadas e a contagem real de moradores
  após uma sequência de operações, que detecta implementações que validam a
  entrada mas esquecem de atualizar o quantitativo;
- o desligamento seguido de novo ingresso, que verifica o decremento e o
  incremento na mesma sequência;
- o caso de concorrência não é objeto desta tarefa e não deve ser verificado.
