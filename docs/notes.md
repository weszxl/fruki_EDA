# Notas sobre as tabelas de origem e o saneamento de dados

## Estrutura das tabelas de origem

O prefixo `dim_` indica um modelo de Data Warehouse. São tabelas de dimensão: em geral não registram transações de venda, e sim os atributos e cadastros que dão contexto a essas vendas.

- **Cliente + Hierarquia:** `dim_cliente` se liga a `dim_hierarquia_comercial` pela coluna `id_hierarquia` (cliente) com `ID_HIERARQUIA` (comercial). Indica quem atende cada cliente.
- **Preço + Item:** `dim_tabela_preco` se conecta a `dim_item` pelo `ID_ITEM`. Mapeia qual produto custa quanto.
- **Cliente + Preço:** `dim_cliente` aponta para `ID_TABELA_PRECO`. Indica qual regra de preço se aplica a cada cliente.

## Limitações dos dados

- **Sem dados de transação de venda.** Não há histórico de pedidos nas planilhas; o histórico (Pedido e Item_Pedido) é simulado.
- **Base de clientes majoritariamente inativa.** São 24.565 clientes inativos e 2.925 ativos. Após o filtro de suspensão de crédito, restam 2.861 clientes. As simulações devem considerar apenas perfis ativos, pois clientes inativos não têm utilidade para a API.
- **Precificação complexa.** Há 374.131 preços diferentes para apenas 262 produtos, ou seja, mais de 1.400 regras de preço por item. O preço não é fixo por produto: varia conforme tabela de preço, região e cliente. No Supabase isso foi resumido em `preco_base` e `preco_minimo` por produto, com a variação regional representada pelo `multiplicador` da tabela `Cidade`.
- **Preço por região.** Regiões como litoral, serra e centro recebiam valores diferentes para os mesmos produtos. Padronizar os nomes das regiões usados nos documentos (Litoral, Serra, Região Metropolitana e Interior).
- **Vendedores mudam de carteira.** Como o modelo de venda é pré-compra, esse movimento tem baixo impacto na solução.
- **Vendedores órfãos e regiões sem responsável.** Há vendedores órfãos e regiões não repassadas. Antes de criar a chave estrangeira `cliente.id_hierarquia`, verificar valores nulos ou sem correspondência em `Hierarquia`.
- **CEPs inválidos.** Parte dos CEPs é inválida. Latitude e longitude podem ser aproveitadas após correção. **os CEPs foram tratados no `cliente.csv` (foram corrigidos ou simulados).]**

## Regras de negócio e decisões

- **Filtrar cedo e com frequência (Filter Early, Filter Often).** Reduzir as tabelas logo nas primeiras etapas (clientes ativos, itens em comercialização, tabelas de preço em vigor).
- **IDs de 1 a 30.000** são apenas a ordem cronológica do sistema, sem significado de negócio.
- **O catálogo de produtos funciona como o "RG" do produto,** já que o valor é variável. Serve para a base de visualização, e o ERP consulta o ID para saber qual produto é.
