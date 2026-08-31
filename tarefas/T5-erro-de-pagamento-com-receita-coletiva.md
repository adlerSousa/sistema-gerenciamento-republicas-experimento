# T5 — Tratamento de erro no pagamento com receita coletiva

| Campo | Valor |
|---|---|
| Identificador | T5 |
| Tipo de tarefa | Tratamento de erro |
| Regras de negócio | BR05 (Receita coletiva), BR06 (Receitas e despesas), RF09 (Registro de operações) |
| Casos de uso | UC9 — Registrar pagamento de receita ou despesa |

---

## Estado inicial no repositório-base

Já existe:

- a entidade `ReceitaColetiva`, com o saldo acumulado por república, e as
  operações de crédito e débito;
- o *endpoint*
  `POST /api/lancamentos/{id}/pagamentos-receita-coletiva`, que debita o valor
  informado do saldo e quita as participações cobertas por ele;
- o registro de operações, pela porta `RegistradorOperacoes`, com os formatos
  CSV, JSON e XML.

O pagamento **não** verifica se o saldo é suficiente. Quando o valor informado
excede o saldo, a operação chega até a camada de persistência e falha em uma
restrição do banco de dados, resultando em resposta de código `500` que expõe a
mensagem de erro do banco e o comando SQL ao cliente. Nenhum registro de falha é
gravado no log de operações.

---

## Requisito canônico

> Rejeitar o pagamento de lançamentos de despesa com receita coletiva quando o
> saldo acumulado for insuficiente, registrando a falha no log de operações.
>
> O valor utilizado da receita coletiva não pode exceder o saldo acumulado da
> república. Quando exceder, a operação deve ser recusada, o saldo e as
> participações no rateio devem permanecer inalterados, e a falha deve ser
> registrada no log de operações conforme o formato definido pelas convenções
> técnicas do projeto.
>
> A resposta da recusa deve identificar a insuficiência de saldo como motivo, sem
> expor detalhes internos de persistência.

---

## Critérios de aceitação

1. O pagamento com valor menor ou igual ao saldo acumulado continua a ser
   efetivado, debitando o saldo e quitando as participações cobertas.
2. O pagamento com valor maior que o saldo acumulado é recusado com código `409`.
3. Após a recusa, o saldo da receita coletiva permanece exatamente igual ao
   anterior à tentativa.
4. Após a recusa, nenhuma participação no rateio é marcada como paga e a
   situação do lançamento permanece inalterada.
5. A resposta da recusa não contém mensagem de erro do banco de dados, comando
   SQL, nome de restrição, nome de tabela nem rastro de pilha.
6. A recusa gera um registro de falha no log de operações, no formato de
   mensagem de falha definido pelas convenções técnicas do projeto.
7. O pagamento bem-sucedido continua a gerar registro de sucesso no log.
8. O pagamento com valor exatamente igual ao saldo é efetivado, e não recusado.
9. A tentativa de pagar um lançamento de receita com receita coletiva permanece
   recusada, como antes da alteração.
10. A tentativa de pagamento de lançamento inexistente responde com código `404`.
11. As regras arquiteturais do projeto permanecem satisfeitas.

---

## Observações para a suíte-oráculo

O comportamento defeituoso do repositório-base é observável e serve de linha de
base: na tentativa de pagar valor superior ao saldo, a resposta atual é `500`
com o texto da restrição `ck_receita_coletiva_saldo_nao_negativo` e o comando
`update receita_coletiva ...`.

Merecem verificação explícita:

- a igualdade do saldo antes e depois da recusa, e não apenas o código de
  resposta, pois uma implementação que debita e depois lança exceção sem
  desfazer a transação passaria em uma verificação superficial;
- a ausência de participações quitadas após a recusa;
- o conteúdo da resposta, verificando que não contém os termos característicos
  do vazamento atual;
- a presença do registro de falha no arquivo de log, que é a parte do requisito
  mais facilmente omitida;
- o caso de fronteira em que o valor iguala o saldo, que distingue a comparação
  correta de uma comparação estrita equivocada.
