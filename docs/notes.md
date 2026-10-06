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

- 
