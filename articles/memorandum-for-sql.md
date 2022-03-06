---
title: "SQLの備忘録"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SQL"]
published: false
---

SQLを一から学んでいく備忘録です。
[スッキリわかるSQL入門](https://amzn.to/3HL5ibI)を参考にしています。

特定列の抽出

```sql
SELECT カラム名
FROM テーブル名
```

複数列の抽出

```sql
SELECT カラム１, カラム２, カラム３
FROM テーブル名
```

全列の抽出

```sql
SELECT *
FROM テーブル名
```

条件に一致する行の抽出

```sql
SELECT カラム１, カラム２, カラム３
FROM テーブル名
WHERE カラム３ > 3000
```

複数の条件を指定

```sql
SELECT カラム１, カラム２, カラム３
FROM テーブル名
WHERE (カラム３ > 3000 AND カラム３ < 5000)
OR カラム２ != 0
```
