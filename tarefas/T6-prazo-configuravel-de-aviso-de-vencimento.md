# T6 — Prazo configurável de aviso de vencimento

| Campo | Valor |
|---|---|
| Identificador | T6 |
| Tipo de tarefa | Alteração de comportamento existente |
| Regras de negócio | BR01.1 (Definições de notificação da república), BR03 (Representante) |
| Requisitos funcionais | RF01 — Notificar sobre lançamentos com vencimento próximo |

---

## Estado inicial no repositório-base

Já existe:

- o *endpoint*
  `POST /api/moradores/{id}/notificacoes/avisos-vencimento`, que apura os
  lançamentos pendentes com vencimento próximo e registra as notificações
  correspondentes;
- a manutenção do aviso após o vencimento, enquanto o lançamento permanecer
  pendente, conforme RF01;
- a não repetição de avisos já emitidos para o mesmo morador e lançamento;
- a entidade `Representante`, que identifica o representante vigente de cada
  república.

O prazo de antecedência é **fixo em cinco dias** e vale igualmente para todas as
repúblicas. Ele é lido de uma propriedade de configuração da aplicação,
`republicas.notificacao.prazo-aviso-vencimento-dias-padrao`, e não pode ser
alterado em tempo de execução nem definido por república.

---

## Requisito canônico

> Tornar configurável, pelo representante da república, o prazo de antecedência
> do aviso de lançamentos com vencimento próximo.
>
> Cada república possui o seu próprio prazo, expresso em dias. O valor padrão é
> de cinco dias, aplicável às repúblicas que não o alterarem. O representante da
> república pode alterá-lo para o prazo que desejar.
>
> A apuração dos avisos de vencimento passa a considerar o prazo da república do
> morador, e não mais um prazo único para todo o sistema.

---

## Critérios de aceitação

1. Toda república possui um prazo de aviso de vencimento, e as repúblicas já
   existentes passam a ter o valor cinco após a alteração de esquema.
2. Uma república criada após a alteração recebe o prazo padrão de cinco dias
   quando ele não for informado.
3. O representante da república consegue alterar o prazo, e o novo valor é
   persistido.
4. A apuração dos avisos considera o prazo da república do morador.
5. Duas repúblicas com prazos diferentes produzem conjuntos de avisos diferentes
   para lançamentos com as mesmas datas de vencimento relativas.
6. Ampliar o prazo de uma república faz passar a ser avisado um lançamento que
   antes não era, sem que nada mais mude.
7. Reduzir o prazo de uma república faz deixar de ser avisado um lançamento que
   antes era.
8. Somente o representante vigente da república pode alterar o prazo. A
   tentativa por outro morador é recusada.
9. A tentativa de definir prazo negativo é recusada.
10. O aviso continua a ser mantido após o vencimento, enquanto o lançamento
    permanecer pendente, como antes da alteração.
11. Avisos já emitidos continuam a não ser repetidos, como antes da alteração.
12. A alteração de esquema é feita por migração versionada, sem modificar
    migrações já existentes.
13. As regras arquiteturais do projeto permanecem satisfeitas.

---

## Observações para a suíte-oráculo

Esta é a única tarefa do conjunto que altera comportamento já existente e
verificado. A suíte deve, portanto, verificar tanto o comportamento novo quanto
a preservação do antigo.

Merecem verificação explícita:

- os critérios 10 e 11, que descrevem comportamento preexistente e que uma
  reimplementação do zero tende a perder;
- o critério 5, que distingue uma implementação que de fato passou a usar o
  prazo por república de uma que apenas acrescentou o campo e continuou lendo a
  propriedade global;
- as repúblicas da massa de dados inicial, que devem receber o valor cinco pela
  migração, e não permanecer sem valor;
- o caso de fronteira em que o vencimento ocorre exatamente no último dia do
  prazo, que deve ser avisado.
