---
slug: /ru/introduction/performance
sidebar_position: 6
sidebar_label: "Производительность"
---

# Производительность {#proizvoditelnost}

По результатам внутреннего тестирования в Яндексе ClickHouse обладает наиболее высокой производительностью (как наиболее высокой пропускной способностью на длинных запросах, так и наиболее низкой задержкой на коротких запросах) при соответствующем сценарии работы среди доступных для тестирования систем подобного класса. Результаты тестирования можно посмотреть на [отдельной странице](https://clickhouse.com/benchmark/dbms/).

Также это подтверждают многочисленные независимые бенчмарки. Их несложно найти в Интернете самостоятельно, либо можно воспользоваться [небольшой коллекцией ссылок по теме](https://clickhouse.com/#independent-benchmarks).

## Пропускная способность при обработке одного большого запроса {#propusknaia-sposobnost-pri-obrabotke-odnogo-bolshogo-zaprosa}

Пропускную способность можно измерять в строчках в секунду и в мегабайтах в секунду. При условии, что данные помещаются в страничный кэш (page cache), не слишком сложный запрос обрабатывается на современном железе со скоростью около 2—10 ГБ/с несжатых данных на одном сервере (в простейшем случае скорость может достигать 30 ГБ/с). Если данные не помещаются в страничный кэш, то скорость работы зависит от скорости подсистемы ввода-вывода и коэффициента сжатия данных. Например, если подсистема ввода-вывода позволяет читать данные со скоростью 400 МБ/с, а коэффициент сжатия данных составляет 3, то скорость будет около 1,2 ГБ/с. Для получения скорости в строках в секунду следует поделить скорость в байтах в секунду на суммарный размер используемых в запросе столбцов. Например, если запрашиваются столбцы на 10 байт, то скорость будет в районе 100—200 млн строк в секунду.

При распределённой обработке запроса скорость обработки запроса растёт почти линейно, но только при условии, что в результате агрегации или при сортировке получается не слишком большое множество строк.

## Задержки при обработке коротких запросов {#zaderzhki-pri-obrabotke-korotkikh-zaprosov}

Если запрос использует первичный ключ и выбирает для обработки не слишком большое количество строк (сотни тысяч), и при этом использует не слишком большое количество столбцов, то вы можете рассчитывать на задержку менее 50 миллисекунд (от единиц миллисекунд в лучшем случае) при условии, что данные помещаются в страничный кэш. В иных случаях задержка зависит от количества операций поиска (seek). Если вы используйте дисковые накопители, то на не слишком сильно нагруженной системе задержка вычисляется по формуле: seek time (10 мс) \* количество столбцов в запросе \* количество кусков с данными.

## Пропускная способность при обработке многочисленных коротких запросов {#propusknaia-sposobnost-pri-obrabotke-mnogochislennykh-korotkikh-zaprosov}

При тех же условиях ClickHouse может обработать несколько сотен (до нескольких тысяч в лучшем случае) запросов в секунду на одном сервере. Так как такой сценарий работы не является типичным для аналитических СУБД, рекомендуется рассчитывать на скорость не более, чем 100 запросов в секунду.

## Производительность при вставке данных {#proizvoditelnost-pri-vstavke-dannykh}

Данные рекомендуется вставлять пачками не менее 1000 строк или не более одного запроса в секунду. При вставке в таблицу типа MergeTree из текстового файла с табуляцией в качестве разделителя скорость вставки будет в районе 50—200 МБ/с. Если вставляются строки размером около 1 КБ, то скорость будет в районе 50 000 — 200 000 строк в секунду. Если строки маленькие — производительность в строках в секунду будет выше (на данных БК - `>` 500 000 строк в секунду, на данных Graphite - `>` 1 000 000 строк в секунду). Для увеличения производительности можно делать несколько запросов INSERT параллельно — при этом производительность растёт линейно.
