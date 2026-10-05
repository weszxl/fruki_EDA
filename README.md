## Estrutura do projeto

```text
/fruki_EDA
│
├── /docs
│   └── insights.md              #Insights obtidos após a análise dos dados
│
├── /data                        # planilhas
│
├── requirements.txt             # bibliotecas necessárias
└── fruki_eda.ipynb                # código do EDA
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