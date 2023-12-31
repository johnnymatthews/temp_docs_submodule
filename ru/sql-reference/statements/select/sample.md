---
slug: /ru/sql-reference/statements/select/sample
sidebar_label: SAMPLE
---

# Секция SAMPLE {#select-sample-clause}

Секция `SAMPLE` позволяет выполнять запросы приближённо. Например, чтобы посчитать статистику по всем визитам, можно обработать 1/10 всех визитов и результат домножить на 10.

Сэмплирование имеет смысл, когда:

1.  Точность результата не важна, например, для оценочных расчетов.
2.  Возможности аппаратной части не позволяют соответствовать строгим критериям. Например, время ответа должно быть &lt;100 мс. При этом точность расчета имеет более низкий приоритет.
3.  Точность результата участвует в бизнес-модели сервиса. Например, пользователи с бесплатной подпиской на сервис могут получать отчеты с меньшей точностью, чем пользователи с премиум подпиской.

:::note Внимание
Не стоит использовать сэмплирование в тех задачах, где важна точность расчетов. Например, при работе с финансовыми отчетами.
:::
Свойства сэмплирования:

-   Сэмплирование работает детерминированно. При многократном выполнении одного и того же запроса `SELECT .. SAMPLE`, результат всегда будет одинаковым.
-   Сэмплирование поддерживает консистентность для разных таблиц. Имеется в виду, что для таблиц с одним и тем же ключом сэмплирования, подмножество данных в выборках будет одинаковым (выборки при этом должны быть сформированы для одинаковой доли данных). Например, выборка по идентификаторам посетителей выберет из разных таблиц строки с одинаковым подмножеством всех возможных идентификаторов. Это свойство позволяет использовать выборки в подзапросах в секции [IN](../../operators/in.md#select-in-operators), а также объединять выборки с помощью [JOIN](join.md#select-join).
-   Сэмплирование позволяет читать меньше данных с диска. Обратите внимание, для этого необходимо корректно указать ключ сэмплирования. Подробнее см. в разделе [Создание таблицы MergeTree](../../../engines/table-engines/mergetree-family/mergetree.md#table_engine-mergetree-creating-a-table).

Сэмплирование поддерживается только таблицами семейства [MergeTree](../../../engines/table-engines/mergetree-family/mergetree.md#table_engine-mergetree-creating-a-table) и только в том случае, если для таблиц был указан ключ сэмплирования (выражение, на основе которого должна производиться выборка). Подробнее см. в разделе [Создание таблиц MergeTree](../../../engines/table-engines/mergetree-family/mergetree.md#table_engine-mergetree-creating-a-table).

Выражение `SAMPLE` в запросе можно задать следующими способами:

| Способ задания SAMPLE | Описание                                                                                                                                                                                                                                                         |
|-----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `SAMPLE k`            | Здесь `k` – это дробное число в интервале от 0 до 1.<br/> Запрос будет выполнен по `k` доле данных. Например, если указано `SAMPLE 1/10`, то запрос будет выполнен для выборки из 1/10 данных. [Подробнее](#select-sample-k)                                     |
| `SAMPLE n`            | Здесь `n` – это достаточно большое целое число.<br/> Запрос будет выполнен для выборки, состоящей из не менее чем `n` строк. Например, если указано `SAMPLE 10000000`, то запрос будет выполнен для не менее чем 10,000,000 строк. [Подробнее](#select-sample-n) |
| `SAMPLE k OFFSET m`   | Здесь `k` и `m` – числа от 0 до 1.<br/> Запрос будет выполнен по `k` доле данных. При этом выборка будет сформирована со смещением на `m` долю. [Подробнее](#select-sample-offset)                                                                               |

## SAMPLE k {#select-sample-k}

Здесь `k` – число в интервале от 0 до 1. Поддерживается как дробная, так и десятичная форма записи. Например, `SAMPLE 1/2` или `SAMPLE 0.5`.

Если задано выражение `SAMPLE k`, запрос будет выполнен для `k` доли данных. Рассмотрим пример:

``` sql
SELECT
    Title,
    count() * 10 AS PageViews
FROM hits_distributed
SAMPLE 0.1
WHERE
    CounterID = 34
GROUP BY Title
ORDER BY PageViews DESC LIMIT 1000
```

В этом примере запрос выполняется по выборке из 0.1 (10%) данных. Значения агрегатных функций не корректируются автоматически, поэтому чтобы получить приближённый результат, значение `count()` нужно вручную умножить на 10.

Выборка с указанием относительного коэффициента является «согласованной»: для таблиц с одним и тем же ключом сэмплирования, выборка с одинаковой относительной долей всегда будет составлять одно и то же подмножество данных. То есть выборка из разных таблиц, на разных серверах, в разное время, формируется одинаковым образом.

## SAMPLE n {#select-sample-n}

Здесь `n` – это достаточно большое целое число. Например, `SAMPLE 10000000`.

Если задано выражение `SAMPLE n`, запрос будет выполнен для выборки из не менее `n` строк (но не значительно больше этого значения). Например, если задать `SAMPLE 10000000`, в выборку попадут не менее 10,000,000 строк.

:::note Примечание
Следует иметь в виду, что `n` должно быть достаточно большим числом. Так как минимальной единицей данных для чтения является одна гранула (её размер задаётся настройкой `index_granularity` для таблицы), имеет смысл создавать выборки, размер которых существенно превосходит размер гранулы.
:::
При выполнении `SAMPLE n` коэффициент сэмплирования заранее неизвестен (то есть нет информации о том, относительно какого количества данных будет сформирована выборка). Чтобы узнать коэффициент сэмплирования, используйте столбец `_sample_factor`.

Виртуальный столбец `_sample_factor` автоматически создается в тех таблицах, для которых задано выражение `SAMPLE BY` (подробнее см. в разделе [Создание таблицы MergeTree](../../../engines/table-engines/mergetree-family/mergetree.md#table_engine-mergetree-creating-a-table)). В столбце содержится коэффициент сэмплирования для таблицы – он рассчитывается динамически по мере добавления данных в таблицу. Ниже приведены примеры использования столбца `_sample_factor`.

Предположим, у нас есть таблица, в которой ведется статистика посещений сайта. Пример ниже показывает, как рассчитать суммарное число просмотров:

``` sql
SELECT sum(PageViews * _sample_factor)
FROM visits
SAMPLE 10000000
```

Следующий пример показывает, как посчитать общее число визитов:

``` sql
SELECT sum(_sample_factor)
FROM visits
SAMPLE 10000000
```

В примере ниже рассчитывается среднее время на сайте. Обратите внимание, при расчете средних значений, умножать результат на коэффициент сэмплирования не нужно.

``` sql
SELECT avg(Duration)
FROM visits
SAMPLE 10000000
```

## SAMPLE k OFFSET m {#select-sample-offset}

Здесь `k` и `m` – числа в интервале от 0 до 1. Например, `SAMPLE 0.1 OFFSET 0.5`. Поддерживается как дробная, так и десятичная форма записи.

При задании `SAMPLE k OFFSET m`, выборка будет сформирована из `k` доли данных со смещением на долю `m`. Примеры приведены ниже.

**Пример 1**

``` sql
SAMPLE 1/10
```

В этом примере выборка будет сформирована по 1/10 доле всех данных:

`[++------------------]`

**Пример 2**

``` sql
SAMPLE 1/10 OFFSET 1/2
```

Здесь выборка, которая состоит из 1/10 доли данных, взята из второй половины данных.

`[----------++--------]`
