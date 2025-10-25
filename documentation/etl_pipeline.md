# Documentação Técnica: Parte 2 - Pipeline de ETL

Este documento detalha o processo de **Extract, Transform, Load (ETL)** aplicado aos 5 arquivos brutos ("sujos") do projeto, localizados em `data/raw_data/`.

O objetivo deste pipeline é limpar, padronizar, tratar tipos de dados e unificar as fontes para criar arquivos "limpos" (camada `processed`), prontos para a carga no Data Mart (MYSQL).

O pipeline é executado de forma modular, com um notebook de processamento para cada fonte.

**Para visualizar os dados brutos (`raw`) e processados (`processed`) deste projeto, acesse o Google Drive:**
* **[Link para os Dados (Google Drive)](https://drive.google.com/drive/folders/1wcNNKqTksXHEjQkV3sZkzLMz_L50wQAT?usp=sharing)**

---

## 1. Processamento: `compra_produtos.csv`

* **Notebook:** `processing/processing_compra_produtos.ipynb`
* **Fonte (RAW):** `data/raw_data/compra_produtos.csv`
* **Saída (Processed):** `data/processed_data/compra_produtos.parquet`

### Desafios:
Este arquivo, vindo do "sistema de compras", era complexo e apresentava 6 desafios principais de limpeza:

1.  **Leitura (Encoding/Sep):** O arquivo não era um CSV padrão (`utf-8` com vírgula), exigindo o uso dos parâmetros `sep=';'` e `encoding='ansi'`.

2.  **Nulos em Chaves:** Linhas com `compra_id` ou `pedido_id` nulos foram encontradas, o que corrompe a integridade referencial dos dados.

3.  **Tipos de Dados (Datas):** As 4 colunas de data (`data_pedido`, `data_recebimento`, `data_producao`, `data_validade`) eram lidas como `object` (texto) e continham formatos ambíguos e valores inválidos que precisavam ser tratados.

4.  **Tipos de Dados (Valores):** As colunas numéricas (`preco_unit_compra`, `frete`, `desconto`, `custo_total`) eram lidas como `object` (texto). Elas estavam "poluídas" com vírgula (`,`) para decimal e, em alguns casos, ponto (`.`) para milhar.

5.  **Tipos de Dados (Identificador):** A coluna `nfe` (Nota Fiscal) era lida como `float` (e não `int`) devido à presença de valores `NaN` (nulos).

6.  **Regras de Negócio (Datas):** A validação mais crítica. O risco de existirem dados *inválidos* que passariam na limpeza de tipo, como uma `data_recebimento` anterior à `data_pedido`, ou uma `data_validade` anterior à `data_producao`.

### Solução:
O notebook `processing_compra_produtos.ipynb` aplica o seguinte pipeline de limpeza:

1.  **Leitura (Extract):** O arquivo é lido com `pd.read_csv()`, especificando `sep=';'` e `encoding='ansi'`.

2.  **Tratamento de Nulos (Chaves):** As linhas com `compra_id` ou `pedido_id` nulos são **removidas** imediatamente com `df.dropna(subset=...)` para garantir a integridade.

3.  **Formatação de Datas:** Todas as 4 colunas de data são convertidas para `datetime` usando `pd.to_datetime(..., errors='coerce')`. O `errors='coerce'` foi crucial para forçar a conversão de formatos ambíguos ou "sujos", transformando erros em `NaT`.

4.  **Formatação de Valores (Float):** As colunas numéricas (`preco_unit_compra`, `custo_total`, `frete`, `desconto`) passam por uma "corrente" (`chain`) de limpeza de texto:
    * `.str.replace(",", ".")` para trocar a vírgula de decimal por ponto (padrão `float`).
    * `.str.replace(r"\.(?=.*\.)", "")` (ou similar) para remover os pontos de milhar, garantindo uma conversão limpa.
    * As colunas são convertidas para `float` com `.astype(float)`.

5.  **Tratamento de Nulos (Valores):** `frete` e `desconto`, que podiam ser nulos, tiveram seus `NaN` preenchidos com `0.0` usando `.fillna(0.0)`.

6.  **Tratamento da NFe (Int):** Foi aplicada a regra de negócio "compra sem NFe é dado lixo".
    * Primeiro, **todas as linhas** com `nfe` nula foram **removidas** com `df.dropna(subset=["nfe"], inplace=True)`.
    * *Apenas após* a remoção dos nulos, a coluna foi convertida de `float` para `int` com `.astype(int)`.

7.  **Padronização de Texto (Categóricas):** As colunas de texto (`fornecedor_nome`, `marca`, `loja`, etc.) foram padronizadas com a corrente `.str.lower().str.strip().str.replace(" ","_")` para remover inconsistências de maiúsculas ou espaços.

8.  **Validação de Regras de Negócio (Auditoria de Datas):** O script aplicou duas auditorias lógicas complexas para garantir a sanidade das datas:
    * **Regra 1:** `data_recebimento >= data_pedido`
    * **Regra 2:** `data_validade >= data_producao`
    * Para *ignorar* os `NaT` (Nulos) na validação, foi usada a máscara booleana `(A.isna()) | (B.isna()) | (B >= A)`.
    * As linhas que falharam na auditoria (`validacao_datas == False`) foram identificadas (`.index`) e removidas permanentemente com `df = df.drop(datas_erradas)`.

9.  **Carga (Load):** O DataFrame final, 100% limpo e validado, é salvo na pasta `data/processed_data/` no formato **Parquet** (`.to_parquet(..., index=False)`). O Parquet foi escolhido pela alta performance e por **preservar os dtypes** (datas continuarão datas, ints continuarão ints), o que é crucial para a próxima etapa de análise.

---

## 2. Processamento: `estoque_produtos.xlsx`

* **Notebook:** `processing/processing_estoque_produtos.ipynb`
* **Fonte (RAW):** `data/raw_data/estoque_produtos.xlsx`
* **Saída (Processed):** `data/processed_data/produtos_limpos.parquet`

*(Em desenvolvimento...)*

---

## 3. Processamento: `log_pagamentos.json`

* **Notebook:** `processing/processing_log_pagamentos.ipynb`
* **Fonte (RAW):** `data/raw_data/log_pagamentos.json`
* **Saída (Processed):** `data/processed_data/pagamentos_limpos.parquet`

*(Em desenvolvimento...)*

---

## 4. Processamento: `historico_promocoes.xml`

* **Notebook:** `processing/processing_historico_promocoes.ipynb`
* **Fonte (RAW):** `data/raw_data/historico_promocoes.txt`
* **Saída (Processed):** `data/processed_data/promocoes_limpas.parquet`

*(Em desenvolvimento...)*

---

## 5. Processamento: `precos_produtos.txt`

* **Notebook:** `processing/processing_precos_produtos.ipynb`
* **Fonte (RAW):** `data/raw_data/precos_produtos.txt`
* **Saída (Processed):** `data/processed_data/precos_produtos.parquet`

*(Em desenvolvimento...)*
