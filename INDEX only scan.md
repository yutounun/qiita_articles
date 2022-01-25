# 目的
本でindex only scanについて学んだが、あまり想像がつかなかったため実際に試してみることにする。

# 環境
PostgreSQL 14.1 (Ubuntu 14.1-1.pgdg18.04+1) on x86_64-pc-linux-gnu

# 前提
[この記事](https://qiita.com/yutoun/items/2033a62d62812da359ed)を参考に、データを10000000件入れ直し、検証してみる。

# index only scanとは？？
Index ScanはIndexをスキャン後にテーブルにアクセスし、該当する行のみを取得するためシーケンシャルスキャンよりも早い。

簡単に言うとindex only scanはIndexをスキャンした時に、取得する項目カラムはIndex情報にあるので、
テーブルからデータを引っこ抜いてインデックスの項目を取得せずとも、Indexにあるデータを取得すれば良いというもの

# 比較してみる
```bash
explain (BUFFERS,ANALYZE) select first_name from customer where first_name = '田中10000000';
                                                               QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------------
# ここにindex only scanを利用していることが示されている
 Index Only Scan using customer_first_name_idx on customer  (cost=0.56..8.58 rows=1 width=15) (actual time=0.170..0.176 rows=1 loops=1)
   Index Cond: (first_name = '田中10000000'::bpchar)
   Heap Fetches: 1
   Buffers: shared hit=5
 Planning Time: 0.153 ms
 Execution Time: 0.203 ms
(6 rows)
```

```bash
explain (BUFFERS,ANALYZE) select last_name from customer where last_name = '太郎10000000';
                                                        QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------
 Gather  (cost=1000.00..803373.79 rows=1 width=15) (actual time=982.712..985.573 rows=1 loops=1)
   Workers Planned: 2
   Workers Launched: 2
   Buffers: shared hit=16118 read=723651
# シーケンシャルスキャンつまり全文検索していることが示されている
   ->  Parallel Seq Scan on customer  (cost=0.00..802373.69 rows=1 width=15) (actual time=977.126..977.127 rows=0 loops=3)
         Filter: (last_name = '太郎10000000'::bpchar)
         Rows Removed by Filter: 4006700
         Buffers: shared hit=16118 read=723651
 Planning Time: 0.085 ms
 Execution Time: 985.597 ms
(10 rows)
```

## index only scan vs 通常
```bash
# index only scan:
Execution Time: 0.203 ms

# 通常:
Execution Time: 985.597 ms
```

index only scanを使った場合は通常のフルスキャンと比べて5000倍ほど早いことが判明した。
