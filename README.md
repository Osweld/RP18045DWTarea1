# Datawarehouse TailspinToys

<img width="970" height="580" alt="image" src="https://github.com/user-attachments/assets/e5941fa8-68b5-4d83-9c17-7aae9b2506e3" />

### Creacion de la base de datos con las tablas

```sql
IF DB_ID('DWTailspinToys') IS NULL
    CREATE DATABASE DWTailspinToys;
GO

USE DWTailspinToys;
GO



-- DimDate
CREATE TABLE dbo.DimDate (
    dateKey         INT        NOT NULL PRIMARY KEY,
    [date]          DATETIME   NOT NULL,
    [day]           TINYINT    NOT NULL,
    [month]         TINYINT    NOT NULL,
    [year]          SMALLINT   NOT NULL
);

-- DimProduct
CREATE TABLE dbo.DimProduct (
    productKey       INT IDENTITY(1,1) PRIMARY KEY,
    productId        INT           NOT NULL,
    productSKU       VARCHAR(100)  NULL,
    productName      VARCHAR(200)  NULL,
    productCategory  VARCHAR(100)  NULL,
    itemGroup        VARCHAR(50)   NULL,
    kitType          VARCHAR(30)   NULL,
    channels         VARCHAR(50)   NULL,
    demographic      VARCHAR(50)   NULL,
    retailPrice      DECIMAL(18,2) NULL,
    effectiveFrom    DATE          NOT NULL DEFAULT (CAST(GETDATE() AS date)),
    effectiveTo      DATE          NOT NULL DEFAULT ('9999-12-31'),
    isCurrent        BIT           NOT NULL DEFAULT (1)
);

-- DimState
CREATE TABLE dbo.DimState (
    stateKey      INT IDENTITY(1,1) PRIMARY KEY,
    stateId       INT           NOT NULL,
    stateCode     VARCHAR(2)    NOT NULL,
    stateName     VARCHAR(100)  NOT NULL,
    timeZone      VARCHAR(50)   NULL,
    regionName    VARCHAR(100)  NULL,
    effectiveFrom    DATE       NOT NULL DEFAULT (CAST(GETDATE() AS date)),
    effectiveTo      DATE       NOT NULL DEFAULT ('9999-12-31'),
    isCurrent        BIT        NOT NULL DEFAULT (1)
);

-- FactSales
CREATE TABLE dbo.FactSales (
    orderDateKey      INT           NOT NULL,
    shipDateKey       INT           NOT NULL,
    productKey        INT           NOT NULL,
    stateKey          INT           NOT NULL,
    orderNumber       VARCHAR(50)   NOT NULL,
    promotionCode     VARCHAR(50)   NOT NULL,
    quantity          INT           NOT NULL,
    unitPrice         DECIMAL(19,4) NOT NULL,
    discountAmount    DECIMAL(19,4) NOT NULL,
    netSales          DECIMAL(19,4) NOT NULL,

    CONSTRAINT FK_FactSales_OrderDate      FOREIGN KEY (orderDateKey)     REFERENCES dbo.DimDate(dateKey),
    CONSTRAINT FK_FactSales_ShipDate       FOREIGN KEY (shipDateKey)      REFERENCES dbo.DimDate(dateKey),
    CONSTRAINT FK_FactSales_Product        FOREIGN KEY (productKey)       REFERENCES dbo.DimProduct(productKey),
    CONSTRAINT FK_FactSales_State          FOREIGN KEY (stateKey)         REFERENCES dbo.DimState(stateKey)
);
```


### Procesos de ETL del datawarehouse

```sql

SELECT 
    s.StateID,
    s.StateCode,
    s.StateName,
    s.TimeZone,
    r.RegionName
FROM dbo.State s
INNER JOIN Region r ON r.RegionID = s.RegionID;



SELECT
    ProductID,
    ProductSKU,
    ProductName,
    ProductCategory,
    ItemGroup,
    CASE KitType
        WHEN 'KIT' THEN 'Assembly Kit'
        WHEN 'RTF' THEN 'Ready to Use'
        ELSE 'Unknown'
    END AS kitType,
    CASE Channels
        WHEN 3 THEN 'Channel 3'
        WHEN 4 THEN 'Channel 4'
        WHEN 5 THEN 'Channel 5'
        WHEN 6 THEN 'Channel 6'
        ELSE 'Unknown'
    END AS channels,

    Demographic,
    RetailPrice
FROM dbo.Product;



SELECT
  OrderNumber,
  CAST(OrderDate AS DATETIME) AS OrderDate,
  CAST(ISNULL(ShipDate,'1900-01-01 00:00:00.000') AS DATETIME) AS ShipDate,
  CustomerStateID,
  ProductID,
  Quantity,
  UnitPrice,
  DiscountAmount,
  ISNULL(PromotionCode,'NO PROMO') AS PromotionCode,
  CAST(CAST(Quantity AS DECIMAL(19,4)) * UnitPrice - DiscountAmount AS DECIMAL(19,4)) AS NetSales
FROM dbo.Sales;


```

<img width="1103" height="540" alt="image" src="https://github.com/user-attachments/assets/87ecdfc7-acbc-4703-9ba9-465873af8de5" />


