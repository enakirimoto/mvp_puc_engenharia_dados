
# MVP de Análise de Dados: Impacto das Redes Sociais na Saúde Mental e Bem-Estar

## Identificação

**Aluno:** Eric Koji Nakirimoto 

**Matricula:** 4052025000088

**Disciplina:** Sprint: Engenharia de Dados 2025_02 


## 1. Objetivo do Trabalho

### Problema a ser resolvido
O uso excessivo de redes sociais e o tempo de tela têm sido associados a diversos problemas de saúde mental na sociedade moderna. Este projeto visa quantificar e analisar se existem correlações estatísticas claras entre hábitos digitais (tempo de tela) e indicadores de bem-estar (sono, felicidade, estresse e exercícios) em uma amostra populacional.

### Perguntas para orientar a análise
Para guiar a análise, foram definidas as seguintes perguntas norteadoras:
1.  **Redes Sociais x Estresse:** Existe uma correlação positiva entre o tempo gasto em redes sociais e o nível de estresse reportado?
2.  **Redes Sociais x Felicidade:** O uso intensivo de redes sociais afeta negativamente o índice de felicidade?
3.  **Redes Sociais x Sono:** O tempo de tela impacta a qualidade ou duração do sono?
4.  **Sono, Felicidade e Exercícios:** Existe uma correlação entre qualidade do sono, frequência de exercícios e o índice de felicidade geral?
OBS: Minha esposa vive me incomodando por ficar usando celular antes de dormir. Quero ver se ela está realmente certa. 

---

## 2. Busca e Coleta de Dados

### Fonte de Dados
* **Nome:** Social Media and Mental Health Balance
* **Origem:** Kaggle
* **URL:** [Link para o Dataset](https://www.kaggle.com/datasets/ayeshaimran123/social-media-and-mental-health-balance/data)
* **Formato:** Arquivo plano (CSV).
* **Licença:** Open Data Commons Public Domain Dedication and License (PDDL) v1.0

### Processo de Coleta (Databricks)
1.  **Download:** O arquivo CSV foi baixado da fonte original.
2.  **Upload:** O arquivo foi carregado para o DBFS (Databricks File System) utilizando a interface de upload do Databricks Community Edition.
3.  **Caminho no DBFS:** `/FileStore/tables/social_media_mental_health.csv` (Exemplo).

---

## 3. Modelagem de Dados

Para este MVP, utilizaremos uma arquitetura de **Data Lake** seguindo o padrão **Medallion Architecture** (Bronze, Silver, Gold), que é nativo e otimizado para o Databricks.

### Esquema Proposto (Camadas)

1.  **Camada Bronze (Raw):** Dados brutos ingeridos diretamente do CSV, sem tratamento, salvos em formato Delta.
2.  **Camada Silver (Cleaned/Conformed):**
    * Tratamento de nulos.
    * Renomeação de colunas (tradução para PT-BR ou padronização `snake_case`).
    * Conversão de tipos de dados (String para Decimal/Integer).
3.  **Camada Gold (Aggregated/Business Level):** Tabela analítica final pronta para responder às perguntas de negócio (ex: `analise_bem_estar_social`).

### Catálogo de Dados (Dicionário)


| Nome Original | Nome Tratado (Na camada Silver) | Tipo de Dado | Descrição/Domínio | Valores Esperados |
| :--- | :--- | :--- | :--- | :--- |
| `User_ID` | `id_usuario` | String | Identificador único do participante da pesquisa. | Alfanumérico (ex: IDs anonimizados). |
| `Age` | `idade` | Integer | Idade do participante em anos. | Numérico inteiro (ex: 18 a 60+). |
| `Gender` | `genero` | String | Gênero com o qual o participante se identifica. | Categorias: 'Male', 'Female', 'Non-binary', etc. |
| `Daily_Screen_Time` | `tempo_tela_diario` | Decimal | Tempo médio diário gasto em frente a telas (em horas). | 0.0 a 24.0 horas. |
| `Sleep_Quality` | `qualidade_sono` | Decimal | Avaliação subjetiva da qualidade do sono em uma escala. | 1.0 a 10.0 (onde 10 é excelente). |
| `Stress_Level` | `nivel_estresse` | Decimal | Nível de estresse percebido pelo participante em uma escala. | 1.0 a 10.0 (onde 10 é estresse extremo). |
| `Days_Without_Social_Media` | `dias_sem_redes` | Decimal | Número de dias que a pessoa consegue ou ficou sem usar redes sociais (detox digital). | 0 a valores positivos (dias). |
| `Exercise_Frequency` | `freq_exercicio` | Decimal | Frequência de exercícios físicos realizados por semana. | 0 a 7 (dias por semana) ou valor normalizado. |
| `Social_Media_Platform` | `plataforma_rede_social` | String | Principal plataforma de rede social utilizada pelo participante. | Categorias: 'Instagram', 'TikTok', 'Facebook', 'Twitter/X', etc. |
| `Happiness_Index` | `indice_felicidade` | Decimal | Índice subjetivo de felicidade e bem-estar geral em uma escala. | 1.0 a 10.0 (onde 10 é muito feliz). |

### Linhagem dos Dados
* **Origem:** Kaggle CSV.
* **Processo:** `CSV` -> `DataFrame Spark (Leitura)` -> `Tabela Delta Bronze` -> `Limpeza/Tratamento` -> `Tabela Delta Silver` -> `Análise/Correlação` -> `Tabela Delta Gold`.

---

## 4. Carga e Transformação (ETL)

O processo de ETL será realizado utilizando **PySpark** no Databricks.

### Etapas do Pipeline:
1.  **Extração:** Leitura do arquivo CSV do DBFS com inferência de schema.
2.  **Carga Bronze:** Gravação do DataFrame bruto em formato `delta` no caminho `/dbfs/mnt/bronze/social_health`.
3.  **Transformação (Bronze -> Silver):**
    * Remoção de registros duplicados.
    * Tratamento de *outliers* (ex: tempo de tela > 24h).
    * Padronização dos nomes das colunas.
4.  **Carga Silver:** Gravação dos dados tratados em formato `delta` no caminho `/dbfs/mnt/silver/social_health`.

---

## 5. Análise de Dados

### A. Análise de Qualidade de Dados
Verificação da integridade dos dados antes de responder às perguntas de negócio. Serão analisados:
* **Contagem de Nulos:** Verificar se há campos vazios críticos (ex: Felicidade ou Tempo de Tela).
* **Verificação de Range:** Garantir que `tempo_tela` não seja negativo ou maior que 24h. Garantir que índices de 0 a 10 estejam dentro do limite.
* **Cardinalidade:** Verificar se os campos categóricos (como frequência de exercício) possuem valores padronizados.

### B. Solução do Problema (Insights)
Utilizaremos **Pandas, Matplotlib/Seaborn** (após converter o Spark DF para Pandas nas agregações) para gerar visualizações.

1.  **Correlação Geral (Heatmap):** Matriz de correlação de Pearson entre todas as variáveis numéricas (Tempo de tela, Sono, Estresse, Felicidade).
    * *Objetivo:* Responder de uma só vez quais variáveis estão mais fortemente ligadas.
2.  **Scatter Plot com Linha de Tendência:** `Tempo de Tela` (eixo X) vs `Nível de Estresse` (eixo Y).
    * *Resposta à Pergunta 1.*
3.  **Boxplot:** `Frequência de Exercício` vs `Índice de Felicidade`.
    * *Resposta à Pergunta 4 (parcial).*
4.  **Análise de Distribuição:** Histogramas para entender o perfil da amostra (ex: a maioria das pessoas dorme pouco ou muito?).

---

## 6. Autoavaliação e Conclusão

### Discussão dos Resultados
* (Espaço reservado para descrever se as hipóteses foram confirmadas. Ex: "Foi observado uma correlação forte de 0.7 entre tempo de tela e estresse...").

### Dificuldades Encontradas
* (Exemplo: "Dificuldade em limpar dados categóricos que estavam despadronizados" ou "Limitação de memória do cluster Community ao gerar gráficos pesados").

### Trabalhos Futuros
* Expandir a análise cruzando com dados demográficos (idade, localização) se disponíveis.
* Implementar um modelo de Machine Learning (Regressão) para prever o nível de felicidade com base nos hábitos digitais.

---
