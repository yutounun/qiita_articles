# 目的
VIEWは割と業務で使うようなので、本で学んだ内容をまとめて、実行することで理解を深める。

# 環境
PostgreSQL 14.1 (Ubuntu 14.1-1.pgdg18.04+1) on x86_64-pc-linux-gnu

# VIEWとは
テーブルのようだが、SELECT文を保存するクリップボードのようなものでデータそのものは格納していない 

→ 元のテーブルが更新されると自動で更新される　

## view vs table
**実際にSELECTすると、テーブルが表示されるのは同じ**

テーブルを作ってINSERTするとそのデータは記憶装置に保存される。

viewはどこにも保存せず、SELECT文があるだけ。呼び出された時にそれを使って仮想のテーブルのようなものを作って返す。
# 前提
テーブルは下記のものを用意。
テストテーブルレコードの大量作成はこの記事を確認。


```sql

--profileテーブル--

profile_id |  first_name  |  last_name   |  age
----------+--------------+--------------+--------
        1 | 田中1        | 太郎1        | 1才
        2 | 田中2        | 太郎2        | 2才
        3 | 田中3        | 太郎3        | 3才
        4 | 田中4        | 太郎4        | 4才
        5 | 田中5        | 太郎5        | 5才
        6 | 田中6        | 太郎6        | 6才
        7 | 田中7        | 太郎7        | 7才
        8 | 田中8        | 太郎8        | 8才
        9 | 田中9        | 太郎9        | 9才
       10 | 田中10       | 太郎10       | 10才
       11 | 田中11       | 太郎11       | 11才
       12 | 田中12       | 太郎12       | 12才
       13 | 田中13       | 太郎13       | 13才
       14 | 田中14       | 太郎14       | 14才
・
・
・
       95 | 田中95       | 太郎95       | 95才
       96 | 田中96       | 太郎96       | 96才
       97 | 田中97       | 太郎97       | 97才
       98 | 田中98       | 太郎98       | 98才
       99 | 田中99       | 太郎99       | 99才
      100 | 田中100      | 太郎100      | 100才
```
# VIEWの作成
profileテーブルからVIEWの仮想テーブルを作成する。
全く同じテーブルである必要はなく、条件を絞ったりカラムを絞ったSELECT文で問題ない。

```sql
create view test_view (first_name, last_name)
as
select first_name, last_name, count(*)
from profile
group by first_name, last_name;

-- 結果 --
CREATE VIEW
```

# VIEWの確認
```sql
select * from test_view;

  first_name  |  last_name   | count
--------------+--------------+-------
 田中46       | 太郎46       |     1
 田中73       | 太郎73       |     1
 田中85       | 太郎85       |     1
 田中95       | 太郎95       |     1
 田中56       | 太郎56       |     1
 田中58       | 太郎58       |     1
 田中70       | 太郎70       |     1
 田中69       | 太郎69       |     1
 田中79       | 太郎79       |     1
・
・
・
```

RDBのレコードに順序性はないため、めちゃくちゃに登録されているが特に問題はない

```sql
from test_view;
```
ここでは内部的に

```sql
create view test_view (first_name, last_name)
as
select first_name, last_name, count(*)
from profile
group by first_name, last_name;
```

このviewが実行されている。
それが、この一行を書くだけでこれ以降は何度でも実行することができるということだ。

# VIEWの削除
```sql
DROP VIEW test_view;
```

# 参考
[【PostgreSQL】Explainの見方（analyze , cost , scan , sort）についてのまとめ](https://postgresweb.com/post-4047)
