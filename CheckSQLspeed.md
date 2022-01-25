# 目的
業務でついにチューニングをさせてもらえることになり、

1. 実行しているSQL文のログを出力
2. 現状のSQL文の実行速度を計測
3. SQL文の修正

という流れを考えている。

そこでどのようにして実行速度を計測するのか調べた。


大量データの登録方法は別記事にて確認。

# 環境
PostgreSQL 14.1 (Ubuntu 14.1-1.pgdg18.04+1) on x86_64-pc-linux-gnu

# いざ、EXPLAIN文を実行
```
explain select * from customer;
                        QUERY PLAN
-----------------------------------------------------------
 Seq Scan on customer  (cost=0.00..2.00 rows=100 width=42)
```
なんだか情報が少ない気がする

# analyzeオプションをつける
analyzeオプションをつけることでより詳細な実行結果が表示される

```sql
$ explain analyze select * from customer;

Seq Scan on customer  (cost=0.00..2.00 rows=100 width=42) (actual time=0.007..0.012 rows=100 loops=1)
Planning Time: 0.029 ms
Execution Time: 0.110 ms
(3 rows)
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
# formatオプションをつける

formatオプションをつけると「なんだか、見づらい」と思った人もフォーマットを指定して表示することができる。

```sql
explain (format json) select * from customer;
             QUERY PLAN
------------------------------------
 [                                 +
   {                               +
     "Plan": {                     +
       "Node Type": "Seq Scan",    +
       "Parallel Aware": false,    +
       "Async Capable": false,     +
       "Relation Name": "customer",+
       "Alias": "customer",        +
       "Startup Cost": 0.00,       +
       "Total Cost": 2.00,         +
       "Plan Rows": 100,           +
       "Plan Width": 42            +
     }                             +
   }                               +
 ]
(1 row)
```

# まとめ
これでSQLの実行時間がわかった。
あとはどれくらいの実行速度が早いのか遅いのかをもう少し調査しようと思う。