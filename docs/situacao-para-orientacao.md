# Situação do trabalho — Setembro de 2026

**Trabalho de Conclusão de Curso II**
Aluno: Adler Amorim De Sousa · Orientador: Prof. Dr. Clayton Vieira Fraga Filho

> Documento de situação para revisão. Reúne o que já foi construído do pacote
> experimental, o que ainda falta, e as decisões que dependem de definição para
> que a coleta possa começar.

---

## 1. Referência de progresso

A Seção 4.13 da monografia define o entregável como um pacote experimental
reproduzível composto por dez componentes. A situação de cada um:

| Componente | Situação |
|---|---|
| Repositório-base | Construído e verificado |
| Regras arquiteturais | Construídas e verificadas |
| Tarefas de evolução | Redigidas, pendentes de aprovação |
| Requisitos e critérios de aceitação | Redigidos, pendentes de aprovação |
| Artefatos de especificação | Não iniciados |
| *Prompts* e arquivos de contexto | Não iniciados |
| Suíte-oráculo independente | Não iniciada |
| Registros das execuções | Não iniciados |
| Base de métricas | Não iniciada |
| Análise dos resultados | Depende das execuções |

No cronograma da Seção 6, isso corresponde à **Atividade 2**, que reúne a
construção do repositório-base, a definição das tarefas de evolução, dos
manifestos de contexto, das métricas e da suíte de avaliação independente. A
primeira e a segunda estão concluídas; as três restantes, não.

---

## 2. O repositório-base

O sistema de gerenciamento de repúblicas estudantis foi implementado conforme a
Seção 4.4: Java 21 com Spring Boot, PostgreSQL, interface web em React com
TypeScript, organizado em Arquitetura Limpa.

**Dimensão:** 110 arquivos de código Java, cinco migrações de esquema, nove
*commits*.

**Camadas e dependências.** As dependências apontam sempre para dentro. A camada
de domínio é composta por classes Java sem qualquer referência a *framework*: as
entidades de persistência residem exclusivamente na infraestrutura, com
mapeadores entre os dois modelos. Essa separação é o que torna a aderência
arquitetural efetivamente verificável.

**Regras de negócio.** As regras BR01 a BR09 da especificação de requisitos estão
implementadas, incluindo o controle de vagas, a restrição de residência única, o
rateio de lançamentos entre moradores participantes, o saldo acumulado da receita
coletiva, as tarefas domésticas, as reclamações e sugestões, e o registro de
operações em arquivo nos formatos CSV, JSON e XML.

**Massa de dados.** O repositório parte de uma massa determinística com três
repúblicas, dez moradores, lançamentos quitados e pendentes, tarefas concluídas
dentro e fora do prazo, e reclamações resolvidas e em aberto. O histórico foi
construído de modo a permitir o cálculo dos três índices de reputação.

### 2.1 Ausências deliberadas

O repositório **não** implementa as funcionalidades correspondentes às tarefas de
evolução. Essa ausência é intencional e constitui exatamente o que cada tarefa
deve produzir:

| Tarefa | Ausente no repositório-base |
|---|---|
| T1 | Criação de lançamento de despesa com rateio, periodicidade e parcelas |
| T2 | Cálculo dos índices de reputação |
| T3 | Persistência do histórico de repúblicas do morador |
| T4 | Validação de vaga disponível e atualização do quantitativo |
| T5 | Recusa do pagamento por saldo insuficiente e registro da falha |
| T6 | Prazo de aviso configurável por república |
| Piloto | Busca de repúblicas com vagas disponíveis |

Cada ausência foi conferida diretamente no código.

---

## 3. Como o código gerado será verificado

A sequência de verificação da Seção 4.10 está implementada e executa em quatro
etapas, na ordem prevista: compilação, testes unitários, testes de integração e
regras arquiteturais. Cada etapa produz um resultado próprio, registrado em
arquivo estruturado junto do identificador do *commit* avaliado.

**Situação atual do repositório-base:**

| Etapa | Resultado |
|---|---|
| Compilação | Aprovado |
| Testes unitários | Aprovado — 35 testes |
| Testes de integração | Aprovado — 12 testes |
| Regras arquiteturais | Aprovado — 9 regras |

Três cuidados foram adotados por afetarem diretamente a validade das medidas:

**Ambiente de execução fixado.** Toda compilação e verificação ocorre dentro de
um ambiente Docker com versões fixadas, e não na máquina do pesquisador. Isso
torna os resultados idênticos em qualquer máquina e permite que terceiros
repliquem a avaliação.

**Falha de compilação não é contada duas vezes.** Quando a compilação falha, as
etapas seguintes são registradas como não executadas, e não como falha, conforme
determina a Seção 4.10.

**Violação arquitetural é medida em separado.** As regras arquiteturais são
verificadas em execução isolada das demais etapas. As camadas são delimitadas por
pacotes, e não por módulos de compilação: se fossem módulos, uma violação de
camada se manifestaria como erro de compilação e contaminaria a métrica de falha
de compilação com o que deveria ser contado como violação arquitetural.

**Independência entre execuções da suíte.** O esquema do banco de testes é
recriado a cada execução. Sem isso, o resultado de um teste poderia depender dos
dados deixados por uma execução anterior, e a aprovação em testes deixaria de ser
um indicador estável entre as 54 execuções.

---

## 4. As tarefas de evolução

As seis tarefas da Seção 4.5 e a tarefa-piloto foram redigidas com requisito
canônico e critérios de aceitação, conforme exige aquela seção.

Cada documento contém quatro partes: o estado inicial no repositório-base, o
requisito canônico entregue ao modelo, os critérios de aceitação que integram o
manifesto de contexto, e observações para a construção da suíte-oráculo.

O número de critérios por tarefa varia de dez a dezesseis, redigidos de forma
verificável para que possam ser convertidos em testes.

As observações para a suíte-oráculo atendem à recomendação da Seção 4.10 de que
os testes distingam uma implementação correta de uma implementação incompleta.
Em cada tarefa foram identificados os pontos que uma implementação parcial tende
a omitir — por exemplo, na T4, verificar não apenas que a resposta é de recusa,
mas que o estado permanece integralmente inalterado após a recusa.

### 4.1 Separação entre os repositórios

As tarefas e a suíte-oráculo **não** residem no repositório-base. Se residissem,
o modelo que executa a T1 poderia ler os enunciados e critérios das demais
tarefas, e a execução deixaria de refletir a condição experimental medida.

O pacote experimental passa, portanto, a ser formado por dois repositórios: o
repositório-base, disponibilizado ao modelo, e o repositório do aparato, que
reúne as tarefas, a suíte-oráculo, os *prompts*, os manifestos, os registros e as
métricas. Em cada execução, o modelo recebe apenas o requisito canônico e os
critérios de aceitação da tarefa em curso.

---

## 5. O que o repositório-base não implementa

Além das ausências deliberadas da Seção 2.1, o repositório-base implementa um
**subconjunto** dos casos de uso da especificação de requisitos. Os módulos
construídos foram aqueles exercitados pelas tarefas de evolução.

Não implementados:

| Caso de uso | Descrição |
|---|---|
| UC7 | Manter perfil de morador |
| UC11 | Consultar resultado mensal da república |
| UC15 | Fazer estorno de lançamento *(regra existe no domínio; falta o recurso HTTP)* |
| UC18 | Visualizar gráfico com resultado mensal |
| RF03 | Acesso ao sistema por autenticação *(a tabela de usuários existe; falta o fluxo)* |

A Seção 4.4 exige do repositório-base arquitetura em camadas, dependências,
regras de negócio e testes automatizados previamente estabelecidos, sem exigir a
totalidade dos casos de uso. A delimitação da Seção 4.2 igualmente restringe o
objeto de estudo ao conjunto definido de tarefas.

**Esta é a primeira questão submetida à orientação** e está detalhada na Seção 6.

---

## 6. Decisões pendentes

As decisões abaixo alteram o que será medido e por isso não foram tomadas
unilateralmente.

### 6.1 Abrangência do repositório-base

O repositório-base deve implementar a totalidade dos casos de uso da
especificação, ou é suficiente o subconjunto exercitado pelas tarefas de
evolução?

A ausência de autenticação merece atenção específica: nenhuma métrica prevista na
Seção 4.11 depende dela, e as operações registram o usuário por parâmetro
explícito, mas trata-se de requisito funcional da especificação.

### 6.2 Terceira alternativa das fórmulas de reputação — T2

As regras BR02.2.1 e BR02.2.2 apresentam três alternativas para cada índice: a
razão entre os totais, um valor correspondente a "caso tenha resolvido
reclamações de outros" — ou "concluído tarefa para outros" — e o valor um.

A alternativa intermediária pressupõe distinguir o morador que soluciona uma
reclamação de terceiro daquele sobre quem a reclamação foi registrada. O modelo
de dados da especificação não sustenta essa distinção: o registro guarda o autor
e os envolvidos, sem a figura do solucionador.

*Interpretação adotada:* a fórmula foi reduzida a duas alternativas, mantendo a
razão e o valor um.

*Alternativa:* acrescentar ao modelo de dados a figura do morador solucionador, o
que amplia o escopo da tarefa.

### 6.3 Casos omissos no índice de compromisso com pagamentos — T2

A regra BR02.2.3 define o índice como a razão entre o somatório dos dias
agendados e o somatório dos dias de pagamento efetivo, sem definir dois casos:

**Ausência de pagamentos no mês**, que produziria divisão por zero.
*Interpretação adotada:* o índice vale um, por analogia com os demais índices.

**Participação vencida e ainda não paga.**
*Interpretação adotada:* considera-se paga no último dia do mês corrente, o que
faz o atraso reduzir o índice de forma crescente.
*Alternativa:* excluí-la do somatório, hipótese em que o atraso não produziria
efeito algum sobre o índice.

### 6.4 Sobreposição entre a T3 e a T2

O requisito da T3 na Seção 4.5 inclui o registro da reputação média do morador na
república. Como o repositório-base não calcula reputação, a T3 exige implementar
a mesma regra BR02.2 que constitui o objeto da T2.

Isso não compromete a independência das execuções, pois cada tarefa parte do
mesmo *commit* base em ramificação isolada e nenhuma enxerga o resultado da
outra. Mas a T3 torna-se mais extensa do que a sua classificação como tarefa de
persistência sugere, e a regra BR02.2 passa a ser exercitada por duas das seis
tarefas.

*Interpretação adotada:* manter o requisito conforme o texto da Seção 4.5.

*Alternativa:* restringir a T3 ao registro do período de residência e do
representante da época, deixando a reputação média fora do escopo.

### 6.5 Congelamento do repositório-base

A Seção 4.4 determina que o *commit* de referência seja fixado para que todas as
execuções partam do mesmo ponto. O congelamento está pendente das decisões
anteriores, uma vez que alterações no escopo do repositório-base ou nas tarefas
implicariam novo ponto de partida.

---

## 7. Trabalho subsequente

Concluídas as decisões da Seção 6, restam quatro componentes antes do início da
coleta:

**Suíte-oráculo.** Testes por tarefa, mantidos fora do contexto disponibilizado ao
modelo e executados em etapa externa de avaliação, conforme a Seção 4.10.

**Manifestos de contexto e *prompts*.** O contexto basal comum às três condições e
os artefatos próprios de cada tratamento, conforme as Seções 4.7 e 4.8.

**Instrumentos de registro.** O registro estruturado de cada execução, com os
campos enumerados na Seção 4.12, e o registro de intervenções humanas como
variável de processo.

**Consolidação das métricas.** A agregação das 54 execuções em taxas comparáveis
entre as três condições.

A execução da tarefa-piloto antecede a coleta e serve para calibrar o protocolo,
os *prompts* e os instrumentos, conforme previsto na Seção 4.5.

---

## 8. Situação em relação ao cronograma

O cronograma da Seção 6 prevê a Atividade 2 entre julho e agosto, e o início das
Atividades 3 e 4 em agosto. A construção do repositório-base e a definição das
tarefas estão concluídas; a suíte de avaliação, os manifestos e as métricas,
não. O início da coleta depende das decisões da Seção 6 deste documento.
