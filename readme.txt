# Modelo de Credit Scoring 
Pacote de entrega contendo o pipeline analítico completo de modelagem estatística. O projeto contempla desde o diagnóstico exploratório e tratamento de dados sem contaminação temporal (*data leakage*) até o treinamento da Regressão Logística, validação *Out-of-Time* (OOT) e agrupamento ótimo de *ratings* de risco via CART para suporte à esteira de concessão.

---

## 1. Como Reproduzir o Ambiente

Este projeto utiliza o gerenciador de dependências [`renv`](https://rstudio.github.io/renv/) para garantir o isolamento e a reprodutibilidade exata das bibliotecas do R utilizadas.

1. Extraia o arquivo compactado (`.zip`) em uma pasta de sua preferência.
2. Abra o arquivo de projeto no RStudio dando um duplo clique em:
   ```text
   case-agibank.Rproj
   ```
   *(Ao abrir o arquivo, o script `.Rprofile` iniciará automaticamente o ambiente isolado do `renv`).*
3. No console do R, execute o comando para restaurar as dependências e versões registradas no arquivo `renv.lock`:
   ```r
   renv::restore()
   ```

---

## 2. Estrutura do Diretório

```text
sua-pasta/
├── data/
│   ├── raw/         # Base bruta original (concessoes_cp.csv)
│   ├── trusted/     # Feature stores tratadas (treino e teste OOT)
│   └── delivery/    # Base final pontuada (scores preditos e ratings A-E)
├── model/           # Objeto serializado do modelo final (.rds)
├── renv/            # Configurações e scripts de inicialização do renv
├── reports/         # Relatórios executivos completos compilados (PDF / HTML)
├── scripts/         # Scripts R com a esteira analítica sequencial
├── .Rprofile        # Script de inicialização automática do renv
├── case-agibank.Rproj
├── renv.lock        # Trava de dependências e versões dos pacotes R
└── README.md
```

---

## 3. Ordem de Execução dos Scripts

Os scripts dentro da pasta `scripts/` estão estruturados para execução sequencial:

* **Passo 1: `01_eda.R` (ou `.Rmd`)**
  * Realiza o levantamento de valores ausentes, inspeção de assimetria e *outliers* via *boxplots*, matriz de correlação de Spearman e cálculo de *Information Value* (IV) para triagem inicial de atributos.
* **Passo 2: `02_data_prep.R` (ou `.Rmd`)**
  * Aplica regras de expurgo para blindagem contra vazamento temporal (*leakage*), imputação de nulos pela mediana de cada safra e divisão temporal estrita (Treino: Jan-Set/2016; Teste OOT: Out-Dez/2016).
  * Executa o *capping* de valores extremos no percentil 99 com base na amostra de treino, realiza *Target Encoding* bayesiano em `addr_state` e *One-Hot Encoding* para atributos categóricos, gravando as bases limpas em `data/trusted/`.
* **Passo 3: `03_model.R` (ou `.Rmd`)**
  * Ajusta a Regressão Logística binomial, computa as métricas de ordenação e calibração no teste cego OOT, monitora a estabilidade mensal de safras e deriva a segmentação ótima de *ratings* (A a E) via árvore de decisão CART.
  * Salva o artefato do modelo treinado em `model/` e grava o arquivo com as predições finais em `data/delivery/base_escorada_oot.csv`.
