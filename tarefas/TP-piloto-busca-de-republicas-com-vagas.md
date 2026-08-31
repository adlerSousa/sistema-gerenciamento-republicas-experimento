# TP — Tarefa-piloto: busca de repúblicas com vagas disponíveis

| Campo | Valor |
|---|---|
| Identificador | TP |
| Situação | **Fora da amostra.** Utilizada para calibrar o protocolo, os *prompts* e os instrumentos de coleta. Não compõe a análise. |
| Regras de negócio | BR07 (Vagas), BR01 (República) |
| Casos de uso | UC13 — Buscar vagas em repúblicas |

---

## Finalidade

Conforme a Seção 4.5, esta tarefa é executada antes das demais para calibrar o
protocolo de execução, os *prompts* e os instrumentos de coleta, sem compor a
análise dos resultados. Serve também para o pesquisador exercitar a operação das
três condições experimentais antes do início da coleta.

Foi escolhida por ser representativa em estrutura, porém menos extensa que as
tarefas da amostra: exercita consulta, filtros e regra de negócio simples, sem
alterar estado.

---

## Estado inicial no repositório-base

Já existe:

- o *endpoint* `GET /api/republicas`, que lista todas as repúblicas e aceita
  filtro por nome;
- o cálculo das vagas disponíveis pela diferença entre o total de vagas e as
  vagas ocupadas, já exposto na resposta.

Não existe consulta que restrinja o resultado às repúblicas com vagas
disponíveis, nem os filtros por vantagens oferecidas e por faixa de despesa
média.

---

## Requisito canônico

> Implementar a busca de repúblicas com vagas disponíveis.
>
> A consulta apresenta as repúblicas que possuem ao menos uma vaga disponível.
> O resultado pode ser filtrado por nome, por parte do texto das vantagens
> oferecidas e por intervalo de valor médio de despesas por morador.
>
> Os filtros são opcionais e combináveis entre si.

---

## Critérios de aceitação

1. A consulta apresenta apenas repúblicas com ao menos uma vaga disponível.
2. Repúblicas com todas as vagas ocupadas não aparecem no resultado.
3. O filtro por nome seleciona as repúblicas cujo nome contém o texto informado,
   sem diferenciar maiúsculas de minúsculas.
4. O filtro por vantagens seleciona as repúblicas cujo texto de vantagens contém
   o termo informado, sem diferenciar maiúsculas de minúsculas.
5. O filtro por faixa de despesa média seleciona as repúblicas cuja despesa média
   por morador esteja entre os limites informados, inclusive.
6. Os limites da faixa são independentes: informar apenas o limite inferior ou
   apenas o superior é admitido.
7. Filtros combinados são aplicados conjuntamente.
8. A consulta sem filtro algum apresenta todas as repúblicas com vagas
   disponíveis.
9. Uma consulta sem resultados devolve lista vazia com código `200`, e não erro.
10. A ordenação do resultado é determinística.
11. As regras arquiteturais do projeto permanecem satisfeitas.

---

## Observações para a suíte-oráculo

Na massa de dados inicial, as repúblicas `1` e `2` possuem duas vagas
disponíveis cada e a república `3` está com todas as vagas ocupadas, o que
permite verificar diretamente os critérios 1 e 2.

Merecem verificação explícita:

- a exclusão da república sem vagas, que é o núcleo da tarefa e que uma
  implementação incompleta tende a omitir ao apenas acrescentar os filtros;
- a combinação de dois filtros, que distingue a aplicação conjunta da aplicação
  de apenas um deles;
- os limites inclusivos da faixa de despesa média.
