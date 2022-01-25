# 目的
Viewとサブクエリは書き方が多少違うが、一度だけ使うという目的であれば同じように使うことができると考えている。
そこで、VIEWとサブクエリの実行速度などを調べてみることでどのような状況だとどちらが向いているのか実際に検証していく。

# 環境
PostgreSQL 14.1 (Ubuntu 14.1-1.pgdg18.04+1) on x86_64-pc-linux-gnu

# VIEWの場合
```bash
create view test_view (first_name, last_name)
as
select first_name, last_name, count(*)
from customer
group by first_name, last_name;

explain analyze select * from test_view;
                                                 QUERY PLAN
-------------------------------------------------------------------------------------------------------------
 HashAggregate  (cost=2.75..3.75 rows=100 width=38) (actual time=0.074..0.085 rows=100 loops=1)
   Group Key: customer.first_name, customer.last_name
   Batches: 1  Memory Usage: 32kB
   ->  Seq Scan on customer  (cost=0.00..2.00 rows=100 width=30) (actual time=0.004..0.009 rows=100 loops=1)
 Planning Time: 0.089 ms
 Execution Time: 0.115 ms
(6 rows)
```


# サブクエリの場合

```bash
$ explain analyze 
select first_name, last_name
from (select first_name, last_name, count(*)
      from customer
      group by first_name, last_name) as test;
                                                    QUERY PLAN
-------------------------------------------------------------------------------------------------------------------
 Subquery Scan on test  (cost=2.50..4.50 rows=100 width=30) (actual time=0.074..0.106 rows=100 loops=1)
   ->  HashAggregate  (cost=2.50..3.50 rows=100 width=38) (actual time=0.073..0.092 rows=100 loops=1)
         Group Key: customer.first_name, customer.last_name
         Batches: 1  Memory Usage: 32kB
         ->  Seq Scan on customer  (cost=0.00..2.00 rows=100 width=30) (actual time=0.008..0.018 rows=100 loops=1)
 Planning Time: 0.186 ms
 Execution Time: 0.148 ms
(7 rows)
```

# 比較してみる

```
cost(a..b): aは最初の行を返すコスト。bは最後の行を返すコスト
width: 取得される行の平均サイズ
actual time：処理時間
rows：実行した結果、戻ってきた行数
loops：ステップの実行回数
Planning time：解析されたクエリから実行計画を生成し、最適化するまでの時間
Execution time：実行時間
```
この情報に基づいて、考えてみると
やはりサブクエリよりもVIEWを使ったSELECT文の方が
- Execution Time
- Planning Time
- actual time
何をとっても時間的にはやいことがわかった

# まとめ
作られたVIEWを呼び出してSELECTを実行する方が、SELECTの中のサブクエリを利用するよりも速度的に優位なことがわかった。

だが、やはりVIEWを作成する一手間で時間も作業も増えることを考えると複数回同じVIEWを回すのでない場合はサブクエリを利用したほうがいいかと思う。s