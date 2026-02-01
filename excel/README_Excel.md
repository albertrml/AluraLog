# AluraLog

Como parte da Semana 1 do Challenge BI, a AluraLog é uma empresa fictícia de logística e que necessita dos serviços de um profissional de Business Intelligence (BI).

## Contexto

A gerência da área de logística da AluraLog  está enfrentando algumas mudanças em razão do aumento da demanda dos serviços de logística no período da pandemia. Deseja-se manter a qualidade do serviço, mas para isso é necessário acompanhar constantemente as métricas do departamento para tomar as melhores decisões. 

Assim sendo, o objetivo é desenvolver um dashboard para o setor de logística com os KPIs informados pela gerência.

## KPIs

- Pedidos entregues por região e por estados
- Calcular os índices S2D (Ship to Door) e SLA (Service Level Agreement).
- Calcular o número de entregas feitas no prazo.
- Calcular  a quantidade de entregas que foram feitas atrasadas.
- Identificar o número de veículos disponíveis para entrega.
- Calcular o Ship to Door (S2D), que é o tempo da expedição até a chegada do produto para o cliente.
- Calcular o Ship to Door (SLA), que é a diferença do tempo de chegada do produto para o cliente e do tempo estimado para chegar à casa do mesmo.

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
1. ETL via Power Query
    1. Tabelas dimensões
        1. DIM_Localidade
           |    ID  |   Lat     |   Lon     |   UF da entrega   |   Estado  |   Região  |
           |--------|-----------|-----------|-------------------|-----------|-----------|
        2. DIM_Produto   
           |    ID  |  Produto  |   Preço   |
           |--------|-----------|-----------|
        3. DIM_Status_Entrega
           |    ID  |   Categoria   |
           |--------|---------------|
        4. DIM_Veiculo
           |    ID  |   Codigo  |   Tipo    |   Status  |
           |--------|-----------|-----------|-----------|
    2. Tabelas fatos
        1. FT_Estoque
           |    ID Produto    | Data atualização |  Quantidade  |
           |------------------|------------------|--------------|
        2. FT_Entregas
           |    ID    | ID Produto | ID Veiculo | ID Localidade | ID Status | Quantidade Comprada | Data de entrega | PTL (dias) | SLA | Modulo SLA (dias) | Subtotal |
           |----------|------------|------------|---------------|-----------|---------------------|-----------------|------------|-----|-------------------|----------|
2. Relacionamentos
    1. Para FT_Estoque
        1. DIM_Produto[ID] (1) --> (n) FT_Estoque[ID Produto]
        2. Calendar[Date] (1) --> (n) FT_Estoque[ID Produto]. Calendar é gerada via Power Pivot
    2. Para FT_Entregas
        1. DIM_Localidade[ID] (1) --> (n) FT_Entregas[ID Localidade]
        2. DIM_Produto[ID] (1) --> (n) FT_Entregas[ID Produto]
        3. DIM_Status_Entrega[ID] (1) --> (n) FT_Entregas[ID Status]
        4. DIM_Veiculo[ID] (1) --> (n) FT_Entregas[ID Veiculo]
3. Gráficos e campos
    1. Sem segmentação de dados: 
        1. Campos de disponibildade de veículos
        2. Gráficos de estoque por trimestres e anos
    2. Com segmentação por região: 
        1. Gráfico de pedidos atendidos
    3. Com segmentação por região e timeline:
        1. Gráfico de pedidos atendidos por região
        2. Campos de indicadores de entregas S2D e SLA
        3. Campos de entregas antecipadas, no prazo e atrasadas.
        
## Conclusões

1. Em termos de pedidos atendidos, as regiões Sudeste, Nordeste e Norte ocupam as três primeiras posições no ranking, considerando recortes de tempo mensal, trimestral e anual. 
2. A quantidade de pedidos atendidos na região Sudeste é impulsionada majoritariamente por São Paulo e Rio de Janeiro. 
3. Mesmo sendo a terceira região com mais pedidos atendidos, a região Norte possui a quantidade equivalente à soma registradas pela região Centro-Oeste e Sul, considerando recortes de tempo mensal, trimestral e anual
4. Há aproximadamente 15% de pedidos sendo entregues fora do prazo.
5. O comportamento dos indicadores de entrega são semelhantes para os recortes de tempo e região. Assim sendo:
    1. Em relação ao indicador S2D:
        1. Para as entregas antecipadas, 50% das entregas são realizadas entre 6 a 13 dias. No melhor e pior caso, as entregas ocorrem em 3 e 19 dias, respectivamente.
        2. Para as entregas atrasadas, 50% das entregas ocorrem entre 18 a 20 dias.
        3. Para as entregas no prazo, 50% das entregas ocorrem entre 16 a 19 dias.
    2. Em relação ao indicador SLA:
        1. Para as entregas antecipadas, 50% das entregas ocorrem com 4 a 11 dias de antecipação. No melhor caso a entrega ocorre em até 17 antes do prazo.
        2. Para as entregas atrasadas, 50% das entregas atrasam em 1 a 3 dias. No pior caso, o pedido atrasa em 5 dias.
    3. Com base nos indicadores, aconselha-se: 
        1. Paliativamente, ajustar a estimativa de entrega dentro do intervalo de 17 a 20 dias para qualquer região.
        2. Verificar as estimativas de entrega para os casos de atraso em relação ao tipo de transportes e produtos, visando identificar inadequações.
