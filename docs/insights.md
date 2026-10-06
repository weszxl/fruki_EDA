- 'dim_' indica um modelo de Data Warehouse. São tabelas de dimensão. Elas não costumam registrar transações de vendas, mas sim os atributos e cadastros que dão contexto a essas vendas.

- Cliente + Hierarquia: A tabela dim_cliente se liga à dim_hierarquia_comercial usando a coluna id_hierarquia (no cliente) com a ID_HIERARQUIA (na comercial). Isso diz quem atende qual cliente.

- Preço + Item: A tabela dim_tabela_preco se conecta à dim_item através do ID_ITEM. Isso mapeia qual produto custa quanto.

- Cliente + Preço: A tabela dim_cliente aponta para a ID_TABELA_PRECO. Isso indica qual regra de preço se aplica a cada cliente.

- SEM DADOS de transação de venda