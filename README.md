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
