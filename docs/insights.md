# As quatro planilhas representam a estrutura de um Data Warehouse voltado para a distribuição de bebidas. Elas contêm os cadastros estruturais que sustentam a operação comercial:

- Clientes (dim_cliente): Cadastro físico e comercial, contendo localização, segmento de atuação, perfil de crédito e a tabela de preço vinculada.

- Hierarquia Comercial (dim_hierarquia_comercial): A estrutura da força de vendas, mapeando a cadeia de representantes, supervisores e gerentes por rota.

- Itens (dim_item): O catálogo do portfólio, detalhando características físicas dos produtos, como sabor, tipo de embalagem, litragem e família (ex: Linha PET).

- Preços (dim_tabela_preco): As marcações de precificação, definindo tetos mínimos e preços de venda para os itens ativos.