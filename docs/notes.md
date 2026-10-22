- 'dim_' indica um modelo de Data Warehouse. São tabelas de dimensão. Elas não costumam registrar transações de vendas, mas sim os atributos e cadastros que dão contexto a essas vendas.

- Cliente + Hierarquia: A tabela dim_cliente se liga à dim_hierarquia_comercial usando a coluna id_hierarquia (no cliente) com a ID_HIERARQUIA (na comercial). Isso diz quem atende qual cliente.

- Preço + Item: A tabela dim_tabela_preco se conecta à dim_item através do ID_ITEM. Isso mapeia qual produto custa quanto.

- Cliente + Preço: A tabela dim_cliente aponta para a ID_TABELA_PRECO. Isso indica qual regra de preço se aplica a cada cliente.

- SEM DADOS de transação de venda

- Não tem histórico nas planilhas.

- A base de clientes é na maior prte inútil (24.565 estão "Inativos" e 2.925 "Ativos").
- - as simulações devem ser feitas considerando os perfis ativos, clientes inativos são inúteis pra API;

- Precificação sem sentido (T-T), com 374.131  preços diferentes para apenas 262 produtos. (?????????)
- - O preço não é fixo por produto. Há uma média, mais de 1.400 regras de preço para cada item do catálogo Os preços variam dependendo dos fatores.

* Vendedores se movem, que nem as moças do jequiti
* * meio que fodase, pq o modelo de venda é pré-compra

- Filtre cedo, filtre com frequência (Filter Early, Filter Often)

- ceps invalidos (apontam pro ira)
- lat/long valido se arrumar

- ID's de 1 a 30000 são apenas cronologia do sistema
- - o catalogo deve ser estritamente o "RG" do produto, visto que o valor é variavel
- - servirá para a base di visualização, ERP vai consultar o id para ver que produto é aquele

- regiões como liotral, centro ou serra recebiam valores diferentes para os produtos

- há vendedores orfãos / regiões não repassadas


-----------------------

# Relatório de Saneamento e Estruturação de Dados (MVP Fruki)

O Data Warehouse original fornecido possuía excesso de metadados logísticos e histórico defasado, sendo inadequado para consumo direto por uma API de recomendação em tempo real. A base foi reduzida estritamente ao necessário para a regra de negócio do motor e a geração de contexto para o LLM.

---

## 1. Diagnóstico do Modelo
A base de dados é puramente dimensional (estática). **Não há tabela de fatos** (transações de vendas). A solução arquitetural definida foi estabilizar as dimensões reais e gerar um script que simule o comportamento de compra (mock de vendas e estoque) plantando padrões matemáticos (sazonalidade, ciclo de recompra, cruzamento segmento-produto).

---

## 2. Otimização das Entidades (Cortes e Filtros)

### 👥 Clientes (`dim_cliente`)
* **Estrutura:** Colunas reduzidas de **24 para as essenciais** (IDs de cruzamento, Estado, Cidade, Bairro e Segmento).
* **Filtro:** Eliminados clientes inativos ou com crédito suspenso. O motor não processará dados de CNPJs bloqueados para faturamento.

### 📦 Catálogo (`dim_item`)
* **Estrutura:** Colunas reduzidas de **41 para 7**. Todo o lixo de chão de fábrica (peso de palete, regras de vasilhame retornável) foi descartado.
* **Filtro:** Identificado que o código anonimizado `Obs_1512816538` representa a flag de produto ativo. A base foi filtrada para manter apenas estes **201 produtos reais**, impedindo a recomendação de itens fora de linha.

### 💰 Preços (`dim_tabela_preco`)
* **Estrutura:** Reduzida para **4 colunas vitais** (Tabela, Item, Preço de Venda e Preço Mínimo). O preço mínimo foi mantido intencionalmente para servir de insumo de negociação para o LLM.
* **Filtro (Ganho de Performance):** A exclusão dos preços com flag "Inativo" removeu quase **350 mil linhas de histórico morto**. Restaram apenas **~24.000 ligações vigentes**, garantindo que o aplicativo não exiba valores defasados e aliviando a memória do banco de dados.

### 👔 Força de Vendas (`dim_hierarquia_comercial`)
* **Estrutura:** Colunas reduzidas de **26 para 3**. A estrutura de chefia (supervisores, gerentes) e geografia de rota foi descartada. Manteve-se apenas a ligação direta entre o ID da Rota e o Vendedor logado.
* **Auditoria:** Confirmado que **não existem rotas "órfãs"** (0 valores nulos). Todo ID possui um vendedor atrelado.

---

## 3. Insight Crítico: O Bloqueio do Mascaramento
Os atributos de texto mais importantes para a Inteligência Artificial redigir mensagens persuasivas estão anonimizados (ex: `Sabor_2048964324`, `Seg_2137998953`). **Um LLM não consegue operar com códigos mascarados.** 

Para a banca de apresentação, será **obrigatório** criar um "De/Para" manual, substituindo as tags por nomes reais (ex: "Guaraná", "Restaurante", "Garrafa PET 2L") em uma amostra controlada de produtos e clientes.

---

## 4. Direcionamento
Com as dimensões estáticas limpas e prontas, o projeto tem duas dependências técnicas para fechar o back-end:

1. **Criação do Banco de Dados (PostgreSQL):** Subir os quatro arquivos `.csv` finais utilizando comandos DDL (`CREATE TABLE`) com as chaves primárias e estrangeiras mapeadas.
2. **Motor de Simulação (Python):** Desenvolver o script de mock para gerar as tabelas virtuais de `Estoque` e `Fato_Vendas`, inserindo os vieses comportamentais necessários para o algoritmo ter o que pontuar.
