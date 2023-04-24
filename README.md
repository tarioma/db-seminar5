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
SELECT * FROM Bank
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
SELECT * FROM Bank
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
SELECT * FROM Bank
```
