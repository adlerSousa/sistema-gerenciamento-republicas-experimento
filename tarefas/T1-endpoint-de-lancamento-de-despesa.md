# T1 — Endpoint de lançamento de despesa

| Campo | Valor |
|---|---|
| Identificador | T1 |
| Tipo de tarefa | Criação de *endpoint* |
| Regras de negócio | BR06 (Receitas e despesas), RF09 (Registro de operações) |
| Casos de uso | UC4 — Registrar receitas e despesas |

---

## Estado inicial no repositório-base

Já existe:

- a entidade de domínio `Lancamento` e a entidade `ParticipacaoLancamento`, com
  as tabelas `lancamento` e `participacao_lancamento`;
- a porta `LancamentoRepositorio` e sua implementação sobre JPA;
- os *endpoints* de consulta `GET /api/lancamentos` e
  `GET /api/lancamentos/{id}`;
- o registro de operações, pela porta `RegistradorOperacoes`.

Não existe nenhum *endpoint* de criação de lançamentos.

---

## Requisito canônico

> Implementar o *endpoint* de registro de lançamentos de despesa de uma
> república.
>
> A requisição informa a república, a descrição, o valor total, a data de
> vencimento, a periodicidade e a relação de moradores participantes do rateio.
> A periodicidade é única, semanal ou mensal. Nas periodicidades semanal e
> mensal, a requisição informa também o número de parcelas.
>
> O rateio é feito por percentual ou por valor fixo em reais. Cada morador
> participante informa o seu percentual ou o seu valor, conforme a forma de
> rateio adotada no lançamento.
>
> Um lançamento com periodicidade semanal ou mensal dá origem a um lançamento
> por parcela. Os lançamentos derivados repetem os demais dados e recebem, ao
> final da descrição, a indicação sequencial da parcela no formato
> `<descricao> <numero>/<total>`. A data de vencimento avança sete dias a cada
> parcela na periodicidade semanal e trinta dias a cada parcela na periodicidade
> mensal, a partir da data de vencimento informada.
>
> O rateio incide sobre o valor de cada parcela, e não sobre o valor total.
>
> A operação é registrada no log de operações do sistema.

### Definições que integram o requisito

Estas definições acompanham o requisito canônico e são fornecidas ao modelo,
por eliminarem ambiguidades que impediriam a avaliação objetiva do resultado.

- **Valor da parcela.** Na periodicidade única, o valor da parcela é o valor
  total. Nas demais, é o valor total dividido pelo número de parcelas,
  arredondado para duas casas decimais. A diferença acumulada pelo
  arredondamento é somada à última parcela, de modo que a soma das parcelas
  seja exatamente igual ao valor total.
- **Valor devido por participante.** No rateio por percentual, é o valor da
  parcela multiplicado pelo percentual do participante, arredondado para duas
  casas decimais, com a diferença acumulada somada ao último participante. No
  rateio por valor fixo, é o valor informado para o participante.
- **Arredondamento.** Duas casas decimais, modo `HALF_UP`.

---

## Critérios de aceitação

1. A requisição de criação responde com o código `201`, com o cabeçalho
   `Location` apontando para o lançamento criado, e com o corpo do lançamento.
2. O lançamento criado tem tipo `DESPESA` e situação `PENDENTE`.
3. No rateio por percentual, a soma dos percentuais dos participantes deve ser
   igual a cem. Caso não seja, a requisição é rejeitada e nenhum lançamento é
   criado.
4. No rateio por valor fixo, a soma dos valores dos participantes deve ser igual
   ao valor da parcela. Caso não seja, a requisição é rejeitada e nenhum
   lançamento é criado.
5. A soma dos valores devidos pelos participantes de um lançamento é exatamente
   igual ao valor daquele lançamento.
6. Na periodicidade única, é criado um único lançamento, sem indicação de
   parcela na descrição e sem número nem total de parcelas.
7. Na periodicidade mensal com `n` parcelas, são criados `n` lançamentos, com
   descrições terminadas em `1/n` a `n/n` e vencimentos espaçados de trinta
   dias a partir da data informada.
8. Na periodicidade semanal com `n` parcelas, o espaçamento entre os
   vencimentos é de sete dias.
9. A soma dos valores dos lançamentos gerados é exatamente igual ao valor total
   informado.
10. Os lançamentos derivados referenciam o lançamento que lhes deu origem.
11. Somente moradores que residem na república informada podem participar do
    rateio. A presença de um morador de outra república, ou sem república,
    resulta em rejeição da requisição.
12. A república informada deve existir. Caso não exista, a resposta é `404`.
13. Requisições com valor total não positivo, sem participantes, ou com número
    de parcelas inferior a um em periodicidade repetitiva, são rejeitadas com
    código `400` ou `422`.
14. Nenhuma requisição rejeitada deixa lançamento, participação ou registro
    parcial persistido.
15. A criação bem-sucedida gera um registro de sucesso no log de operações, no
    formato definido pelas convenções técnicas do projeto.
16. As regras arquiteturais do projeto permanecem satisfeitas.

---

## Observações para a suíte-oráculo

A suíte deve distinguir uma implementação correta de uma implementação
incompleta. Merecem verificação explícita:

- o caso de arredondamento não exato, por exemplo valor total de `100,00` em
  três parcelas, em que a soma das parcelas só fecha se a diferença for
  atribuída à última;
- o rateio percentual que não divide exatamente, por exemplo `100,00` entre três
  participantes;
- a rejeição por soma de percentuais diferente de cem, que uma implementação
  incompleta tende a omitir;
- a ausência de persistência parcial após uma rejeição;
- a numeração das parcelas na descrição, cujo formato é verificável literalmente.
