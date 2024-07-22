# Projeto de Integração SQL Server e Power BI

## Apresentação
O projeto de integração entre SQL Server e Power BI tem como objetivo criar uma visualização analítica robusta e detalhada, 
utilizando o banco de dados AdventureWorks 2014. Este projeto permitirá a análise de diversos indicadores de performance relacionados às vendas e aos clientes, 
proporcionando insights valiosos para a tomada de decisões estratégicas.

## Download do Banco de Dados AdventureWorks 2022
O banco de dados AdventureWorks 2022 pode ser baixado e configurado seguindo as instruções disponíveis no link oficial da Microsoft: 
AdventureWorks 2022 - Instruções de Instalação e Configuração.

## Definindo os Indicadores do Projeto

### Aba Geral:
1. Receita Total<br/>
2. Quantidade Vendida<br/>
3. Total de Categorias de Produtos<br/>
4. Quantidade de Clientes<br/>
5. Receita Total e Lucro Total por Mês<br/>
6. Margem de Lucro<br/>
7. Quantidade Vendida por Mês<br/>
8. Lucro por País<br/>

### Aba Clientes:
1. Vendas por País<br/>
2. Clientes por País<br/>
3. Vendas por Gênero<br/>
4. Vendas por Categoria<br/>

## Definindo as Tabelas e Colunas a Serem Usadas no Projeto
### Tabelas:
**FactInternetSales**<br/>
**DimProductCategory**<br/>
**DimCustomer**<br/>
**DimGeography**<br/>

### Colunas:
**SalesOrderNumber** (FactInternetSales)<br/>
**OrderDate** (FactInternetSales)<br/>
**EnglishProductCategoryName** (DimProductCategory)<br/>
**CustomerKey** (DimCustomer)<br/>
**FirstName + ' ' + LastName** (DimCustomer)<br/>
**Gender** (DimCustomer)<br/>
**EnglishCountryRegionName** (DimGeography)<br/>
**OrderQuantity** (FactInternetSales)<br/>
**SalesAmount** (FactInternetSales)<br/>
**TotalProductCost** (FactInternetSales)<br/>
**SalesAmount - TotalProductCost** (FactInternetSales)<br/>

## Criando a View  vw_AnaliseVendas
Para facilitar a visualização dos dados no Power BI, será criada uma view chamada **vw_AnaliseVendas**:
```
-- Criação ou alteração da view RESULTADOS_ADW
CREATE OR ALTER VIEW vw_AnaliseVendas AS
SELECT
    -- Número do pedido de venda
    fis.SalesOrderNumber AS 'NumeroPedido',
    
    -- Data do pedido de venda
    fis.OrderDate AS 'DataPedido',
    
    -- Nome da categoria do produto (em inglês)
    dpc.EnglishProductCategoryName AS 'CategoriaProduto',
    
    -- Chave do cliente (ID)
    fis.CustomerKey AS 'IDCliente',
    
    -- Nome completo do cliente
    dc.FirstName + ' ' + dc.LastName AS 'NomeCliente',
    
    -- Gênero do cliente (substituindo 'M' por 'Masculino' e 'F' por 'Feminino')
    REPLACE(REPLACE(dc.Gender, 'M', 'Masculino'), 'F', 'Feminino') AS 'GeneroCliente',
    
    -- Nome do país do cliente (em inglês)
    dg.EnglishCountryRegionName AS 'PaisCliente',
    
    -- Quantidade vendida
    fis.OrderQuantity AS 'QuantidadeVendida',
    
    -- Valor da venda
    fis.SalesAmount AS 'ValorVenda',
    
    -- Custo total da venda
    fis.TotalProductCost AS 'CustoVenda',
    
    -- Lucro da venda (valor da venda menos o custo total)
    fis.SalesAmount - fis.TotalProductCost AS 'LucroVenda'
    
FROM FactInternetSales fis
-- Relacionamento com a tabela de produtos
INNER JOIN DimProduct dp ON fis.ProductKey = dp.ProductKey
-- Relacionamento com a subcategoria dos produtos
INNER JOIN DimProductSubcategory dps ON dp.ProductSubcategoryKey = dps.ProductSubcategoryKey
-- Relacionamento com a categoria dos produtos
INNER JOIN DimProductCategory dpc ON dps.ProductCategoryKey = dpc.ProductCategoryKey
-- Relacionamento com a tabela de clientes
INNER JOIN DimCustomer dc ON fis.CustomerKey = dc.CustomerKey
-- Relacionamento com a tabela de geografia (localização dos clientes)
INNER JOIN DimGeography dg ON dc.GeographyKey = dg.GeographyKey
```

## Relacionamentos Entre Tabelas
Para a correta análise dos dados, serão definidos relacionamentos entre as tabelas:

### Tabelas a Serem Analisadas:
**1. FactInternetSales**<br/>
**2. DimCustomer**<br/>
**3. DimSalesTerritory**<br/>
**4. DimProductCategory**<br/>

Nota: A tabela DimProductCategory necessitará de um relacionamento em cadeia com outras tabelas de produtos para garantir a integridade e a completude dos dados analisados.

## Relacionamentos Entre Tabelas
Para a correta análise dos dados no Power BI, é necessário definir relacionamentos entre as tabelas envolvidas.<br/> 
Aqui estão os relacionamentos principais que devem ser estabelecidos:

### FactInternetSales e DimCustomer
Relacionar a coluna **CustomerKey em FactInternetSales** com a coluna **CustomerKey em DimCustomer**.<br/>

### FactInternetSales e DimGeography
Relacionar a coluna **CustomerKey em FactInternetSales** com a coluna **CustomerKey em DimCustomer**.<br/>
Em seguida, relacionar a coluna **GeographyKey em DimCustomer** com a coluna **GeographyKey em DimGeography**.

### FactInternetSales e DimProductCategory
Relacionar a coluna **ProductKey em FactInternetSales** com a coluna **ProductKey em DimProduct**.<br/>
Em seguida, relacionar a coluna **ProductSubcategoryKey em DimProduct** com a coluna **ProductSubcategoryKey em DimProductSubcategory**.

Por fim, relacionar a coluna **ProductCategoryKey em DimProductSubcategory** com a coluna **ProductCategoryKey em DimProductCategory**.<br/>
Esses relacionamentos permitirão a análise completa dos dados de vendas em relação aos clientes, localização geográfica e categorias de produtos.

## Criando Medidas no Power BI
Após a importação dos dados no Power BI, será necessário criar medidas (measures) para calcular os indicadores definidos.<br/> 
Aqui estão alguns exemplos de medidas que podem ser criadas:
```
-- Receita Total
Receita Total = SUM('FactInternetSales'[ValorVenda])

-- Quantidade Vendida
Quantidade Vendida = SUM('FactInternetSales'[QuantidadeVendida])

-- Total de Categorias de Produtos
Total de Categorias de Produtos = DISTINCTCOUNT('DimProductCategory'[CategoriaProduto])

-- Quantidade de Clientes
Quantidade de Clientes = DISTINCTCOUNT('DimCustomer'[IDCliente])

-- Receita Total por Mês
Receita Total por Mês = CALCULATE(SUM('FactInternetSales'[ValorVenda]), ALLEXCEPT('FactInternetSales', 'FactInternetSales'[DataPedido].[Month]))

-- Lucro Total por Mês
Lucro Total por Mês = CALCULATE(SUM('FactInternetSales'[LucroVenda]), ALLEXCEPT('FactInternetSales', 'FactInternetSales'[DataPedido].[Month]))

-- Margem de Lucro
Margem de Lucro = DIVIDE(SUM('FactInternetSales'[LucroVenda]), SUM('FactInternetSales'[ValorVenda]))

-- Quantidade Vendida por Mês
Quantidade Vendida por Mês = CALCULATE(SUM('FactInternetSales'[QuantidadeVendida]), ALLEXCEPT('FactInternetSales', 'FactInternetSales'[DataPedido].[Month]))

-- Lucro por País
Lucro por País = SUMMARIZE('FactInternetSales', 'DimGeography'[PaisCliente], "Lucro Total", SUM('FactInternetSales'[LucroVenda]))
```

## Criação dos Dashboards no Power BI
Com os dados e medidas importados, é possível criar dashboards interativos no Power BI para visualizar os indicadores de forma clara e intuitiva.<br/> 
Aqui estão algumas sugestões de visualizações para cada aba:

### Aba Geral:
Dashboard Visão Geral.<br/>
![image](https://github.com/niedzielski81/Projeto-Integra-o-SQL-Server-e-PowerBI/blob/main/Img/DashBoard_tela1_Vis%C3%A3o%20Geral.jpg)

Gráfico de colunas para Receita Total e Gráfico de linha para Lucro Total por Mês.<br/>
![image](https://github.com/niedzielski81/Projeto-Integra-o-SQL-Server-e-PowerBI/blob/main/Img/Grf%20Receita%20Total%20e%20Lucro%20Total%20por%20M%C3%AAs.jpg)

Gráfico de linha para Total Vendido por Mês.<br/>
![image](https://github.com/niedzielski81/Projeto-Integra-o-SQL-Server-e-PowerBI/blob/main/Img/Grf%20Total%20Vendido%20por%20M%C3%AAs.jpg)

Gráfico de barras para Lucro Total por País.<br/>
![image](https://github.com/niedzielski81/Projeto-Integra-o-SQL-Server-e-PowerBI/blob/main/Img/Grf%20Lucro%20Total%20por%20Pa%C3%ADs.jpg)

Gráfico de rosca para Margem de Lucro.<br/>
![image](https://github.com/niedzielski81/Projeto-Integra-o-SQL-Server-e-PowerBI/blob/main/Img/Grf%20Margem%20de%20Lucro.jpg)


### Aba Clientes:
Dashboard Clientes.<br/>
![image](https://github.com/niedzielski81/Projeto-Integra-o-SQL-Server-e-PowerBI/blob/main/Img/DashBoard_tela2_Clientes.jpg)

Mapa para exibir Vendas por País.<br/>
![image](https://github.com/niedzielski81/Projeto-Integra-o-SQL-Server-e-PowerBI/blob/main/Img/mapa%20Lucro%20por%20pa%C3%ADs.jpg)

Gráfico de colunas para Total de Clientes por País.<br/>
![image](https://github.com/niedzielski81/Projeto-Integra-o-SQL-Server-e-PowerBI/blob/main/Img/Grf%20Total%20de%20Clientes%20por%20Pa%C3%ADs.jpg)

Gráfico de rosca para Total de Vendas por Gênero.<br/>
![image](https://github.com/niedzielski81/Projeto-Integra-o-SQL-Server-e-PowerBI/blob/main/Img/Grf%20Total%20Clientes%20por%20G%C3%AAnero.jpg)

Gráfico de barras para Total de Vendas por Categoria.<br/>
![image](https://github.com/niedzielski81/Projeto-Integra-o-SQL-Server-e-PowerBI/blob/main/Img/Grf%20Total%20de%20Vendas%20por%20Categoria.jpg)

## Considerações Finais
O projeto de integração entre SQL Server e Power BI utilizando o banco de dados AdventureWorks 2022 visa fornecer uma solução completa para análise de dados de vendas e clientes.<br/> Através da definição de indicadores, criação de views, estabelecimento de relacionamentos e desenvolvimento de dashboards, é possível obter insights valiosos que podem auxiliar na tomada de decisões estratégicas para a organização.
