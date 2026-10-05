# Fraud Anomaly Detection & Behavioral Risk Analytics

## Detecção de anomalias, velocidade transacional e comportamento suspeito com Python e Machine Learning

Este projeto apresenta uma abordagem prática para **detecção de anomalias e comportamentos transacionais suspeitos**, combinando engenharia de features, regras de negócio, análise comportamental e Machine Learning não supervisionado.

O objetivo não foi simplesmente construir um modelo capaz de gerar alertas.

A pergunta central foi:

> **Como aumentar a capacidade de detectar fraude sem transformar comportamento legítimo em uma avalanche de falsos positivos?**

Ao longo do experimento foram comparadas estratégias baseadas em **velocity features**, regras multissinal, comportamento histórico do cliente, **Isolation Forest** e uma abordagem híbrida.

Além das métricas tradicionais de Machine Learning, o projeto incorpora uma dimensão fundamental para Fraud Analytics:

**o custo dos erros para o negócio.**

---

# 1. Problema de negócio

Sistemas antifraude precisam identificar rapidamente comportamentos como:

- alta velocidade transacional;
- múltiplas operações em pequenas janelas de tempo;
- movimentações incompatíveis com o histórico do cliente;
- uso de novos dispositivos;
- valores atípicos;
- combinações incomuns de comportamento.

O desafio é que:

> **comportamento atípico não significa necessariamente fraude.**

Um sistema excessivamente sensível pode gerar muitos falsos positivos, aumentando:

- revisões manuais;
- bloqueios indevidos;
- autenticações adicionais;
- custos operacionais;
- atrito na jornada;
- insatisfação de clientes legítimos.

Por outro lado, reduzir demais a sensibilidade pode aumentar os falsos negativos e permitir que fraudes reais atravessem o sistema.

Portanto, Fraud Analytics exige encontrar um equilíbrio entre:

**detecção, custo e experiência do cliente.**

---

# 2. Objetivos

O experimento foi desenvolvido para avaliar diferentes estratégias de detecção:

- Velocity Rules;
- regras multissinal;
- análise comportamental por cliente;
- regras contextuais;
- Isolation Forest;
- modelo híbrido combinando regras e Machine Learning.

As estratégias foram avaliadas utilizando:

- Precision;
- Recall;
- False Positive Rate — FPR;
- False Positives — FP;
- False Negatives — FN;
- valor financeiro fraudulento capturado;
- valor financeiro fraudulento não detectado;
- custo econômico hipotético dos erros.

---

# 3. Tecnologias

- Python
- Pandas
- NumPy
- Matplotlib
- Scikit-learn
- Jupyter Notebook

Algoritmo principal de Machine Learning:

**Isolation Forest**

---

# 4. Dataset

Foi utilizado um dataset **sintético**, desenvolvido especificamente para fins educacionais e experimentais.

O conjunto final contém:

- **5.120 transações**
- **5.000 transações legítimas**
- **120 transações fraudulentas**
- aproximadamente **2,34% de fraude**

As transações incluem informações como:

- cliente;
- cartão;
- estabelecimento;
- dispositivo;
- timestamp;
- valor;
- tipo de transação;
- indicador de dispositivo novo.

Fraudes sintéticas foram adicionadas ao conjunto para representar cenários de ataque com concentração temporal, valores elevados e utilização de novos dispositivos.

---

# 5. Feature Engineering

Uma das principais etapas do projeto foi transformar eventos transacionais em sinais de comportamento.

## Tempo desde a última transação

Foi calculado o intervalo entre uma transação e a anterior do mesmo cliente.

Essa feature permite identificar comportamentos de alta frequência.

---

## Velocity Features

Foram criadas janelas temporais como:

`transactions_10min`

Número de transações realizadas pelo cliente em uma janela de 10 minutos.

`transactions_1h`

Número de transações realizadas na última hora.

`amount_1h`

Valor financeiro acumulado pelo cliente dentro da janela de uma hora.

Essas features permitem transformar sequências de eventos em variáveis numéricas utilizáveis por regras e modelos.

---

# 6. Behavioral Features

Além da velocidade transacional, foi criado um baseline comportamental individual para cada cliente.

Foram calculadas características como:

- valor médio;
- valor mediano;
- desvio padrão;
- quantidade histórica de transações.

A partir desse baseline foram construídas duas features importantes.

## Amount vs Customer Average

`amount_vs_customer_avg`

Representa quantas vezes o valor atual é maior ou menor que o valor médio daquele cliente.

Nas transações fraudulentas do experimento, esse indicador apresentou média próxima de **4 vezes o comportamento médio do cliente**.

---

## Customer Z-Score

`amount_zscore_customer`

Mede quantos desvios padrão o valor da transação está distante do comportamento médio daquele cliente.

Essa abordagem permite analisar uma transação dentro de seu contexto individual.

Uma compra de R$ 1.000 pode ser absolutamente normal para determinado cliente e altamente atípica para outro.

---

# 7. Estratégias de detecção

## Regra 1 — Velocity

A primeira abordagem utilizou o intervalo temporal entre transações para identificar comportamentos extremamente rápidos.

Resultado:

| Métrica | Resultado |
|---|---:|
| Precision | 88,89% |
| Recall | 100,00% |
| FPR | 0,30% |
| FP | 15 |
| FN | 0 |

Embora o resultado seja excelente, existe uma importante limitação experimental.

As fraudes sintéticas foram deliberadamente construídas utilizando bursts de transações em pequenos intervalos de tempo.

Consequentemente, existe um **viés favorável à regra de velocity**.

Esse resultado não deve ser interpretado como evidência de que uma regra simples de velocidade seria suficiente em produção.

---

# 8. Regra 2 — Multissinal

A segunda estratégia combinou:

- quantidade de transações em 10 minutos;
- valor financeiro acumulado em uma hora.

Resultado:

| Métrica | Resultado |
|---|---:|
| Precision | 100,00% |
| Recall | 75,00% |
| FPR | 0,00% |
| FP | 0 |
| FN | 30 |

A regra eliminou falsos positivos, porém deixou passar **30 das 120 fraudes**.

Isso demonstra um trade-off clássico:

> aumentar a precisão pode reduzir a capacidade de detectar fraude real.

---

# 9. Regra 3 — Contextual

A terceira estratégia combinou dois caminhos independentes de risco:

**Velocity + volume financeiro**

OU

**desvio comportamental do cliente**

Resultado:

| Métrica | Resultado |
|---|---:|
| Precision | 42,58% |
| Recall | 90,83% |
| FPR | 2,94% |
| FP | 147 |
| FN | 11 |

O Recall aumentou significativamente em relação à regra multissinal.

Porém, o aumento da sensibilidade trouxe um custo:

**147 transações legítimas passaram a gerar alertas.**

Esse resultado reforça um princípio importante deste projeto:

> **Anomalia não é sinônimo de fraude.**

---

# 10. Isolation Forest

O próximo passo foi utilizar Machine Learning não supervisionado.

O **Isolation Forest** identifica observações que são mais facilmente isoladas da maioria dos dados por meio de particionamentos aleatórios.

Diferentemente de um modelo supervisionado, o algoritmo não recebeu `fraud_real` durante o treinamento.

Foram utilizadas as seguintes features:

- `amount`
- `transactions_10min`
- `transactions_1h`
- `amount_1h`
- `new_device`
- `amount_vs_customer_avg`
- `amount_zscore_customer`

O experimento utilizou aproximadamente **3% de contamination**.

O algoritmo identificou:

- **154 anomalias**
- **4.966 observações consideradas normais**

Somente após o treinamento essas anomalias foram comparadas com os rótulos conhecidos de fraude.

---

# 11. Resultado do Isolation Forest

Matriz de resultados:

- TN: **4.958**
- FP: **42**
- FN: **8**
- TP: **112**

Métricas:

| Métrica | Resultado |
|---|---:|
| Precision | **72,73%** |
| Recall | **93,33%** |
| FPR | **0,84%** |

O modelo encontrou:

**112 das 120 fraudes**

sem utilizar o rótulo `fraud_real` durante o treinamento.

Entretanto, também classificou 42 transações legítimas como anômalas.

Isso evidencia a diferença entre:

**detecção de anomalia**

e

**classificação de fraude**.

---

# 12. Investigando os falsos negativos

Uma etapa importante do projeto foi não parar nas métricas agregadas.

As **8 fraudes não detectadas pelo Isolation Forest** foram analisadas individualmente.

O padrão observado foi particularmente interessante:

- `transactions_10min = 1`
- `transactions_1h = 1`
- valores entre aproximadamente R$ 566 e R$ 933;
- desvio comportamental moderado.

Essas fraudes não apresentavam velocity.

Eram operações isoladas e menos extremas no espaço multidimensional.

Isso demonstra uma limitação importante:

> **Uma fraude pode parecer estatisticamente normal quando observada isoladamente.**

---

# 13. Regras e Machine Learning encontram as mesmas fraudes?

Não completamente.

A análise mostrou que:

- a regra contextual encontrou fraudes que o Isolation Forest não encontrou;
- o Isolation Forest encontrou fraudes que a regra contextual não encontrou.

Foram identificadas:

- **2 fraudes detectadas pela regra contextual e perdidas pelo Isolation Forest**
- **5 fraudes detectadas pelo Isolation Forest e perdidas pela regra contextual**

Isso forneceu evidência de complementaridade entre os mecanismos de detecção.

A partir dessa descoberta foi criada uma abordagem híbrida.

---

# 14. Modelo Híbrido

O modelo híbrido gera um alerta quando:

**Regra Contextual OR Isolation Forest**

Resultado:

- TN: **4.850**
- FP: **150**
- FN: **6**
- TP: **114**

Métricas:

| Métrica | Resultado |
|---|---:|
| Precision | 43,18% |
| Recall | **95,00%** |
| FPR | 3,00% |

O híbrido aumentou a cobertura de fraude.

Entretanto, o ganho teve um preço operacional importante:

**150 falsos positivos.**

---

# 15. Comparação geral

| Estratégia | Precision | Recall | FPR | FP | FN |
|---|---:|---:|---:|---:|---:|
| Velocity | 88,89% | 100,00% | 0,30% | 15 | 0 |
| Multissinal | 100,00% | 75,00% | 0,00% | 0 | 30 |
| Contextual | 42,58% | 90,83% | 2,94% | 147 | 11 |
| Isolation Forest | **72,73%** | **93,33%** | **0,84%** | **42** | **8** |
| Híbrido | 43,18% | **95,00%** | 3,00% | 150 | **6** |

O híbrido apresentou Recall superior ao Isolation Forest.

Mas isso não significa automaticamente que ele seja a melhor escolha.

---

# 16. Incorporando custo à decisão

Foi criado um cenário hipotético:

- custo de FP = **R$ 25**
- custo de FN = **R$ 1.000**

A função simplificada foi:

`Custo = FP × custo_FP + FN × custo_FN`

Resultados:

| Modelo | Custo |
|---|---:|
| Velocity | R$ 375 |
| Isolation Forest | **R$ 9.050** |
| Híbrido | R$ 9.750 |
| Contextual | R$ 14.675 |
| Multissinal | R$ 30.000 |

O excelente resultado da regra Velocity deve novamente ser interpretado considerando o viés existente na geração do dataset sintético.

Entre Isolation Forest e Híbrido, o Isolation Forest apresentou menor custo sob essas premissas.

---

# 17. Análise de sensibilidade

Uma pergunta importante foi:

**E se o custo de uma fraude não detectada mudar?**

O custo dos falsos negativos foi variado mantendo o custo de FP em R$ 25.

| Custo FN | Isolation Forest | Híbrido |
|---:|---:|---:|
| R$ 250 | R$ 3.050 | R$ 5.250 |
| R$ 500 | R$ 5.050 | R$ 6.750 |
| R$ 1.000 | R$ 9.050 | R$ 9.750 |
| **R$ 1.350** | **R$ 11.850** | **R$ 11.850** |
| R$ 1.500 | R$ 13.050 | **R$ 12.750** |
| R$ 2.500 | R$ 21.050 | **R$ 18.750** |
| R$ 5.000 | R$ 41.050 | **R$ 33.750** |

Foi encontrado um ponto de equilíbrio próximo de:

**R$ 1.350 por falso negativo.**

Abaixo desse valor, o Isolation Forest apresenta menor custo no cenário analisado.

Acima desse valor, o ganho de Recall do modelo híbrido passa a compensar seus falsos positivos adicionais.

Portanto:

> **Não existe melhor modelo independentemente do contexto econômico.**

---

# 18. Captura de valor fraudulento

Contar fraudes também não conta toda a história.

Uma fraude de R$ 100 e uma fraude de R$ 10.000 representam igualmente um FN nas métricas tradicionais, mas possuem impactos financeiros diferentes.

O valor fraudulento total do experimento foi:

**R$ 123.394,80**

Resultados:

| Modelo | Fraude capturada | Fraude perdida | Valor capturado |
|---|---:|---:|---:|
| Velocity | R$ 123.394,80 | R$ 0,00 | 100,00% |
| Multissinal | R$ 93.298,47 | R$ 30.096,33 | 75,61% |
| Contextual | R$ 114.402,21 | R$ 8.992,59 | 92,71% |
| Isolation Forest | **R$ 117.269,61** | **R$ 6.125,19** | **95,04%** |
| Híbrido | **R$ 118.756,37** | **R$ 4.638,43** | **96,24%** |

O híbrido capturou mais valor fraudulento.

Entretanto, para recuperar aproximadamente **R$ 1.486,76 adicionais** em valor fraudulento quando comparado ao Isolation Forest, gerou **108 falsos positivos adicionais**.

Sob a hipótese de R$ 25 por falso positivo:

`108 × R$ 25 = R$ 2.700`

Portanto, nesse cenário simplificado, o custo operacional adicional supera o valor fraudulento adicional identificado.

---

# 19. Principal conclusão

O experimento começou como um problema de detecção de anomalias.

Terminou como um problema de **decisão**.

O modelo com maior Recall não é necessariamente o melhor modelo para o negócio.

Um sistema antifraude precisa equilibrar:

- fraude detectada;
- fraude não detectada;
- falsos positivos;
- custo operacional;
- capacidade de investigação;
- experiência do cliente;
- impacto financeiro.

Assim:

> **A melhor métrica técnica não determina necessariamente a melhor decisão de negócio.**

---

# 20. Limitações

Este projeto possui limitações importantes.

### Dataset sintético

Os dados foram gerados artificialmente e não representam toda a complexidade existente em ambientes transacionais reais.

### Viés da regra de velocity

As fraudes foram construídas utilizando concentração de transações em pequenas janelas temporais.

Isso favorece diretamente detectores baseados em velocity.

### Baseline comportamental

Para fins didáticos, o histórico legítimo conhecido foi utilizado na construção do baseline dos clientes.

Em produção, features comportamentais devem utilizar somente informações disponíveis antes da transação analisada, evitando **temporal leakage**.

### Avaliação do Isolation Forest

O Isolation Forest foi avaliado sobre o mesmo conjunto utilizado durante o `fit`.

Uma implementação mais próxima de produção exigiria estratégia de validação temporal e avaliação fora da amostra.

### Custos hipotéticos

Os custos de FP e FN utilizados são parâmetros experimentais.

Em ambiente real devem ser estimados utilizando dados financeiros, operacionais e de experiência do cliente.

### Fraude perdida como proxy financeira

O valor integral das transações fraudulentas não detectadas foi utilizado como proxy de exposição financeira.

Na prática, perda líquida pode envolver recuperação de valores, chargebacks, custos de investigação e outros componentes.

---

# 21. Próximos passos

Possíveis evoluções incluem:

- validação temporal;
- backtesting;
- tuning do `contamination`;
- análise contínua do anomaly score;
- thresholds orientados a custo;
- segmentação de risco;
- autenticação adaptativa;
- supervised learning;
- combinação entre score supervisionado e anomaly score;
- monitoramento de concept drift;
- avaliação de capacidade operacional da fila de alertas.

E existe uma evolução particularmente interessante.

## Graph Intelligence

Modelos tabulares analisam principalmente características da própria transação e do comportamento histórico conhecido.

Fraudes coordenadas, entretanto, podem aparecer através de **relações**.

Exemplos:

`Customer → Card → Transaction → Device → Merchant`

ou:

`Customer → Device → IP → Account → Transaction`

Uma transação isoladamente pode parecer normal.

Mas seu relacionamento com outras entidades pode revelar:

- dispositivos compartilhados;
- contas conectadas;
- comunidades suspeitas;
- ataques coordenados;
- estruturas de fraude;
- padrões relacionais invisíveis em uma análise puramente tabular.

Uma evolução futura deste projeto poderá explorar **Neo4j, Graph Data Science e Graph Intelligence** para investigar essa camada relacional.

---

# Final Insight

Este projeto reforçou três princípios:

> **Anomalia não é sinônimo de fraude.**

> **Maior Recall não significa automaticamente melhor decisão.**

> **Fraud Analytics não termina na previsão — termina na decisão que o negócio consegue tomar a partir dela.**

---

## Autor

**Alexandre Lotti <!LOTTI>**

Fraud Analytics | Data Science | Graph Intelligence | Neo4j | Python | SQL | Cypher | AML | PLD/FT