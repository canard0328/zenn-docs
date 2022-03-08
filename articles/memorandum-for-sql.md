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

テーブルにデータ（行）を挿入
文字列はシングルクォーテーションで囲みます。ダブルクォーテーションではありません。文字列内にシングルクォーテーションを含む場合は、シングルクォーテーションを2連続で記載するとエスケープできます。

```sql
INSERT INTO テーブル名
VALUES ('hoge', 'foo', 123, 456)
```

データの修正
WHEREを忘れると全行の値が置き換わってしまうため注意。

```sql
UPDATE テーブル名
SET カラム３ = 999
WHERE カラム１ = 'hoge'
```

データ（行）の削除

```sql
DELETE FROM テーブル名
WHERE カラム１ = 'hoge'
```
