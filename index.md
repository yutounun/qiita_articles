# 目的
SQLのパフォーマンスチューニングにはINDEXの利用が欠かせないという話はいろんな方面から聞いてきたため、INDEXなるものを実際に使ってどれほどの差が出るのか実際に検証してみる。

# 環境
PostgreSQL 14.1 (Ubuntu 14.1-1.pgdg18.04+1) on x86_64-pc-linux-gnu

# INDEXとは
データのレコードにつける印のようなもの。

これがあると本で目次を見て、みたいページに飛ぶように、数あるレコードの中で条件に一致したレコードに瞬時に飛ぶことができる。

# テストデータを注入してみる
[この記事](https://qiita.com/yutoun/items/2033a62d62812da359ed)を参考に、データを10000000件入れ直し、検証してみる。

# INDEXを設定してみる
作成したテーブルのfirst_nameにINDEXを設定してみる。

```bash
create index on customer (first_name);

\dCREATE INDEX
test=# \d customer;
                   Table "public.customer"
   Column   |     Type      | Collation | Nullable | Default
------------+---------------+-----------+----------+---------
 profile_id | integer       |           |          |
 first_name | character(10) |           |          |
 last_name  | character(10) |           |          |
 age        | integer       |           |          |
Indexes:
    "customer_first_name_idx" btree (first_name)
```
実際にINDEXが設定できているのか確認する。

customerテーブルのカラムの詳細を確認するとINDEX情報も記される。

- INDEX名を指定していないため、customer_first_name_idxがINDEX名
- 使用されているINDEXの種類がb-tree
- first_nameにINDEXが設定されている

ということを示している。


```bash
\d customer;

                   Table "public.customer"
   Column   |     Type      | Collation | Nullable | Default
------------+---------------+-----------+----------+---------
 profile_id | integer       |           |          |
 first_name | character(10) |           |          |
 last_name  | character(10) |           |          |
 age        | integer       |           |          |
Indexes:
    "customer_first_name_idx" btree (first_name)
```

# INDEXの有無で比較してみる
INDEXを設定していないlast_nameとINDEXを設定したfirst_nameで検索をかけた場合を比較してみる

```bash

# INDEXを利用しない場合
explain (BUFFERS,ANALYZE) select * from customer where last_name = '太郎1000001';
                                                        QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..803373.79 rows=1 width=38) (actual time=719.513..1000.599 rows=1 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   Buffers: shared hit=16018 read=723751
   ->  Parallel Seq Scan on customer  (cost=0.00..802373.69 rows=1 width=38) (actual time=899.488..992.163 rows=0 loops=3)
         Filter: (last_name = '太郎1000001'::bpchar)
         Rows Removed by Filter: 4006700
         Buffers: shared hit=16018 read=723751
 Planning Time: 0.094 ms
 Execution Time: 1000.629 ms
(10 rows)

# INDEXを利用する場合
explain (BUFFERS,ANALYZE) select * from customer where first_name = '太郎1003001';
                                                            QUERY PLAN
-----------------------------------------------------------------------------------------------------------------------------------
 Index Scan using customer_first_name_idx on customer  (cost=0.56..8.58 rows=1 width=38) (actual time=0.024..0.025 rows=0 loops=1)
   Index Cond: (first_name = '太郎1003001'::bpchar)
   Buffers: shared hit=4
 Planning Time: 0.096 ms
 Execution Time: 0.048 ms
(5 rows)
```

INDEXを利用した場合(first_nameで検索をした場合):
Execution Time: 1000.629 ms

INDEXを利用していない場合(last_nameで検索をした場合):
Execution Time: 0.048 ms

# 結果
結果を比較すると、INDEXを設定したカラムで検索をする方が20000倍以上実行時間が早いことがわかった。
通りでINDEXは大事だと言われるわけだ。

INDEXのイメージがなかなかついていなかったが、これでINDEXの理解が深まった。