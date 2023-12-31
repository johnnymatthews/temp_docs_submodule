---
slug: /ru/sql-reference/aggregate-functions/reference/kurtsamp
sidebar_position: 154
---

# kurtSamp {#kurtsamp}

Вычисляет [выборочный коэффициент эксцесса](https://ru.wikipedia.org/wiki/Статистика_(функция_выборки)) для последовательности.

Он представляет собой несмещенную оценку эксцесса случайной величины, если переданные значения образуют ее выборку.

``` sql
kurtSamp(expr)
```

**Аргументы**

`expr` — [выражение](../../syntax.md#syntax-expressions), возвращающее число.

**Возвращаемое значение**

Коэффициент эксцесса заданного распределения. Тип — [Float64](../../../sql-reference/data-types/float.md). Если `n <= 1` (`n` — размер выборки), тогда функция возвращает `nan`.

**Пример**

``` sql
SELECT kurtSamp(value) FROM series_with_value_column;
```
