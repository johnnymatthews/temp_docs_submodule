---
slug: /zh/engines/table-engines/special/view
---
# 视图 {#table_engines-view}

用于构建视图（有关更多信息，请参阅 `CREATE VIEW 查询`）。 它不存储数据，仅存储指定的 `SELECT` 查询。 从表中读取时，它会运行此查询（并从查询中删除所有不必要的列）。
