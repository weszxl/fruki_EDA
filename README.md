# Fruki EDA e Estruturação de Dados para o Supabase

## Contexto do Projeto

Este repositório contém o pipeline em Python para exploração, saneamento, transformação e geração de dados a partir das tabelas de Data Warehouse da Fruki. O objetivo do processamento é preparar e exportar os dados de cadastro e de histórico de vendas para o banco de dados no Supabase (PostgreSQL), que fornece a base de consulta para a API em Java e para a geração de mensagens via LLM.

O processamento dos dados ocorre em quatro etapas:
1. Exploração (`eda/data_visuali.ipynb`): leitura das tabelas de origem, verificação de esquemas, contagem de linhas e identificação de códigos de status e de colunas sem uso no cálculo de recomendação.
2. Limpeza (`eda/data_cleanning.ipynb`): remoção de colunas de controle do sistema, filtragem de clientes em atividade e sem suspensão de crédito (2.861 registros), filtragem de itens em comercialização sob a flag `Obs_1512816538` (201 registros) e filtragem de tabelas de preço em vigor (24.642 registros).
3. Transformação (`eda/transform_data.ipynb`): conversão de códigos em nomes de produtos, segmentos, subsegmentos, vendedores e municípios do Rio Grande do Sul e de Santa Catarina (48 cidades), com distribuição de tabelas de preço por região (Litoral, Serra, Região Metropolitana e Interior).
4. Geração de dados de simulação (`eda/mock_data.ipynb`): geração de CNPJ, telefone, datas de compra e histórico de pedidos e itens para compor os arquivos CSV de carga no Supabase.

---

## Estrutura do Projeto

```text
/fruki_EDA
├── docs/
│   ├── insights.md                              # Apontamentos sobre as tabelas de origem e remoção de colunas
│   └── notes.md                                 # Registro do saneamento de dados e regras de negócio
├── eda/
│   ├── data/                                    # Arquivos CSV para importação no Supabase
│   │   ├── cidade.csv                           # Cadastro de 48 municípios (RS e SC)
│   │   ├── cliente.csv                          # Cadastro de 2.861 pontos de venda B2B
│   │   ├── hierarquia.csv                       # Cadastro de representantes e vendedores (500 registros)
│   │   ├── item_pedido.csv                      # Linhas de itens por pedido (29.972 registros)
│   │   ├── pedido.csv                           # Cabeçalho do histórico de pedidos (9.999 registros)
│   │   └── produto.csv                          # Catálogo de 201 itens da Fruki
│   ├── data_/                                   # Diretório de processamento (fora do controle do Git)
│   │   ├── fruki_data_base/                     # Tabelas de origem do Data Warehouse
│   │   ├── clean_fruki_data/                    # Tabelas após corte de colunas e filtragem de registros
│   │   ├── data_format/                         # Tabelas em etapa de formatação
│   │   └── data_fruki_final/                    # Tabelas com mapeamento de nomes e regiões
│   ├── data_visuali.ipynb                       # Inspeção das tabelas de origem
│   ├── data_cleanning.ipynb                     # Corte de colunas e filtragem de registros
│   ├── transform_data.ipynb                     # Mapeamento de códigos para nomes e geolocalização
│   └── mock_data.ipynb                          # Geração de atributos e dados de simulação
├── .gitignore                                   # Regras de exclusão de arquivos do Git
├── README.md                                    # Documentação do projeto
└── requirements.txt                             # Dependências do projeto em Python
```

---

## Banco de Dados (Supabase) e Integração com Java

Estrutura de tabelas no Supabase para integração com a aplicação em Java.

### 1. Entidades

#### A. Entidades de Cadastro
- **Hierarquia** (`id_hierarquia`, `cod_representante`, `nome_vendedor`): armazena os representantes e vendedores. Identifica qual vendedor atende cada cliente e direciona os alertas para o aplicativo.
- **Cidade** (`id_cidade`, `nome`, `uf`): armazena os 48 municípios que o `ContextoService` utiliza para consulta em APIs de terceiros.
- **Cliente** (`id_cliente`, `cnpj`, `nome`, `segmento`, `subsegmento`, `numero_celular`, `cep`, `ultima_compra`, `id_hierarquia`, `id_cidade`): cadastro dos 2.861 pontos de venda B2B. A coluna `cnpj` identifica o registro nas buscas do banco. As colunas `segmento` e `subsegmento` indicam o ramo de comércio para o cálculo de pontuação (`scores`). A coluna `ultima_compra` funciona como atalho para a API verificar clientes sem compras no período sem percorrer a tabela de pedidos a cada verificação, reduzindo custo e processamento.
- **Produto** (`id_item`, `nome`, `cod_familia`, `preco_base`, `preco_minimo`, `ativo`): catálogo de 201 itens da Fruki. A coluna `ativo` indica se o produto está à venda. As colunas `preco_base` e `preco_minimo` definem os limites de valor para negociações no varejo e no atacado e para sugestão de promoções.
- **Produto_Complementar** (`produto_id`, `complementar_id`): tabela de associação que substitui a lista em cache (`List<Long> complementaresIds`). Armazena os pares de cruzamento de vendas (`cross-sell`) que o algoritmo identifica para leitura pela API, reduzindo processamento e custo.

#### B. Entidades de Transação e Histórico
- **Pedido** (`id_pedido`, `data_pedido`, `id_cliente`): cabeçalho das vendas com vínculo à entidade `Cliente`. Registra quem comprou e em qual data.
- **Item_Pedido** (`pedido_id`, `produto_id`, `quantidade`, `valor_unitario`): armazena as linhas de cada compra (dados de simulação). Serve de base para calcular a frequência de compra, o volume por segmento e o valor de venda em cada operação.

#### C. Entidades de Regras e Inteligência (LLM e Score)
- **Promocao** (`id_promocao`, `nome`, `produto_id`, `categoria`, `tipo_desconto`, `valor_desconto`, `inicio`, `fim`, `ativa`): registra campanhas de vendas e promoções por período.
- **Notificacao** (`id_notificacao`, `cliente_id`, `tipo`, `mensagem`, `canal`, `status`, `criada_em`): registra as mensagens do LLM e o estado de entrega.

#### D. Entidade de Estoque (Simulação de ERP)
- **Estoque_ERP** (`produto_id`, `quantidade_disponivel`, `ultima_atualizacao`): isola a disponibilidade dos itens e simula a consulta ao sistema ERP.

### 2. Relações e Cardinalidade

| Tabela Origem | Coluna (FK) | Tabela Destino | Cardinalidade | Descrição |
| :--- | :--- | :--- | :--- | :--- |
| `Cliente` | `id_hierarquia` | `Hierarquia` | N : 1 | Liga clientes a um vendedor. |
| `Cliente` | `id_cidade` | `Cidade` | N : 1 | Vincula clientes a uma cidade. |
| `Pedido` | `id_cliente` | `Cliente` | N : 1 | Conecta o histórico de pedidos ao cliente que realizou a compra. |
| `Item_Pedido` | `pedido_id` | `Pedido` | N : 1 | Agrupa os itens de compra dentro de um pedido. |
| `Item_Pedido` | `produto_id` | `Produto` | N : 1 | Identifica qual produto do catálogo consta no item do pedido. |
| `Produto_Complementar` | `produto_id`, `complementar_id` | `Produto` | N : N | Liga um produto aos produtos comprados em conjunto com ele. |
| `Promocao` | `produto_id` | `Produto` | N : 1 | Aplica uma campanha de desconto a um produto. |
| `Notificacao` | `cliente_id` | `Cliente` | N : 1 | Direciona a mensagem para o cliente de destino. |
| `Estoque_ERP` | `produto_id` | `Produto` | 1 : 1 | Define o saldo de estoque de cada produto. |

### 3. Comportamento dos Dados e Fluxos da Aplicação

#### Coerência dos Dados de Simulação e Tabelas sem Carga Inicial
- Os dados de transação seguem as regras das planilhas: clientes do segmento `Atacado` possuem pedidos de 50 a 500 unidades com `valor_unitario` na faixa do `preco_minimo`, enquanto clientes de `Varejo` possuem pedidos de 2 a 40 unidades na faixa do `preco_base`.
- As tabelas `Produto_Complementar`, `Promocao` e `Notificacao` iniciam sem registros no banco porque não representam cadastros ou histórico, mas o resultado da execução da API.

#### Funcionamento de `Produto_Complementar` (Cross-sell)
- A API ou algoritmo analisa periodicamente a tabela `Item_Pedido`, identifica quais itens saem juntos e grava os pares na tabela `Produto_Complementar`.
- No momento da venda, a aplicação em Java realiza a leitura nessa tabela para sugerir o item de complemento com redução de processamento.

#### Funcionamento de `Promocao`
- Leitura no cálculo de recomendação: a API consulta a tabela durante o cálculo para aumentar a pontuação (`score`) dos produtos em oferta.
- Disparo de evento: a inserção de uma linha aciona um evento no sistema informando que existem clientes que precisam receber notificação sobre o desconto.

#### Funcionamento de `Notificacao`
Atua como fila de saída do LLM em conjunto com as tabelas `Cliente` e `Promocao`:
- **Fluxo de Promoção (`Promocao` + `Notificacao`)**: ao detectar uma promoção, o sistema monta a mensagem para os clientes que atendem aos critérios e grava o registro na tabela `Notificacao`.
- **Fluxo de Recuperação (`Cliente` + `Notificacao`)**: o sistema monitora a coluna `ultima_compra` da tabela `Cliente`. Quando um CNPJ ultrapassa o intervalo de compra sem realizar pedidos, a API gera o argumento de venda e grava o registro na tabela `Notificacao` para envio via WhatsApp ou alerta no aplicativo.

#### Cálculo de Pontuação (Scores) no Supabase
- Com cerca de 30.000 registros em `Item_Pedido`, transferir os dados pela rede para a memória da aplicação em Java a cada requisição aumenta o tempo de resposta da API.
- Cálculos como frequência de compra do cliente e pontuação por região podem ser executados no Supabase para reduzir o tráfego de dados e o uso de memória na API.

#### Filtro de Estoque (`Estoque_ERP`)
- Antes de concluir o ranking de recomendação ou enviar uma mensagem, a aplicação cruza os identificadores dos produtos da recomendação com a tabela `Estoque_ERP` para validar a disponibilidade em estoque.

---

## Pré-requisitos

- Python 3.10 ou superior
- `pip` (gerenciador de pacotes do Python)
- Jupyter Notebook ou extensão Jupyter no editor de código
- Conta e instância de banco de dados no Supabase (PostgreSQL) para carga dos arquivos CSV

---

## Instruções de Execução

### 1. Configurar o ambiente Python

Na raiz do projeto (`/fruki_EDA`), crie o ambiente de execução:

```bash
python -m venv .venv
```

Ative o ambiente:

- **Linux ou macOS:**
  ```bash
  source .venv/bin/activate
  ```
- **Windows:**
  ```bash
  .venv\Scripts\activate
  ```

### 2. Instalar as dependências

```bash
pip install -r requirements.txt
```
