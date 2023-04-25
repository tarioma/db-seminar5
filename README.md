# 5 семинар

#### 1 задание
Используя курсор, посчитать средние значения всех числовых столбцов для результатов, возвращенных запросом №1 своего варианта индивидуального задания к лабораторной работе №3.
```
DECLARE
  @totalAmount decimal(12, 2),
  @totalAmountSum AS decimal(12, 2),
  @rowsCount AS int;
SET @totalAmountSum = 0;

DECLARE @totalAmounts TABLE (
  TotalAmount decimal(12, 2)
);

INSERT INTO @totalAmounts (TotalAmount)
  SELECT
    o.TotalAmount
  FROM
    [Order] AS o
  WHERE
    o.OrderDate BETWEEN '2013-12-1' AND '2014-2-1'

DECLARE avgNumbers CURSOR FOR
SELECT * FROM @totalAmounts;

OPEN avgNumbers;
FETCH NEXT FROM avgNumbers INTO @totalAmount;
WHILE @@FETCH_STATUS = 0 BEGIN
  SET @totalAmountSum = @totalAmountSum + @totalAmount;
  FETCH NEXT FROM avgNumbers INTO @totalAmount;
END
CLOSE avgNumbers;
DEALLOCATE avgNumbers;

SELECT @rowsCount = COUNT(*) FROM @totalAmounts;
PRINT N'Средняя стоимость: ';
PRINT @totalAmountSum / @rowsCount;
```
![изображение](https://user-images.githubusercontent.com/125894838/234026537-d0335f33-7db4-415f-8245-d963212fadc2.png)

![изображение](https://user-images.githubusercontent.com/125894838/234093901-d8c62465-34fa-42fb-a3de-859f37ee81e8.png)

#
#### 2 задание
В базе данных, созданной в лабораторной работе №2, выбрать 2 таблицы, связанные между собой отношением один-ко-многим, и для них написать хранимые процедуры, реализующие добавление, обновление и удаление записей.
```
CREATE TABLE Bank
(
    Id INT PRIMARY KEY,
    [Name] VARCHAR(255) NOT NULL UNIQUE CHECK (LEN([Name]) BETWEEN 2 AND 255)
);

CREATE TABLE ATM
(
    Number INT PRIMARY KEY,
    [Address] VARCHAR(255) NOT NULL CHECK (LEN(Address) BETWEEN 2 AND 255),
    RemainingCurrency MONEY NOT NULL,
    BankId INT NOT NULL REFERENCES Bank(Id)
);
```

```
CREATE PROCEDURE AddBank
  @id AS INT,
  @name AS VARCHAR(255)
AS
BEGIN
  INSERT INTO Bank (Id, [Name])
    VALUES (@id, @name)
END
EXEC AddBank 1, N'Exim'
```

```
CREATE PROCEDURE UpdateBank
  @id AS INT,
  @name AS VARCHAR(255)
AS
BEGIN
  UPDATE Bank
    SET [Name] = @name
  WHERE Id = @id
END
EXEC UpdateBank 1, N'EximUpdated'
```

```
CREATE PROCEDURE DeleteBank
  @id AS INT
AS
BEGIN
  DELETE FROM Bank
    WHERE Id = @id
END
EXEC DeleteBank 1
```

```
CREATE PROCEDURE AddATM
  @address AS VARCHAR(255),
  @remainingCurrency AS MONEY,
  @bankId AS INT
AS
BEGIN
  INSERT INTO ATM ([Address], RemainingCurrency, BankId)
    VALUES (@address, @remainingCurrency, @bankId)
END
EXEC AddATM N'Tiraspol', 16.35, 1
```

```
CREATE PROCEDURE UpdateATMAddress
  @number AS INT,
  @address AS VARCHAR(255)
AS
BEGIN
  UPDATE ATM
    SET [Address] = @address
  WHERE Number = @number
END
EXEC UpdateATMAddress 1, N'Bender'
```

```
CREATE PROCEDURE UpdateATMRemainingCurrency
  @number AS INT,
  @remainingCurrency AS MONEY
AS
BEGIN
  UPDATE ATM
    SET RemainingCurrency = @remainingCurrency
  WHERE Number = @number
END
EXEC UpdateATMRemainingCurrency 1, 16.30
```

```
CREATE PROCEDURE DeleteATM
  @number AS INT
AS
BEGIN
  DELETE FROM ATM
    WHERE Number = @number
END
EXEC DeleteATM 1
```

#
#### 3 задание
Написать скалярную функцию, принимающую в качестве параметра название продукта, и возвращающую его количество во всех заказах.
```
CREATE FUNCTION GetProductsCountByProductName
(
  @queryProductName AS VARCHAR(255)
)
RETURNS INT
AS
BEGIN
  DECLARE @productsCount AS INT;
  SELECT @productsCount =
    COUNT(*)
  FROM
    OrderItem
    JOIN Product ON Product.Id = OrderItem.ProductId
  WHERE
    Product.ProductName = @queryProductName
  RETURN @productsCount
END

SELECT dbo.GetProductsCountByProductName(N'Chang')
```
![изображение](https://user-images.githubusercontent.com/125894838/234368205-139e293e-134d-4d56-b6f3-493d98f1ab82.png)

#
#### 4 задание
Для базы данных, созданной в лабораторной работе №2, написать табличную функцию, которая будет возвращать следующую информацию:
- Вариант №1: Сведения об операциях определенного клиента на определенную дату. Фамилия клиента и дата передаются в качестве параметров.
<details>
  <summary>Создание и заполнение таблиц</summary>
  
    CREATE TABLE Bank
    (
        Id INT PRIMARY KEY IDENTITY(1, 1),
        [Name] VARCHAR(255) NOT NULL UNIQUE CHECK (LEN([Name]) BETWEEN 2 AND 255)
    );

    CREATE TABLE ATM
    (
        Number INT PRIMARY KEY IDENTITY(1, 1),
        [Address] VARCHAR(255) NOT NULL CHECK (LEN([Address]) BETWEEN 2 AND 255),
        RemainingCurrency MONEY NOT NULL,
        BankId INT NOT NULL REFERENCES Bank (Id) ON DELETE CASCADE
    );

    CREATE TABLE Client
    (
        PassportId INT PRIMARY KEY,
        FirstName VARCHAR(20) NOT NULL CHECK (LEN(FirstName) BETWEEN 2 AND 20),
        LastName VARCHAR(20) NOT NULL CHECK (LEN(LastName) BETWEEN 2 AND 20),
        Patronymic VARCHAR(20) CHECK (LEN(Patronymic) BETWEEN 2 AND 20 or Patronymic IS NULL),
        Balance MONEY NOT NULL CHECK (balance >= 0),
        Phone VARCHAR(14) CHECK (LEN(Phone) BETWEEN 5 AND 20 or Phone IS NULL),
        [Address] VARCHAR(255) NOT NULL CHECK (LEN([Address]) BETWEEN 8 AND 255),
        BankId INT NOT NULL REFERENCES Bank (Id) ON DELETE CASCADE
    );

    CREATE TABLE CreditCard
    (
        Number VARCHAR(16) PRIMARY KEY,
        ValidityPerson DATE NOT NULL,
        CVV SMALLINT NOT NULL CHECK (CVV BETWEEN 1 AND 999),
        Fee DECIMAL  NOT NULL CHECK (Fee BETWEEN 1 AND 100),
        ClientPassportId INT REFERENCES Client (PassportId) ON DELETE CASCADE
    );

    CREATE TABLE Operation
    (
        Id INT PRIMARY KEY IDENTITY(1, 1),
        [Date] DATE NOT NULL,
        Amount MONEY NOT NULL CHECK (Amount > 0),
        AtmNumber INT NOT NULL REFERENCES ATM (Number) ON DELETE CASCADE,
        CreditCardNumber VARCHAR(16) REFERENCES CreditCard (Number)
    );

    INSERT INTO Bank ([Name])
    VALUES
      ('Bank of America'),
      ('Wells Fargo'),
      ('JPMorgan Chase'),
      ('Citibank'),
      ('HSBC');

    INSERT INTO ATM ([Address], RemainingCurrency, BankId)
    VALUES
      ('123 Main Street, New York, NY', 1500.00, 1),
      ('456 Market Street, San Francisco, CA', 3000.00, 2),
      ('789 High Street, Boston, MA', 2000.00, 3),
      ('555 Oak Avenue, Los Angeles, CA', 2500.00, 4),
      ('777 Elm Street, Chicago, IL', 1750.00, 5);

    INSERT INTO Client (PassportId, FirstName, LastName, Patronymic, Balance, Phone, [Address], BankId)
    VALUES
      (123456, 'John', 'Doe', 'Smith', 1000, '1111111111', '123 Main St, Anytown USA', 1),
      (987654, 'Jane', 'Doe', NULL, 500, NULL, '456 Elm St, Anytown USA', 2),
      (456789, 'Alex', 'Johnson', 'Michael', 2500, '2222222222', '789 Maple Ave, Anytown USA', 3),
      (789123, 'Samantha', 'Williams', NULL, 750, '3333333333', '456 Oak Blvd, Anytown USA', 4),
      (654321, 'Mark', 'Davis', 'Robert', 10000, '4444444444', '987 Pine Rd, Anytown USA', 5);

    INSERT INTO CreditCard (Number, ValidityPerson, CVV, Fee, ClientPassportId)
    VALUES
      ('1234567812345678', '2025-12-31', 123, 3.5, 123456),
      ('2345678923456789', '2024-11-30', 234, 4.0, 987654),
      ('3456789034567890', '2026-08-31', 345, 2.5, 456789),
      ('4567890145678901', '2025-05-31', 456, 1.5, 789123),
      ('5678901256789012', '2024-06-30', 567, 3.0, 654321);

    INSERT INTO Operation ([Date], Amount, AtmNumber, CreditCardNumber)
    VALUES
      ('2022-03-15', 1500, 2, '3456789034567890'),
      ('2022-03-18', 500, 3, '2345678923456789'),
      ('2022-03-20', 1000, 1, '5678901256789012'),
      ('2022-03-22', 2000, 4, '1234567812345678'),
      ('2022-03-25', 700, 2, '4567890145678901'),
      ('2022-03-27', 1500, 3, '3456789034567890'),
      ('2022-03-28', 2000, 1, '2345678923456789'),
      ('2022-03-29', 1000, 4, '5678901256789012'),
      ('2022-04-01', 800, 2, '1234567812345678'),
      ('2022-04-03', 1500, 3, '4567890145678901');
</details>

```
CREATE FUNCTION GetClientOperations
(
  @clientLastName AS VARCHAR(20),
  @operationDate AS DATE
)
RETURNS TABLE
AS
RETURN
(
  SELECT
    Client.*,
    Operation.*
  FROM 
    Client
    JOIN CreditCard ON CreditCard.ClientPassportId = Client.PassportId
    JOIN Operation ON Operation.CreditCardNumber = CreditCard.Number
  WHERE
    Client.LastName = @clientLastName
    AND Operation.Date = @operationDate
)

SELECT * FROM dbo.GetClientOperations(N'Doe', '2022-03-22')
```
![изображение](https://user-images.githubusercontent.com/125894838/234380088-0603a889-a79e-4176-a095-9905dad36fdb.png)
