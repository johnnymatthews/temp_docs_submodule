---
slug: /ru/sql-reference/aggregate-functions/reference/groupbitxor
sidebar_position: 127
---

# groupBitXor {#groupbitxor}

Применяет побитовое `ИСКЛЮЧАЮЩЕЕ ИЛИ` для последовательности чисел.

``` sql
groupBitXor(expr)
```

**Аргументы**

`expr` – выражение, результат которого имеет тип данных `UInt*`.

**Возвращаемое значение**

Значение типа `UInt*`.

**Пример**

Тестовые данные:

``` text
binary     decimal
00101100 = 44
00011100 = 28
00001101 = 13
01010101 = 85
```

Запрос:

``` sql
SELECT groupBitXor(num) FROM t
```

Где `num` — столбец с тестовыми данными.

Результат:

``` text
binary     decimal
01101000 = 104
```
