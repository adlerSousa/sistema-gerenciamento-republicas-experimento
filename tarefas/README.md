# Tarefas de evolução

Conjunto de tarefas executadas sobre o repositório-base nas três condições
experimentais, conforme a Seção 4.5 da monografia.

Cada tarefa é executada em **três repetições independentes** em cada uma das
**três condições**, totalizando **54 execuções principais**. A tarefa-piloto é
executada antes das demais e **não compõe a análise**.

| Tarefa | Tipo | Documento |
|---|---|---|
| TP | Calibração do protocolo *(fora da amostra)* | [Busca de repúblicas com vagas](TP-piloto-busca-de-republicas-com-vagas.md) |
| T1 | Criação de *endpoint* | [Lançamento de despesa](T1-endpoint-de-lancamento-de-despesa.md) |
| T2 | Implementação de regra de negócio | [Reputação mensal do morador](T2-reputacao-mensal-do-morador.md) |
| T3 | Persistência de dados | [Histórico de repúblicas do morador](T3-historico-de-republicas-do-morador.md) |
| T4 | Validação de entrada | [Validação de vagas no ingresso](T4-validacao-de-vagas-no-ingresso.md) |
| T5 | Tratamento de erro | [Pagamento com receita coletiva](T5-erro-de-pagamento-com-receita-coletiva.md) |
| T6 | Alteração de comportamento existente | [Prazo configurável de aviso](T6-prazo-configuravel-de-aviso-de-vencimento.md) |

## Estrutura dos documentos

Cada documento contém:

- **Estado inicial no repositório-base** — o que já existe e o que falta. Serve
  ao pesquisador e à construção da suíte-oráculo; não integra o material
  entregue ao modelo.
- **Requisito canônico** — o enunciado da tarefa, **idêntico nas três
  condições**. É o texto entregue ao modelo, junto das definições que o
  acompanham.
- **Critérios de aceitação** — as condições que a implementação deve satisfazer.
  Integram o manifesto de contexto, conforme a Seção 4.7.
- **Observações para a suíte-oráculo** — orientações para que os testes
  distingam uma implementação correta de uma implementação incompleta, cuidado
  recomendado pela Seção 4.10.

## O que o modelo recebe

Da presente pasta, o modelo recebe, em cada execução, **apenas** o requisito
canônico e os critérios de aceitação **da tarefa em curso**. Não recebe o estado
inicial, as observações para a suíte-oráculo nem qualquer documento das demais
tarefas.

## Pontos pendentes de decisão

| Tarefa | Ponto | Onde está descrito |
|---|---|---|
| T2 | Terceira alternativa das fórmulas de ISS e IRT, não sustentada pelo modelo de dados | [T2, Decisões de interpretação](T2-reputacao-mensal-do-morador.md#decisões-de-interpretação--pendentes-de-validação) |
| T2 | Divisão por zero no índice de compromisso com pagamentos | idem |
| T2 | Tratamento de participações vencidas e não pagas | idem |
| T3 | Sobreposição com a T2 pela exigência da reputação média | [T3, Ponto de atenção](T3-historico-de-republicas-do-morador.md#ponto-de-atenção--sobreposição-com-a-t2) |

Estes pontos devem ser resolvidos antes do congelamento das tarefas, pois
alteram o que será medido.
