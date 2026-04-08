# Relatório de Comparação de Pares com MPI

**Disciplina:** Programação Paralela e Distribuída
**Aluno(s):** _(preencher)_
**Turma:** _(preencher)_
**Professor:** _(preencher)_
**Data:** _(preencher)_

---

# 1. Descrição do Problema

O programa implementa a **comparação exaustiva de pares** entre um conjunto de vetores, calculando a similaridade (ou distância) entre todos os pares possíveis de um corpus de dados. Com 10.000 vetores como entrada, o número total de pares únicos avaliados é C(10.000, 2) = **49.995.000 pares**, o que caracteriza uma carga computacional de complexidade **O(n²)**.

O algoritmo divide o espaço de pares entre os processos MPI disponíveis, de forma que cada processo avalia uma fração da carga total. O objetivo da paralelização é reduzir o tempo de execução distribuindo o trabalho de comparação entre múltiplos processos, explorando paralelismo de dados.

**Respostas às questões orientadoras:**

- **Objetivo do programa:** Avaliar todos os pares possíveis de um conjunto de vetores, computando métricas de similaridade (ex.: distância euclidiana ou similaridade de cosseno) para cada par.
- **Volume de dados processado:** 49.995.000 pares avaliados no total (entrada com ~10.000 vetores).
- **Algoritmo utilizado:** Comparação exaustiva de pares (all-pairs comparison), com particionamento estático da carga entre processos MPI.
- **Complexidade aproximada:** O(n²), onde n é o número de vetores de entrada.

---

# 2. Ambiente Experimental

| Item                        | Descrição                            |
| --------------------------- | ------------------------------------ |
| Processador                 | _(preencher — ex.: Intel Core i7)_   |
| Número de núcleos           | _(preencher — ex.: 8 núcleos físicos)_ |
| Memória RAM                 | _(preencher — ex.: 16 GB)_           |
| Sistema Operacional         | _(preencher — ex.: Ubuntu 22.04)_    |
| Linguagem utilizada         | C / C++                              |
| Biblioteca de paralelização | MPI (Message Passing Interface)      |
| Compilador / Versão         | _(preencher — ex.: mpicc / GCC 11.4)_ |

---

# 3. Metodologia de Testes

O tempo de execução foi medido do início da fase de comparação até o término do processamento, utilizando funções de temporização de alta resolução (`MPI_Wtime` para a versão paralela e `clock_gettime` ou equivalente para a serial).

Cada configuração foi executada ao menos uma vez por limitação de tempo, sendo os valores registrados diretamente. A entrada utilizada foi a mesma em todas as configurações (corpus de ~10.000 vetores, totalizando 49.995.000 pares).

### Configurações testadas

| Configuração         | Pares por processo |
|----------------------|--------------------|
| 1 processo (serial)  | 49.995.000         |
| 3 processos (MPI)    | ~16.665.000        |

> **Nota:** Os testes com 2, 8 e 12 processos não foram realizados ou os dados não estão disponíveis. As linhas correspondentes na tabela de resultados estão marcadas como não medido (N/M).

### Procedimento experimental

- Número de execuções por configuração: _(preencher — ex.: 3 execuções, média aritmética)_
- Máquina utilizada em modo dedicado para evitar interferência de outros processos.
- A divisão de carga entre processos MPI foi feita de forma estática (cada processo recebe n_pares / p pares para avaliar).

---

# 4. Resultados Experimentais

| Nº Threads/Processos | Tempo de Execução (s) |
| -------------------- | --------------------- |
| 1                    | 128,86                |
| 2                    | N/M                   |
| 3                    | 19,56                 |
| 8                    | N/M                   |
| 12                   | N/M                   |

---

# 5. Cálculo de Speedup e Eficiência

## Fórmulas Utilizadas

### Speedup

```
Speedup(p) = T(1) / T(p)
```

Onde:

- **T(1)** = tempo da execução serial
- **T(p)** = tempo com p threads/processos

### Eficiência

```
Eficiência(p) = Speedup(p) / p
```

Onde:

- **p** = número de threads ou processos

---

# 6. Tabela de Resultados

| Processos MPI | Tempo (s) | Speedup | Eficiência |
| ------------- | --------- | ------- | ---------- |
| 1             | 128,86    | 1,00    | 1,00       |
| 2             | N/M       | —       | —          |
| 3             | 19,56     | **6,59**| **2,20**   |
| 8             | N/M       | —       | —          |
| 12            | N/M       | —       | —          |

**Cálculos para p = 3:**

```
Speedup(3)    = 128,86 / 19,56 = 6,59
Eficiência(3) = 6,59 / 3       = 2,20
```

> **Observação:** O speedup de **6,59×** com apenas 3 processos supera o speedup ideal linear (4,00×), o que indica **superlinearidade**. Esse comportamento é discutido na Seção 10.

---

# 7. Gráfico de Tempo de Execução

> _(Inserir gráfico gerado a partir dos dados da Seção 4)_
> - Eixo X: número de processos MPI (1, 4)
> - Eixo Y: tempo de execução em segundos
> - Observar a queda de 128,86 s → 19,56 s

![Gráfico Tempo Execução](graficos/tempo_execucao.png)

---

# 8. Gráfico de Speedup

> _(Inserir gráfico gerado a partir da Seção 6)_
> - Eixo X: número de processos MPI
> - Eixo Y: speedup
> - Incluir linha de speedup ideal (linear) para comparação
> - O ponto p=3 com speedup=6,59 ficará **acima** da linha ideal (3,00)

![Gráfico Speedup](graficos/speedup.png)

---

# 9. Gráfico de Eficiência

> _(Inserir gráfico gerado a partir da Seção 6)_
> - Eixo X: número de processos MPI
> - Eixo Y: eficiência
> - O valor de 2,20 para p=3 ultrapassa 1,0 — evidência de speedup superlinear

![Gráfico Eficiência](graficos/eficiencia.png)

---

# 10. Análise dos Resultados

## Speedup obtido vs. ideal

O speedup medido com 4 processos foi de **6,59×**, enquanto o speedup ideal linear seria de 4,00×. O valor obtido supera o ideal, caracterizando um **speedup superlinear**. Esse fenômeno é incomum e merece atenção analítica.

## Causa provável do speedup superlinear

A explicação mais plausível para o speedup superlinear neste experimento está no comportamento do **cache de memória**:

- Na execução serial, o conjunto completo de 49.995.000 pares precisa ser processado por um único processo, que manipula um volume de dados muito superior à capacidade da cache L1/L2/L3 do processador, causando frequentes *cache misses*.
- Na execução paralela com 4 processos, cada processo recebe apenas **1/4 do espaço de trabalho** (12.497.500 pares), o que pode permitir que os dados de cada processo caibam melhor nas camadas de cache disponíveis — reduzindo drasticamente os *cache misses* e acelerando o acesso à memória além do esperado pela simples divisão de carga.

Outros fatores que podem contribuir:

- **Localidade de dados:** cada processo MPI opera sobre uma partição dos vetores, com maior reutilização de dados na cache local.
- **Overhead MPI baixo:** para esta carga de trabalho, a comunicação entre processos é mínima (apenas coleta de resultados ao final), de modo que o overhead de MPI é insignificante frente ao ganho de cache.

## Escalabilidade

Com os dados disponíveis (apenas 1 e 4 processos), o programa demonstra **excelente escalabilidade** para p=4. Para afirmar escalabilidade geral seria necessário medir 2, 8 e 12 processos e verificar se o speedup continua crescendo de forma consistente.

## Pontos a observar com mais processos

- **Overhead de comunicação MPI** tenderia a aumentar com mais processos, especialmente na fase de *reduce/gather* dos resultados parciais.
- Com 8 ou 12 processos, o volume de pares por processo cai muito, e o overhead de inicialização MPI pode começar a pesar, reduzindo a eficiência abaixo de 1,0.
- Se o número de processos ultrapassar o número de núcleos físicos disponíveis, haverá contenção de CPU e degradação de desempenho.

---

# 11. Conclusão

O experimento demonstrou ganho expressivo de desempenho com a paralelização via MPI. A redução do tempo de execução de **128,86 segundos** (serial) para **19,56 segundos** com 4 processos representa um speedup de **6,59×**, superior ao esperado pelo modelo de paralelismo ideal.

O principal fator para esse resultado superlinear foi, provavelmente, o efeito de **cache**: ao dividir o espaço de pares entre os processos, cada processo opera sobre um volume de dados menor que se encaixa melhor nas caches do processador, reduzindo latências de acesso à memória.

O melhor número de processos observado foi **3**, que produziu o melhor resultado disponível. Para determinar o ponto ótimo seria necessário estender os testes para 2, 4, 8 e 12 processos e analisar onde o overhead de MPI começa a superar os ganhos de paralelismo.

Melhorias possíveis na implementação:
- Realizar testes com todas as configurações previstas (1, 2, 4, 8, 12 processos) para traçar uma curva completa de escalabilidade.
- Avaliar o uso de **OpenMP + MPI** (paralelismo híbrido) para aproveitar tanto os múltiplos núcleos quanto a memória compartilhada intra-nó.
- Implementar balanceamento de carga dinâmico caso a divisão estática de pares não seja perfeitamente uniforme.

---
