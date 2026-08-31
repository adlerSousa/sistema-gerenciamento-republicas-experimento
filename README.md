# Aparato experimental

Este repositório reúne os componentes do pacote experimental que **não** podem
ficar visíveis ao modelo de linguagem durante a geração de código.

O pacote experimental completo, previsto na Seção 4.13 da monografia, é formado
por **dois repositórios**:

| Repositório | Conteúdo | Visível ao modelo |
|---|---|---|
| `sistema-gerenciamento-republicas` | Repositório-base, convenções técnicas, regras arquiteturais | Sim |
| `sistema-gerenciamento-republicas-experimento` | Tarefas, suíte-oráculo, *prompts*, manifestos, registros e métricas | Não |

## Por que dois repositórios

A Seção 4.10 estabelece que a suíte-oráculo permanece "em diretório ou
repositório não acessível ao modelo" e que a avaliação ocorre em etapa externa,
executada sobre a ramificação produzida.

O mesmo cuidado se aplica às tarefas: o modelo que executa a T1 não pode ler os
enunciados e os critérios de aceitação da T2 à T6, sob pena de a execução deixar
de refletir a condição experimental que se pretende medir.

Durante cada execução, o modelo recebe **apenas** o requisito canônico e os
critérios de aceitação da tarefa em curso, por meio do manifesto de contexto
(Seção 4.7).

## Organização

```
.
├── tarefas/      Requisitos canônicos e critérios de aceitação das tarefas
├── oraculo/      Suíte de testes independente, executada na avaliação externa
├── prompts/      Prompts das três condições experimentais
├── manifestos/   Manifestos de contexto por tarefa e condição
├── execucoes/    Registros das execuções realizadas
└── metricas/     Pipeline de métricas e resultados consolidados
```

## Situação

| Componente | Situação |
|---|---|
| Tarefas e critérios de aceitação | Redigidos, **pendentes de aprovação** |
| Suíte-oráculo | Não iniciada |
| *Prompts* das três condições | Não iniciados |
| Manifestos de contexto | Não iniciados |
| Registros das execuções | Não iniciados |
| Pipeline de métricas | Não iniciado |
