# 目的
sqlの勉強をしている中で、INはEXISTSのどちらもサブクエリを利用でき、似ていることが判明した。
そこでどのような時にどちらを使うべきなのかまとめることにした。

# 環境
PostgreSQL 14.1 (Ubuntu 14.1-1.pgdg18.04+1) on x86_64-pc-linux-gnu

# 前提

>The Exists keyword evaluates true or false, but the IN keyword will compare all values in the corresponding subuery column.  If you are using the IN operator, the SQL engine will scan all records fetched from the inner query. On the other hand, if we are using EXISTS, the SQL engine will stop the scanning process as soon as it found a match.The EXISTS subquery is used when we want to display all rows where we have a matching column in both tables.  In most cases, this type of subquery can be re-written with a standard join to improve performance.

>The EXISTS clause is much faster than IN when the subquery results is very large. Conversely, the IN clause is faster than EXISTS when the subquery results is very small.

参考: SQL Exists vs. [IN clause](http://www.dba-oracle.com/t_exists_clause_vs_in_clause.htm#:~:text=The%20Exists%20keyword%20evaluates%20true,fetched%20from%20the%20inner%20query.)


つまり、データが多いほど、EXISTSの方が圧倒的に早くなるということだ。

[この記事](https://qiita.com/yutoun/items/2033a62d62812da359ed)を参考に、データを10000000件入れ直し、検証してみる。

これなら大きく差が出ると思う。

しかしどうやら差が大きく出るのはINとEXISTSではなく、NOT INとNOT EXISTSの場合のようである。

NOT INとNOT EXISTSの検証を行う

```bash
#basketball_team table
 member_id |  position  | profile_id
-----------+------------+------------
         1 | center     |          1
         2 | foward     |          2
         3 | guard      |          3
         4 | center     |          4
         5 | foward     |       1000

# member_detail table
 profile_id |  first_name  |  last_name   | age
------------+--------------+--------------+-----
          1 | 田中1        | 太郎1        |   1
          2 | 田中2        | 太郎2        |   2
          3 | 田中3        | 太郎3        |   3
          4 | 田中4        | 太郎4        |   4
・
・
・
     999999 | 田中99       | 太郎99       |  9999 
    10000000 | 田中100      | 太郎100      | 10000
(10000000 rows)
```
# IN
```bash
explain analyze select *
from basketball_team
where profile_id NOT IN (select profile_id
                     from customer
                     where profile_id = 1000);
                                                             QUERY PLAN
-------------------------------------------------------------------------------------------------------------------------------------
 Seq Scan on basketball_team  (cost=1203115.67..1203138.42 rows=510 width=52) (actual time=3297.341..5453.583 rows=4 loops=1)
   Filter: (NOT (hashed SubPlan 1))
   Rows Removed by Filter: 1
   SubPlan 1
     ->  Gather  (cost=1000.00..1203115.66 rows=1 width=4) (actual time=0.828..5453.533 rows=5 loops=1)
           Workers Planned: 2
           Workers Launched: 2
           ->  Parallel Seq Scan on customer  (cost=0.00..1202115.56 rows=1 width=4) (actual time=2016.804..3288.583 rows=2 loops=3)
                 Filter: (profile_id = 1000)
                 Rows Removed by Filter: 4006698
 Planning Time: 3.043 ms
 Execution Time: 5453.621 ms
(12 rows)
```

```
cost(a..b): aは最初の行を返すコスト。bは最後の行を返すコスト
width: 取得される行の平均サイズ
actual time：処理時間
rows：実行した結果、戻ってきた行数
loops：ステップの実行回数
Planning time：解析されたクエリから実行計画を生成し、最適化するまでの時間
Execution time：実行時間
```

# NOT EXISTS
```bash
explain analyze select *
from basketball_team
where NOT EXISTS (select *
                     from customer
                     where profile_id = 1000
                     and basketball_team.profile_id = customer.profile_id);
                                                               QUERY PLAN
----------------------------------------------------------------------------------------------------------------------------------------
 Nested Loop Anti Join  (cost=1000.00..1203151.17 rows=1020 width=52) (actual time=1085.024..1087.790 rows=4 loops=1)
   Join Filter: (basketball_team.profile_id = customer.profile_id)
   Rows Removed by Join Filter: 20
   ->  Seq Scan on basketball_team  (cost=0.00..20.20 rows=1020 width=52) (actual time=0.010..0.012 rows=5 loops=1)
   ->  Materialize  (cost=1000.00..1203115.67 rows=1 width=4) (actual time=0.081..217.553 rows=4 loops=5)
         ->  Gather  (cost=1000.00..1203115.66 rows=1 width=4) (actual time=0.398..1087.749 rows=5 loops=1)
               Workers Planned: 2
               Workers Launched: 2
               ->  Parallel Seq Scan on customer  (cost=0.00..1202115.56 rows=1 width=4) (actual time=634.223..1079.238 rows=2 loops=3)
                     Filter: (profile_id = 1000)
                     Rows Removed by Filter: 4006698
 Planning Time: 0.124 ms
 Execution Time: 1087.821 ms
(13 rows)
```

## NOT IN vs NOT EXISTS
```bash
# NOT IN:
Execution Time: 5453.621 ms

# NOT EXISTS:
Execution Time: 1087.821 ms
```

NOT EXISTSの圧勝だ。
５倍以上の速さでNOT EXISTSは1千万件のデータを処理できることがわかった。

## なぜこうなったか？？
NOT INは全レコードを検索する

NOT EXISTSは条件に合致する行を見つけたらそこで検索を打ち切る為、全レコードを検索の必要がないので高速らしい

## まとめ
やはりNOT INよりNOT EXISTSの方がパフォーマンスが優れていることがわかった。

フォローお願いします！
Twitter: [こちら](https://twitter.com/a_pathToFreedom)