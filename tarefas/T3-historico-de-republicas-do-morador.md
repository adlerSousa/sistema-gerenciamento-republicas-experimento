# T3 — Histórico de repúblicas do morador

| Campo | Valor |
|---|---|
| Identificador | T3 |
| Tipo de tarefa | Persistência de dados |
| Regras de negócio | BR02.1 (Histórico de repúblicas do morador), BR03 (Representante) |
| Casos de uso | UC16 — Ver histórico de repúblicas que o morador residiu |

---

## Estado inicial no repositório-base

Já existe:

- o desligamento do morador da república, pelo *endpoint*
  `DELETE /api/moradores/{id}/republica`, que devolve o morador à condição de
  sem teto;
- a entidade `Representante`, com o período de mandato de cada representante, o
  que permite identificar quem representava a república em uma data passada;
- a data de ingresso do morador na república.

Não existe nenhuma tabela, entidade ou consulta de histórico de residências. Ao
sair, o vínculo do morador com a república é simplesmente desfeito e a
informação se perde.

---

## Requisito canônico

> Manter o histórico das repúblicas em que o morador já residiu.
>
> A cada saída de um morador de uma república, deve ser registrado um item de
> histórico contendo a república, o período de residência, o representante da
> república à época da saída e a reputação média do morador naquela república.
>
> O histórico deve poder ser consultado por morador, apresentando os itens
> registrados.

### Definições que integram o requisito

- **Período de residência.** Data de ingresso do morador na república e data de
  saída.
- **Representante da época.** Morador cujo mandato de representante da república
  estava vigente na data de saída.
- **Reputação média.** Média aritmética das reputações mensais do morador
  naquela república, considerando os meses compreendidos no período de
  residência. A reputação mensal é a soma do índice de solução de reclamações,
  do índice de realização de tarefas e do índice de compromisso com pagamentos,
  conforme a regra BR02.2. Quando o período não compreender nenhum mês completo
  de dados, a reputação média é registrada como zero.
- **Precisão.** Quatro casas decimais, arredondamento `HALF_UP`.

---

## Critérios de aceitação

1. Ao desligar um morador de uma república, é criado um item de histórico
   correspondente.
2. O item registra a república, a data de ingresso, a data de saída, o
   representante vigente na data de saída e a reputação média.
3. O item permanece registrado após a saída, mesmo que o morador passe a residir
   em outra república.
4. Um morador que residiu em duas repúblicas em momentos distintos possui dois
   itens de histórico, distinguíveis pela república e pelo período.
5. O histórico de um morador pode ser consultado e vem ordenado de forma
   determinística.
6. A consulta do histórico de um morador sem residências anteriores devolve uma
   lista vazia, e não um erro.
7. A consulta do histórico de um morador inexistente responde com código `404`.
8. O desligamento continua a devolver o morador à condição de sem teto, como
   antes da alteração.
9. A alteração de esquema é feita por migração versionada, sem modificar
   migrações já existentes.
10. Nenhum item de histórico é criado quando o desligamento é rejeitado.
11. As regras arquiteturais do projeto permanecem satisfeitas.

---

## Ponto de atenção — sobreposição com a T2

O requisito da Seção 4.5 para esta tarefa inclui a reputação média do morador,
e o repositório-base não possui cálculo de reputação. Em consequência, a T3
exige implementar a mesma regra BR02.2 que constitui o objeto da T2.

Isso não compromete a independência das execuções, pois cada tarefa é executada
a partir do mesmo *commit* base, em ramificação isolada, e nenhuma enxerga o
resultado da outra. Mas significa que:

- a T3 é mais extensa do que a sua classificação como tarefa de persistência
  sugere;
- a regra BR02.2 é exercitada por duas tarefas do conjunto, sob enquadramentos
  diferentes: consulta mensal na T2 e média histórica na T3.

**Decisão a confirmar com o orientador:** manter o requisito como redigido, ou
restringir a T3 ao registro do período e do representante, deixando a reputação
média fora do escopo. A primeira opção preserva a literalidade do texto da
Seção 4.5; a segunda preserva a caracterização da tarefa como exercício de
persistência.

---

## Observações para a suíte-oráculo

- Verificar que o histórico sobrevive a uma segunda mudança de república, caso
  em que uma implementação incompleta tende a sobrescrever o item anterior.
- Verificar a identificação do representante da época com um cenário em que o
  representante mudou durante o período de residência, de modo que o
  representante da data de saída seja diferente do representante da data de
  ingresso.
- Verificar que a consulta não devolve itens de outros moradores.
- Verificar que a migração acrescentada não altera as tabelas já existentes de
  forma incompatível com o restante do sistema.
