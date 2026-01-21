# AluraLog

Como parte da Semana 1 do Challenge BI, a AluraLog é uma empresa fictícia de logística e que necessita dos serviços de um profissional de Business Intelligence (BI).

## Contexto

A gerência da área de logística da AluraLog  está enfrentando algumas mudanças em razão do aumento da demanda dos serviços de logística no período da pandemia. Deseja-se manter a qualidade do serviço, mas para isso é necessário acompanhar constantemente as métricas do departamento para tomar as melhores decisões. 

Assim sendo, o objetivo é desenvolver um dashboard para o setor de logística com os KPIs informados pela gerência.

## KPIs

- Calcular o número de entregas feitas no prazo.
- Calcular  a quantidade de entregas que foram feitas atrasadas.
- Identificar o número de veículos disponíveis para entrega.
- Calcular o Ship to Door (S2D), que é o tempo da expedição até a chegada do produto para o cliente.
- Calcular e mostrar o índice de ocorrências no processo em cada estado.
- Calcular e mostrar o índice de ocorrências no processo em cada estado.

## Base de Dados
- Tabelas dimensão
  - [Tabela - Produtos.csv](https://github.com/albertrml/AluraLog/blob/main/dados/Tabela%20-%20Produtos.csv)
    |  categoria_produto               |  preço  |
    |----------------------------------|---------|
    |  1-agro_industria_e_comercio     |  155,00 |
    |  2-alimentos                     |  58,00  |
    |  ...                             |  ...    |
  - [Tabelas - Veículos.csv](https://github.com/albertrml/AluraLog/blob/main/dados/Tabelas%20-%20Ve%C3%ADculos.csv)
    |  ID veículos  |  Tipo    |  Status   |
    |---------------|----------|-----------|
    |  VEH01        |  carro   |	Ocupado  |
    |  VEH02        |  carro   |	Ocupado  |
    |  ...          |  ...     |  ...      |
- Tabela fato
  - [Tabelas - Estoque.csv](https://github.com/albertrml/AluraLog/blob/main/dados/Tabelas%20-%20Estoque.csv)
    |  ID Produto  |  Data atualização  |  Quantidade  |
    |--------------|--------------------|--------------|
    |  1           |  1-jan.-2019       |  432         |
    |  2           |  1-jan.-2019       |  412         |
    |  ...         |  ...               |  ...         |
  - [Tabelas - Pedidos.csv](https://raw.githubusercontent.com/albertrml/AluraLog/refs/heads/main/dados/Tabelas%20-%20Pedidos.csv)
    |  ID Pedido  |  ID Produto  |  Quantidade  |  ID Veículo  |  Status do pedido  |  Data da compra  |  Data de entrega  |  Data previsão    |  Latitude  |  Longitude  |  UF da entrega  |
    |-------------|--------------|--------------|--------------|--------------------|------------------|-------------------|-------------------|------------|-------------|-----------------|
    |  1          |  32          |  3           |  19          |  Entregue          |  1/4/21 21:15    |  22/01/2021 21:15 |  23/01/2021 21:15 |  -22.19    |  -48.79     |  SP             |
    |  2          |  50          |  9           |  42          |  Entregue          |  1/5/21 0:15     |  14/01/2021 00:15 |  25/01/2021 00:15 |  -22.19    |  -48.79     |  SP             |
    |  ...        |  ...         |  ...         |  ...         |  ...               |  ...             |  ...              |  ...              |  ...       |  ...        |  ...            |

## Procedimentos do Excel
1. Aplicação de ETL via Power Query
    1. Na Tabela - Produtos.csv
        1. Separação das colunas por primeira ocorrência do delimitador vírgula.
        2. Divisão da coluna categoria_produto por delimitador -.
        3. A primeira e a segunda coluna são renomeadas para ID Produto e Produto.
        4. Substituição dos caracteres vírgula e underline por ponto e espaço, respectivamente.
        5. Na coluna Preço, converter o dado para moeda.
        6. O preço para o registro de ID Produto igual a 52 é a mediana dos preços dos produtos cujo o nome possua móveis, pois pertencem às categorias semelhantes.
    2. Na Tabelas - Veículos.csv
        1. Separação das colunas por cada ocorrência do delimitador vírgula.
        2. Determinar a primeira linha como cabeçalho da tabela.
        3. Criar a coluna ID com associado a linha para ser usada como ponto de ligação com a Tabelas - Pedido.csv
    3. Na Tabelas - Estoque.csv
        1. Separação das colunas por cada ocorrência do delimitador vírgula.
        2. Na coluna Data atualização, remover a ocorrência de ponto.
        3. Na coluna Data atualização, converter o dado de texto para data. Como a data é pt-BR e meu power query opera em inglês, o comando para resolver via editor avançado é:
            ```
              #"Parsed Date"= Table.TransformColumns(
                #"Replaces Value",
                {
                  {
                    "Data atualização",
                    each Date.FromText(_, "pt-BR"),
                    type date
                  }
                }
              )
            ```
    4. Na Tabelas - Pedidos.csv
        1. Separação das colunas por cada ocorrência do delimitador vírgula.
        2. As datas em texto seguem dois tipos de culturas: pt-BR e en-US. A função customizada parseDateTimeMixed converte para datetime, mantendo a cultura
             ```
               //parseDateTimeMixed
               (value as text) as nullable datetime =>
               let
                   pt = try 
                           DateTime.FromText(value, "pt-BR") 
                       otherwise 
                           null,
                   dt = if pt = null then 
                           try
                               DateTime.FromText(value, "en-US") 
                           otherwise 
                               null
                       else
                           pt
               in
                   dt
             ```
       3. Para normalizar as culturas, a função customizada normalizeDateTime trata os datetime
             ```
               //normalizeDateTime
               (value as nullable datetime) as nullable datetime =>
               if value = null then 
                   null
               else
                   #datetime(
                       Date.Year(Date.From(value)),
                           Date.Month(Date.From(value)),
                           Date.Day(Date.From(value)),
                           Time.Hour(Time.From(value)),
                           Time.Minute(Time.From(value)),
                           0
                   )
             ```
