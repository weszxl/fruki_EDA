# As quatro planilhas representam a estrutura de um Data Warehouse voltado para a distribuição de bebidas. Elas contêm os cadastros estruturais que sustentam a operação comercial:

- Clientes (dim_cliente): Cadastro físico e comercial, contendo localização, segmento de atuação, perfil de crédito e a tabela de preço vinculada.

- Hierarquia Comercial (dim_hierarquia_comercial): A estrutura da força de vendas, mapeando a cadeia de representantes, supervisores e gerentes por rota.

- Itens (dim_item): O catálogo do portfólio, detalhando características físicas dos produtos, como sabor, tipo de embalagem, litragem e família (ex: Linha PET).

- Preços (dim_tabela_preco): As marcações de precificação, definindo tetos mínimos e preços de venda para os itens ativos.

# Colunas inúteis

- dim_client: COD_CLIENTE, cod_cadeia, cod_segmento, COD_SUBSEGMENTO, COD_SITUACAO_CREDITO, COD_GRUPO_ECONOMICO

- dim_hierarquia_comercial.csv: VERSION, DATE_FROM, DATE_TO, CHAVE, COD_GERENTE, COD_GERENTE_REGIONAL, COD_SUPERVISOR, COD_CV, GERENTE, GRUPO_GERECIA, CV, GERENTE_REGIONAL, SUPERVISOR, COD_NOME_GERENTE, COD_NOME_GERENTE_REGIONAL, COD_NOME_SUPERVISOR, COD_NOME_REPRESENTANTE,

- dim_item: VERSION, DATE_FROM, DATE_TO, DATA_IMPLANTACAO, COD_ITEM, COD_FAMILIA, COD_TROCA, COD_MARCA, COD_GRUPO_ESTOQUE,ITEM, COD_DESCRICAO, FAMILIA, MARCA, MARCA_ANALISE, NOME_REDUZIDO, GRUPO_ESTOQUE, SABOR_TIPO, UN, TIPO, ORIGEM, UNID_PT, CLASSIFICAO_ABC, GRUPO_ATIVO, ADESIVO, COD_VASILHAME_CAIXA, QTD_VASILHAME_CAIXA, COD_VASILHAME, QTD_VASILHAME, FT_LITROS, FT_PALETE, FT_CHAPA, META_ATIVO

- dim_tabela_preco: ID_TABELA_PRECO, VERSION, DATE_FROM, DATE_TO, DATA_VALIDADE, COD_ITEM, 