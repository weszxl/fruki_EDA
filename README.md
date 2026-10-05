## Estrutura do projeto

```text
/fruki_EDA
│
├── /data                        # CRIAR A ESTRUTURA '/data/fruki_data'
│   └── /fruki_data              # planilhas da fruki (MOVER AS PLANILHAS PARA CÁ)
│
├── /docs
│   └── insights.md              # insights obtidos após a análise dos dados
│
├── venv/                        # ambiente virtual
│
├── .gitignore                   # arquivos que não devem ser commitados
│
├── analyse_fruki.ipynb          # código do EDA
│
├── README.txt
│
└── requirements.txt             # bibliotecas necessárias
```

## Configuração do ambiente local

### 1. Pré-requisitos

- As extensões de **Python** e **Jupyter**

### 2. Ambiente virtual (boa prática, criar uma venv)

na raiz `/fruki_EDA`, rode:

```
python -m venv venv
```

### 3. Ativar o ambiente

- **No Windows:**
  ```
  venv\Scripts\activate
  ```
- **No Linux ou mac:**
  ```
  source venv/bin/activate
  ```

### 4. Instalando as bibliotecas

```bash
pip install -r requirements.txt
```