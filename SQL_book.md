- [SQL ゼロから始めるデータベース操作](#sql-ゼロから始めるデータベース操作)
- [接続](#接続)
- [データベースとSQL](#データベースとsql)
  - [テーブルの変更](#テーブルの変更)
    - [テーブルにカラムを追加する](#テーブルにカラムを追加する)
    - [テーブルのカラムを削除する](#テーブルのカラムを削除する)
    - [テーブル名を変更する](#テーブル名を変更する)
    - [テーブル作成](#テーブル作成)
- [検索の基本 だいぶ難しくなってきたぞおおおおおおおおおおおお](#検索の基本-だいぶ難しくなってきたぞおおおおおおおおおおおお)
  - [SELECT文](#select文)
    - [重複行を省く](#重複行を省く)
  - [論理演算子](#論理演算子)
    - [NOT](#not)
    - [WHEREの強弱](#whereの強弱)
    - [NULLに比較演算子は使えない](#nullに比較演算子は使えない)
  - [集約関数で重複値を除外する](#集約関数で重複値を除外する)
  - [グループ化](#グループ化)
    - [WHEREを使う場合](#whereを使う場合)
  - [複数のGROUP BY](#複数のgroup-by)
  - [集約した結果に条件を指定する: HAVING](#集約した結果に条件を指定する-having)
    - [HAVING vs WHERE](#having-vs-where)
  - [並び替え](#並び替え)
    - [複数のソートキーを利用する](#複数のソートキーを利用する)
  - [練習問題](#練習問題)
- [データの作成/削除/更新/トランザクション](#データの作成削除更新トランザクション)
  - [データの作成](#データの作成)
    - [複数行のINSERT](#複数行のinsert)
    - [DEFAULT](#default)
    - [他のテーブルからコピー](#他のテーブルからコピー)
  - [DELETE](#delete)
    - [テーブルの中身を空にする](#テーブルの中身を空にする)
    - [条件指定](#条件指定)
  - [データの更新](#データの更新)
  - [複数列の更新](#複数列の更新)
- [TRANSACTION](#transaction)
  - [COMMIT](#commit)
  - [ROLLBACK](#rollback)
  - [ACID特性](#acid特性)
    - [原子性](#原子性)
    - [一貫性](#一貫性)
  - [独立性](#独立性)
  - [永続性](#永続性)
- [複雑な問合せ](#複雑な問合せ)
  - [View](#view)
  - [viewとtable](#viewとtable)
  - [メリット](#メリット)
  - [作り方](#作り方)
  - [制限事項](#制限事項)
    - [ORDER BYは使えない](#order-byは使えない)
    - [UPDATE VIEW](#update-view)
    - [View削除](#view削除)
  - [サブクエリ(非相関サブクエリ) (the most complicated part)](#サブクエリ非相関サブクエリ-the-most-complicated-part)
  - [スカラサブクエリ](#スカラサブクエリ)
    - [一つの行しか消すことができない](#一つの行しか消すことができない)
    - [WHEREに書くケース](#whereに書くケース)
  - [相関サブクエリ](#相関サブクエリ)
  - [各種サブクエリの違い](#各種サブクエリの違い)
    - [サブクエリ](#サブクエリ)
    - [スカラサブクエリ](#スカラサブクエリ-1)
    - [相関サブクエリ](#相関サブクエリ-1)
- [Function・Predicate・CASE statement](#functionpredicatecase-statement)
  - [IN](#in)
    - [IN with subquery](#in-with-subquery)
    - [NOT IN](#not-in)
    - [IN vs EXIST](#in-vs-exist)
  - [CASE statenent](#case-statenent)
  - [set operations](#set-operations)
    - [UNION](#union)
      - [UNION ALL](#union-all)
      - [UNION INTERSECT](#union-intersect)
      - [UNION EXCEPT](#union-except)
  - [JOIN](#join)
  - [EXISTS vs JOIN](#exists-vs-join)
    - [INNER JOIN](#inner-join)
    - [OUTER JOIN](#outer-join)
- [Execute advanced SQL (second the most complicated part)](#execute-advanced-sql-second-the-most-complicated-part)
  - [Window functions](#window-functions)
    - [RANK function](#rank-function)
  - [Aggregate functions as window functions](#aggregate-functions-as-window-functions)
  - [GROUPING operator](#grouping-operator)
    - [ROLLUP](#rollup)
  - [cube](#cube)
  - [grouping function](#grouping-function)
- [質問](#質問)
  - [【解決】ソートキー](#解決ソートキー)
  - [【解決】なぜトランザクション途中でも更新してしまう？](#解決なぜトランザクション途中でも更新してしまう)
  - [サブクエリと相関サブクエリの違い](#サブクエリと相関サブクエリの違い)
    - [サブクエリ](#サブクエリ-1)
    - [スカラサブクエリ](#スカラサブクエリ-2)
    - [相関サブクエリ](#相関サブクエリ-2)
  - [相関サブクエリの使い道](#相関サブクエリの使い道)
  - [VIEW vs subquery](#view-vs-subquery)
  - [Postgres以外のDBMSは原子性と一貫性矛盾してる](#postgres以外のdbmsは原子性と一貫性矛盾してる)
  - [TRUNCATEは使う？](#truncateは使う)
  - [サブクエリを使う場面](#サブクエリを使う場面)
  - [【重要】相関関数 on SELECT](#重要相関関数-on-select)
  - [EXISTSの方がINより早い証明ができていない](#existsの方がinより早い証明ができていない)
  - [どうやってbuffer shared hitをあげるのか？？](#どうやってbuffer-shared-hitをあげるのか)
- [参考](#参考)
- [words](#words)
# SQL ゼロから始めるデータベース操作
これまでに知らなかった知識をメモしていく

# 接続
```bash
# postgresというデフォルトユーザーになる
$ sudo -i -u postgres
# ログイン
$ psql
# DB作成
$ create database <database_name>
# select database
$ \c <detabase_name> 
# show all tables
$ \dt
```

# データベースとSQL
## テーブルの変更
すでにテーブルを作成した後にカラムを削除・追加したいなど思ったら、再作成する必要はない


### テーブルにカラムを追加する
```
ALTER TABLE <table_name> ADD COLUMN <column_name>
```
### テーブルのカラムを削除する
```
ALTER TABLE <table_name> DROP COLUMN <column_name>
```

### テーブル名を変更する
```
ALTER TABLE <table_name> RENAME TO <new_table_name>
```

### テーブル作成
スペースやコンマなど気をつける部分が多い


```
CREATE TABLE address
(number INT NOT NULL,
name VARCHAR(128) NOT NULL,
address VARCHAR(256) NOT NULL,
tel_num CHAR(10),
mail_address CHAR(20),
PRIMARY KEY (number));
CREATE TABLE
```

# 検索の基本 だいぶ難しくなってきたぞおおおおおおおおおおおお
## SELECT文
### 重複行を省く
```
SELECT DISTINCT categories FROM shohin 
```

こうすることで商品テーブルのカテゴリー全行が出力されるのではなく、種類文だけ出力される

※NULLも１種類としてみなされる

## 論理演算子
### NOT
```
SELECT * FROM shohin WHERE NOT PRICE >= 1000
```

こんな感じでWHEREの直後にNOT入れる

### WHEREの強弱

ANDはORよりも優先される

```
SELECT * 
FROM shohin
WHERE price >= 1000
OR category = books
AND color = red
```

上記のようなSQL文の場合

赤色のbooksもしくは1000円以上であれば条件に当てはまる。

ORを優先させたい場合は下記のようにする

```
SELECT * 
FROM shohin
WHERE ( price >= 1000
OR category = books )
AND color = red
```

### NULLに比較演算子は使えない

```
SELECT * 
FROM shohin
WHERE price = NULL
```

```
SELECT * 
FROM shohin
WHERE price <> NULL
```

はテーブルがどのようなレコードでもエラーとなる。

NULLに対しては

```
SELECT * 
FROM shohin
WHERE price IS NULL
```

```
SELECT * 
FROM shohin
WHERE price IS NOT NULL
```

としなければならない

## 集約関数で重複値を除外する

```
shop=# select * from Shohin;
 shohin_id |   shohin_mei   | hanbai_tanka | shiire_tanka |  torokubi
-----------+----------------+--------------+--------------+------------
         1 | Tシャツ        |          100 |         1000 | 2000-01-20
         2 | パーカー       |          300 |         2000 | 2000-01-22
         3 | パンツ         |          200 |         3000 |
         4 | 上着           |          222 |              | 2000-03-22
         5 | 上着           |          222 |              | 2000-03-22
(5 rows)

shop=# select sum(hanbai_tanka) from Shohin;
 sum
------
 1044
(1 row)

shop=# select sum(distinct hanbai_tanka) from Shohin;
 sum
-----
 822
(1 row)
```

## グループ化
集計関数は通常一行だけ出力するものだが、

GROUP BYをつけることで、指定したグループごとの集計を行うことができる

ex)

```
 shohin_id |   shohin_mei   | hanbai_tanka | shiire_tanka |  torokubi
-----------+----------------+--------------+--------------+------------
         1 | Tシャツ        |          100 |         1000 | 2000-01-20
         2 | パーカー       |          300 |         2000 | 2000-01-22
         3 | パンツ         |          200 |         3000 |
         4 | 上着           |          222 |              | 2000-03-22
         5 | 上着           |          222 |              | 2000-03-22
         
shop=# select shohin_mei, count(*)
shop-# from Shohin
shop-# group by shohin_mei
shop-# ;
   shohin_mei   | count
----------------+-------
 パーカー       |     1
 Tシャツ        |     1
 パンツ         |     1
 上着           |     2
(4 rows)

shop=# select count(*)
from Shohin
;
 count
-------
     5
(1 row)
```

### WHEREを使う場合
先にWHEREで絞られたレコードの中で上のように、グループ化され、集計が行われる

**FROM → WHERE → GROUP BY → SELECT**

## 複数のGROUP BY

```bash
Table: Subject_Selection

+---------+----------+----------+
| Subject | Semester | Attendee |
+---------+----------+----------+
| ITB001  |        1 | John     |
| ITB001  |        1 | Bob      |
| ITB001  |        1 | Mickey   |
| ITB001  |        2 | Jenny    |
| ITB001  |        2 | James    |
| MKB114  |        1 | John     |
| MKB114  |        1 | Erica    |
+---------+----------+----------+
# When you use a group by on the subject column only; say:

select Subject, Count(*)
from Subject_Selection
group by Subject
You will get something like:

+---------+-------+
| Subject | Count |
+---------+-------+
| ITB001  |     5 |
| MKB114  |     2 |
+---------+-------+
#...because there are 5 entries for ITB001, and 2 for MKB114

# If we were to group by two columns:

select Subject, Semester, Count(*)
from Subject_Selection
group by Subject, Semester
we would get this:

+---------+----------+-------+
| Subject | Semester | Count |
+---------+----------+-------+
| ITB001  |        1 |     3 |
| ITB001  |        2 |     2 |
| MKB114  |        1 |     2 |
+---------+----------+-------+
```

([REF](https://stackoverflow.com/questions/2421388/using-group-by-on-multiple-columns))

## 集約した結果に条件を指定する: HAVING
**FROM → WHERE → GROUP BY → HAVING → SELECT**

```bash
select shohin_mei, count(*)
from Shohin
group by shohin_mei

   shohin_mei   | count
----------------+-------
 パーカー        |     1
 Tシャツ         |     1
 パンツ          |     1
 上着           |     2
```

このようにして集約した結果に対してcount=1のものだけ出力したいという場合は**HAVING**を使う必要がある

ポイントとしては
- WHEREには「行に対する条件」、HAVINGには「集約関数を実行するグループに対する条件」を指定する
- HAVINGはGROUP BYの後ろにかく
- 集約関数をかけるのはSELECT, HAVING, ORDER BYのみ

ex)
```bash
select shohin_mei, count(*)
from Shohin
group by shohin_mei
having count(*) = 1;

   shohin_mei   | count
----------------+-------
 パーカー        |     1
 Tシャツ         |     1
 パンツ          |     1
```

### HAVING vs WHERE
**グループ化されたテーブルは元のレコードではないため、WHEREでレコードを指定できない**

**→ グループ化されたテーブルに対する条件指定は問答無用でHAVING**

理由としては
- ただの行に対する条件はWHEREの方が機能に沿った動きのため、第三者も何をしているかわかりやすい
- WHEREは条件で指定する列にインデックスを作ることで、処理を大幅に高速化できる
- HAVINGよりWHEREの方が実行速度が早い

WHEREの前に負担の大きなソートをするが、逆にHAVINGの後にソートをするため


## 並び替え
**FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY**

という順番で実行されるため、SELECTの後であり、別名を使うこともできる

**集約関数も使える**

### 複数のソートキーを利用する

```
 shohin_id |   shohin_mei   | hanbai_tanka | shiire_tanka |  torokubi
-----------+----------------+--------------+--------------+------------
         1 | Tシャツ        |          100 |         1000 | 2000-01-20
         2 | パーカー       |          300 |         2000 | 2000-01-22
         3 | パンツ         |          200 |         3000 |
         4 | 上着           |          223 |              | 2000-03-22
         5 | 上着           |          222 |              | 2000-03-22

SELECT * 
FROM Shohin
ORDER BY hanbai_tanka, shohin_id

 shohin_id |   shohin_mei   | hanbai_tanka | shiire_tanka |  torokubi
-----------+----------------+--------------+--------------+------------
         1 | Tシャツ        |          100 |         1000 | 2000-01-20
         3 | パンツ         |          200 |         3000 |
         4 | 上着           |          222 |              | 2000-03-22
         5 | 上着           |          222 |              | 2000-03-22
         2 | パーカー       |          300 |         2000 | 2000-01-22
         

```

※ ソートキーがNULLの場合は一番上か下のレコードになる

## 練習問題
要復習。激ムズ。理解はできた。

# データの作成/削除/更新/トランザクション
## データの作成
ポイント
- 原則１レコード１INSERT文
- NULLの場合はNULLと記載する

### 複数行のINSERT
```
insert into Shohin 
values (0005, '上着', 222,NULL, '2000-03-22' ),
(0005, '上着', 222,NULL, '2000-03-22' ),
(0005, '上着', 222,NULL, '2000-03-22' ),
(0005, '上着', 222,NULL, '2000-03-22' );
```
### DEFAULT
```bash
create table Shohin(
shohin_id INT NOT NULL,
shohin_mei CHAR(10) NOT NULL,
hanbai_tanka INT,
# このようにDEFAULTを設定する
shiire_tanka INT DEFAULT 400,
# How to set PRIMARY KEY
torokubi DATE, PRIMARY KEY (shohin_id))
```

### 他のテーブルからコピー
No need to use COPY statement

```bash
INSERT INTO target_table_name 
# どの行をコピー元から持ってくるかによる
SELECT * 
FROM copied_table_name
```

バックアップを使う際などにも使われる

## DELETE
### テーブルの中身を空にする
```
DELETE FROM Shohin;
```

**※ *などは使えない**

### 条件指定
```
DELETE 
FROM Shohin
WHERE price >= 1000;
```
このように削除もレコードを制限できる

## データの更新
```bash
UPDATE <table_name>
# set values
SET <column_name> = <式>
# select record
WHERE ~~
```

```bash
 shohin_id |   shohin_mei   | hanbai_tanka | shiire_tanka |  torokubi
-----------+----------------+--------------+--------------+------------
         1 | Tシャツ        |          100 |         1000 | 2000-01-20
         2 | パーカー       |          300 |         2000 | 2000-01-22
         3 | パンツ         |          200 |         3000 |
         4 | 上着           |          222 |              | 2000-03-22
         5 | 上着           |          222 |              | 2000-03-22
update Shohin
set shohin_mei = '靴下'
where shohin_mei = 'パンツ';

select * from Shohin;
 shohin_id |   shohin_mei   | hanbai_tanka | shiire_tanka |  torokubi
-----------+----------------+--------------+--------------+------------
         1 | Tシャツ        |          100 |         1000 | 2000-01-20
         2 | パーカー       |          300 |         2000 | 2000-01-22
         4 | 上着           |          222 |              | 2000-03-22
         5 | 上着           |          222 |              | 2000-03-22
         3 | 靴下           |          200 |         3000 |
```

NOT NULLなどの制約がない場合はNULLにすることもできる

## 複数列の更新
```bash
update Shohin
# 複数のフィールドを更新
set shohin_mei = '靴下',
    torokubi = NULL
where shohin_mei = 'パンツ';
```

# TRANSACTION
beginとcommitで囲むことでトランザクションを作ることができる。
## COMMIT

```bash
 shohin_id |   shohin_mei   | hanbai_tanka | shiire_tanka |  torokubi
-----------+----------------+--------------+--------------+------------
         1 | Tシャツ        |          100 |         1000 | 2000-01-20
         2 | パーカー       |          300 |         2000 | 2000-01-22
         4 | 上着           |          222 |              | 2000-03-22
         5 | 上着           |          222 |              | 2000-03-22
         3 | キャップ       |          200 |         3000 |
begin transaction;
BEGIN

update Shohin
set shohin_mei = 'イヤリング'
where shohin_id = 3;
UPDATE 1

update Shohin
set shohin_mei = 'シャツ'
where shohin_id = 2;
UPDATE 1

commit;
COMMIT

select * from Shohin;
 shohin_id |   shohin_mei    | hanbai_tanka | shiire_tanka |  torokubi
-----------+-----------------+--------------+--------------+------------
         1 | Tシャツ         |          100 |         1000 | 2000-01-20
         4 | 上着            |          222 |              | 2000-03-22
         5 | 上着            |          222 |              | 2000-03-22
         3 | イヤリング      |          200 |         3000 |
         2 | シャツ          |          300 |         2000 | 2000-01-22
```

## ROLLBACK
途中で中身を確認すると、変わってしまうが、それは問題ない。

トランザクションで囲むことでロールバックできるため。

```bash
 shohin_id |   shohin_mei    | hanbai_tanka | shiire_tanka |  torokubi
-----------+-----------------+--------------+--------------+------------
         1 | Tシャツ         |          100 |         1000 | 2000-01-20
         4 | 上着            |          222 |              | 2000-03-22
         5 | 上着            |          222 |              | 2000-03-22
         3 | イヤリング      |          200 |         3000 |
         2 | シャツ          |          300 |         2000 | 2000-01-22
begin transaction;
BEGIN

update Shohin
set shohin_mei = 'シャツ'
where shohin_id = 1;
UPDATE 1

select * from Shohin;
 shohin_id |   shohin_mei    | hanbai_tanka | shiire_tanka |  torokubi
-----------+-----------------+--------------+--------------+------------
         4 | 上着            |          222 |              | 2000-03-22
         5 | 上着            |          222 |              | 2000-03-22
         3 | イヤリング      |          200 |         3000 |
         2 | シャツ          |          300 |         2000 | 2000-01-22
         1 | シャツ          |          100 |         1000 | 2000-01-20
(5 rows)

rollback;
ROLLBACK

select * from Shohin;
 shohin_id |   shohin_mei    | hanbai_tanka | shiire_tanka |  torokubi
-----------+-----------------+--------------+--------------+------------
         1 | Tシャツ         |          100 |         1000 | 2000-01-20
         4 | 上着            |          222 |              | 2000-03-22
         5 | 上着            |          222 |              | 2000-03-22
         3 | イヤリング      |          200 |         3000 |
         2 | シャツ          |          300 |         2000 | 2000-01-22
(5 rows)
```

## ACID特性
### 原子性
トランザクションが終わったときは、①全て完了　or ②全て元通り
### 一貫性
制約違反のあるDMLは失敗するが、トランザクション内の他のDMLは実行され、失敗文のみ反映されない

※ Postgresの場合のみ、DMLが一つでも失敗した場合にロールバックされる

## 独立性
トランザクションは見えない状態で、入れ子にもならない

## 永続性
ログを取るなどして、障害が発生しても復旧する手段を持つ必要がある

# 複雑な問合せ

## View
- テーブルのようだが、SELECT文を保存するクリップボードのようなものでデータそのものは格納していない → 元のテーブルが更新されると自動で更新される　
- CREATE VIEWで作成
- DROP VIEWで削除
- テーブルも作成される
- 高速化には使われる

ビューをFROMで使う場合は

①最初にビューで定義されたSELECTが実行

②その結果に対して、ビューをFROMに指定したSQLのSELECTが実行される

## viewとtable
**実際にSELECTすると、テーブルが表示されるのは同じ**

テーブルを作ってINSERTするとそのデータは記憶装置に保存される。

viewはどこにも保存せず、SELECT文があるだけ。呼び出された時にそれを使って仮想のテーブルを作って返す。

## メリット
- 記憶装置の容量を節約できる
- SELECT文を書く手間が省ける

## 作り方
```
CREATE VIEW ビュー名(<作成するビューテーブルの列名1>, <作成するビューテーブルビューの列名2>)
AS 
<SELECT文>
```

## 制限事項
### ORDER BYは使えない
ORDER BYをするならVIEWを呼び出しているオリジナルのSQLで使うべき

### UPDATE VIEW
**更新は可能だが、集約された値かDISTINCTを持つViewは更新できない**

GROUP BYなどで集約された値を持つviewを更新すると
グループ分けする前の元のテーブルとの整合性が保てなくなるため

下記のようなviewなら更新が可能
```bash
CREATE VIEW ShohinJim(shohin_id, shohin_name)
AS
SELECT *
FROM Shohin
WHERE shohin_bunrui = '事務用品'

# ビューに追加
INSERT　INTO ShohinJjim VALUES('0001', '印鑑')
```

**しかしこのVIEWに対するINSERTはShohinJjimテーブルにINSERTすることになるため、ShohinJjimテーブルなどを考慮する必要がある**

### View削除
```
DROP VIEW <ビュー名>
```

## サブクエリ(非相関サブクエリ) (the most complicated part)
使い捨てのVIEWのようなもの

**VIEWはVIEWというテーブルのようなものを別の文で定義したが、サブクエリはその文をオリジナルSQLのFROMのなかに直接記述する**


一度使ったらそのあとすぐに消える点もVIEWと異なる

```bash
SELECT shohin_bunrui, cnt_shohin
  FROM(
    SELECT shohin_bunrui, COUNT(*) AS cnt_shohin
    FROM Shohin
    GROUP BY shohin_bunrui
  ) AS ShohinSUM; # サブクエリ名
```

ex)

```bash
 shohin_id |   shohin_mei    | hanbai_tanka | shiire_tanka |  torokubi
-----------+-----------------+--------------+--------------+------------
         1 | Tシャツ         |          100 |         1000 | 2000-01-20
         4 | 上着            |          222 |              | 2000-03-22
         5 | 上着            |          222 |              | 2000-03-22
         3 | イヤリング      |          200 |         3000 |
         2 | シャツ          |          300 |         2000 | 2000-01-22

SELECT shohin_mei, cnt_shohin
FROM(
  # AS cnt_shohinで元のSELECTで使えるように一時的に保存するイメージ
 SELECT shohin_mei, COUNT(*) AS cnt_shohin
 FROM Shohin
 GROUP BY shohin_mei
) as test;
   shohin_mei    | cnt_shohin
-----------------+------------
 イヤリング      |          1
 シャツ          |          1
 Tシャツ         |          1
 上着            |          2
```

サブクエリ名は付けないとダメな仕様

## スカラサブクエリ
集約した数値など何か一つを返すverのサブクエリ

**複数行のレコードなどを返してはいけない→GROUP BYを使うなどすると複数行になるのでその時は相関サブクエリを利用する**

**つまり=や><などの比較演算子などと使われることが多い**

WHERE句で集約関数を使えないため、そこでスカラサブクエリが使われることが多い。

GROUP BYやHAVINGなど**定数や列名をかけるところ**であればどこでも使える

### 一つの行しか消すことができない

```bash
$ select * from friends;
 profile_id |   color    |      name       | age
------------+------------+-----------------+-----
          1 | yellow     | yuto            |  24
          2 | red        | tosa            |  41
          3 | yellow     | ilji            |  21
          4 | jenny      | deji            |  32
          3 | red        | herry           |  23
(5 rows)

# これならグループごとに出力されず、一行のみ返されるから問題なし
$select color, (select avg(age) 
               from friends)
from friends;
   color    |         avg
------------+---------------------
 yellow     | 28.2000000000000000
 red        | 28.2000000000000000
 yellow     | 28.2000000000000000
 jenny      | 28.2000000000000000
 red        | 28.2000000000000000
(5 rows)

# これではカラーごとに合計３行返されるからエラーとなる
$ select color, (select avg(age) 
               from friends 
              group by color)
from friends;
ERROR:  more than one row returned by a subquery used as an expression

```

### WHEREに書くケース
**WHEREには集約関数を書くことができない。** Instead, スカラサブクエリを使う。



```bash
SELECT shohin_id, shohin_mei, shohin_tanka
FROM Shohin
# WHEREでは集約関数使えない
WHERE hanbai_tanka > AVG(hanbai_tanka);

↓

SELECT shohin_id, shohin_mei, shohin_tanka
FROM Shohin
# SELECTでは集約関数を使えるため、スカラサブクエリで無理やりSELECTを使う
WHERE hanbai_tanka > (SELECT AVG(hanbai_tanka
                      FROM Shohin);
```



## 相関サブクエリ
**サブクエリの中で、WHERE句を使用して、その条件としてサブクエリの外の値を参照すること**

**テーブル全体ではなくテーブルの一部のレコード集合に限定した比較を行う**

一般的には、パフォーマンスは悪くなります。

ex)shohin_bunruiごとの平均を算出し、shohin_bunruiごとの平均kakaku以上のkakakuを持つshohin_meiを取り出したい
![](2022-01-20-13-37-22.png)

```bash
SELECT shohin_mei, kakaku
  FROM shohin AS S1
WHERE (SELECT AVG(kakaku)
         FROM shohin AS S2
       WHERE S1.shohin_bunrui = S2.shohin_bunrui
       GROUP BY shohin_bunrui) < kakaku;
```
https://gyroibaraki.com/sql-correlated-subquery/

## 各種サブクエリの違い
### サブクエリ
二重の計算を行う時
二段階以上の集計を行う場合
具体的には下のようなクエリです。学生一覧からクラスの平均在籍数と最小/最大在籍数を算出します。

```
SELECT AVG(`count`), MIN(`count`), MAX(`count`) FROM (
  SELECT COUNT(*) AS `count` FROM `students` GROUP BY `class`
) AS sub;
```

そもそもサブクエリはかテーブルのようなものを作ってオリジナルテーブルがそこからFROMで引っ張ってくる。だから=,>,<などを使う対象ではない。
対して下記の二つはスカラ(=,>,<)と共に使う。
### スカラサブクエリ
単純な計算で、これ全部のレコードの平均年齢は？とか。カテゴリごとはダメかな。WHEREで使うことが多い
=,>,<などと使う
### 相関サブクエリ
=,>,<などと使うときに、複数行返す場合。
スカラサブクエリは１行しか返せないのに複数行返すことになる時に使う。
GROUP BYして複数グループあるとき複数行返してしまう。


select id mei bunrui tnka avg(tanka)

# Function・Predicate・CASE statement
Regarding other functions, plz read books.

The following functions are what I feel difficult to understand.

## IN
This can be used instead of OR

ex)
```bash
WHERE shiire_tanka = 320
OR shiire_tanka = 320
OR shiire_tanka = 320
# instead...
WHERE shiire_tanka IN (320, 200, 100)
```

### IN with subquery
This can be useful When you wanna get some record from other table.

```bash
 profile_id |   color    |      name       | address_id
------------+------------+-----------------+------------
          1 | yellow     | yuto            |          1
          2 | red        | tosa            |          1
          3 | yellow     | ilji            |          3
          4 | jenny      | deji            |          2
          3 | red        | herry           |          2

select * from address;
 id | prefecture
----+------------
  1 | kanagawa
  2 | tokyo
  3 | chiba

select *
from ppl
where address_id IN (select id
                     from address
                     where id = 1);
 profile_id |   color    |      name       | address_id
------------+------------+-----------------+------------
          1 | yellow     | yuto            |          1
          2 | red        | tosa            |          1
```

### NOT IN
**You must not use IN with NULL**

That returns nothing at any time

### IN vs EXIST
>The Exists keyword evaluates true or false, but the IN keyword will compare all values in the corresponding subuery column.  If you are using the IN operator, the SQL engine will scan all records fetched from the inner query. On the other hand, if we are using EXISTS, the SQL engine will stop the scanning process as soon as it found a match.
The EXISTS subquery is used when we want to display all rows where we have a matching column in both tables.  In most cases, this type of subquery can be re-written with a standard join to improve performance.

The EXISTS clause is much faster than IN when the subquery results is very large. 
Conversely, the IN clause is faster than EXISTS when the subquery results is very small.

reference: SQL Exists vs. [IN clause](http://www.dba-oracle.com/t_exists_clause_vs_in_clause.htm#:~:text=The%20Exists%20keyword%20evaluates%20true,fetched%20from%20the%20inner%20query.)


## CASE statenent
The CASE statement goes through conditions and returns a value when the first condition is met (like an if-then-else statement). So, once a condition is true, it will stop reading and return the result. If no conditions are true, it returns the value in the ELSE clause.

If there is no ELSE part and no conditions are true, it returns NULL.

```bash
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    WHEN conditionN THEN resultN
    ELSE result
END;
```

**This runs on each record**

ex)
```bash
select * from friends;
 profile_id |   color    |      name       | age
------------+------------+-----------------+-----
          1 | yellow     | yuto            |  24
          2 | red        | tosa            |  41
          3 | yellow     | ilji            |  21
          4 | jenny      | deji            |  32
          3 | red        | herry           |  23
          5 |            |                 |  20

select color,name,
case when age >= 30
    then 'おじさん'
    when age < 30
    then '若者'
    else null
end as test
from friends;
   color    |      name       |   test
------------+-----------------+----------
 yellow     | yuto            | 若者
 red        | tosa            | おじさん
 yellow     | ilji            | 若者
 jenny      | deji            | おじさん
 red        | herry           | 若者
```

![](2022-01-20-15-21-56.png)


## set operations
This combine multiple records from multiple tables.
### UNION
The UNION operator is used to combine the result-set of two or more SELECT statements.
- Every SELECT statement within UNION must have the same number of columns
- The columns must also have similar data types
- The columns in every SELECT statement must also be in the same order

**If there's completely same records in two tables, only one records is shown**

```bash
# basketball_team1
 member_id |  position  | profile_id
-----------+------------+------------
         1 | center     |         42
         2 | foward     |         21
         3 | guard      |         35
         4 | center     |         78
         5 | foward     |         98

# basketball_team2
 member_id |  position  | profile_id
-----------+------------+------------
         1 | center     |         42
         2 | foward     |         21
         3 | guard      |         35
         4 | center     |         78
         6 | center     |         42
         7 | foward     |         21
         8 | guard      |         35
         9 | center     |         78
        10 | foward     |         98

$ select * from basketball_team
union
select * from basketball_team2;
# records which have member_id 3,4 in basketball_team2 isn't shown.
 member_id |  position  | profile_id
-----------+------------+------------
         5 | foward     |         98
        10 | foward     |         98
         2 | foward     |         21
         7 | foward     |         21
         3 | guard      |         35
         8 | guard      |         35
         6 | center     |         42
         1 | center     |         42
         9 | center     |         78
         4 | center     |         78
```

```A + B - common_part```

#### UNION ALL
The UNION operator selects only distinct values by default. To allow duplicate values, use UNION ALL:

**If there's completely same records in two tables, both records is shown**

```A + B```

```bash
 select * from basketball_team
union all
select * from basketball_team2;
 member_id |  position  | profile_id
-----------+------------+------------
         1 | center     |         42
         2 | foward     |         21
         3 | guard      |         35
         4 | center     |         78
         5 | foward     |         98
         6 | center     |         42
         7 | foward     |         21
         8 | guard      |         35
         9 | center     |         78
        10 | foward     |         98
         4 | center     |         42
         3 | foward     |         21
         1 | center     |         42
         2 | foward     |         21
(14 rows)
```

#### UNION INTERSECT
```common_part```

```bash
select * from basketball_team
intersect
select * from basketball_team2;
 member_id |  position  | profile_id
-----------+------------+------------
         1 | center     |         42
         2 | foward     |         21
```

#### UNION EXCEPT
```A - common_part```

```
select * from basketball_team
except
select * from basketball_team2;
 member_id |  position  | profile_id
-----------+------------+------------
         5 | foward     |         98
         4 | center     |         78
         3 | guard      |         35
(3 rows)
```

## JOIN

## EXISTS vs JOIN
Both help us to get info from other tables. 
However...

- **Use JOIN to output info from other tables**
- **Use EXISTS or IN to use info from other tables for condition and at last only records from one table will be shown**

### INNER JOIN
Only records both tables have is gonna output

```bash
select * from friends;
 profile_id |   color    |      name       | age
------------+------------+-----------------+-----
          1 | yellow     | yuto            |  24
          2 | red        | tosa            |  41
          3 | yellow     | ilji            |  21
          4 | jenny      | deji            |  32
          5 |            |                 |  20
          6 | red        | herry           |  23

select * from basketball_team;
 member_id |  position  | profile_id
-----------+------------+------------
         1 | center     |          1
         2 | foward     |          2
         3 | guard      |          3
         4 | center     |          4
         5 | foward     |          5

select *
from basketball_team
inner join friends
on basketball_team.profile_id=friends.profile_id
where friends.age > 30;
 member_id |  position  | profile_id | profile_id |   color    |      name       | age
-----------+------------+------------+------------+------------+-----------------+-----
         2 | foward     |          2 |          2 | red        | tosa            |  41
         4 | center     |          4 |          4 | jenny      | deji            |  32
```

### OUTER JOIN
All of records master tables has and records both tables have is gonna output.

```bash
select * from friends;
 profile_id |   color    |      name       | age
------------+------------+-----------------+-----
          1 | yellow     | yuto            |  24
          2 | red        | tosa            |  41
          3 | yellow     | ilji            |  21
          4 | jenny      | deji            |  32
          5 |            |                 |  20
          6 | red        | herry           |  23

select * from basketball_team;
 member_id |  position  | profile_id
-----------+------------+------------
         1 | center     |          1
         2 | foward     |          2
         3 | guard      |          3
         4 | center     |          4
         5 | foward     |          5

# inner join
select *
from basketball_team
inner join friends
on basketball_team.profile_id=friends.profile_id;
 member_id |  position  | profile_id | profile_id |   color    |      name       | age
-----------+------------+------------+------------+------------+-----------------+-----
         1 | center     |          1 |          1 | yellow     | yuto            |  24
         2 | foward     |          2 |          2 | red        | tosa            |  41
         3 | guard      |          3 |          3 | yellow     | ilji            |  21
         4 | center     |          4 |          4 | jenny      | deji            |  32
         5 | foward     |          5 |          5 |            |                 |  20
(5 rows)

# outer join
select *
from basketball_team
# This is the only difference
# right table(friends) is gonna be a master table.
# → profile_id on friends table(10) doesn't match any record on basketball_team table but that record is shown
right outer join friends
on basketball_team.profile_id=friends.profile_id;
 member_id |  position  | profile_id | profile_id |   color    |      name       | age
-----------+------------+------------+------------+------------+-----------------+-----
         1 | center     |          1 |          1 | yellow     | yuto            |  24
         2 | foward     |          2 |          2 | red        | tosa            |  41
         3 | guard      |          3 |          3 | yellow     | ilji            |  21
         4 | center     |          4 |          4 | jenny      | deji            |  32
         5 | foward     |          5 |          5 |            |                 |  20
           |            |            |          6 | red        | herry           |  23
(6 rows)
```

# Execute advanced SQL (second the most complicated part)
## Window functions
- **The most import things to say is you can use windows functions only in SELECT.**
- **ORDER BY must be urtilized in OVER**
- **This needs partition to add new column**

Window functions are to cut tables into window and order it in some way.

### RANK function
This has a function for PARTITION and ORDER.

However partition is not required. Let's break down how it's used.

```bash
 shohin_id |   shohin_mei    | hanbai_tanka | shiire_tanka |  torokubi
-----------+-----------------+--------------+--------------+------------
         1 | Tシャツ         |          100 |         1000 | 2000-01-20
         4 | 上着            |          222 |              | 2000-03-22
         5 | 上着            |          222 |              | 2000-03-22
         3 | イヤリング      |          200 |         3000 |
         2 | シャツ          |          300 |         2000 | 2000-01-22

select shohin_mei, hanbai_tanka,
rank () over (partition by shohin_mei
            order by hanbai_tanka)
from Shohin;


   shohin_mei    | hanbai_tanka | rank
-----------------+--------------+------
 シャツ          |          300 |    1
 イヤリング      |          200 |    1
 Tシャツ         |          100 |    1
 上着            |          100 |    1
 上着            |          500 |    2
(5 rows)
```
## Aggregate functions as window functions

the running total sums across the current row and all previous rows of duration_seconds. Scroll down until the start_terminal value changes and you will notice that running_total starts over


ex)
```bash
 shohin_id |   shohin_mei    | hanbai_tanka | shiire_tanka |  torokubi
-----------+-----------------+--------------+--------------+------------
         1 | Tシャツ         |          100 |         1000 | 2000-01-20
         3 | イヤリング      |          200 |         3000 |
         2 | シャツ          |          300 |         2000 | 2000-01-22
         5 | 上着            |          500 |              | 2000-03-22
         4 | 上着            |          100 |              | 2000-03-22

select shohin_mei, hanbai_tanka,
sum(hanbai_tanka) over (order by shohin_id
# including current one, 2 preceding record is the target 
                        rows 2 preceding) as test
from Shohin;

  shohin_mei    | hanbai_tanka | test
-----------------+--------------+------
 Tシャツ         |          100 |  100
 シャツ          |          300 |  400
                               # 100+300+200
 イヤリング      |          200 |  600
 上着            |          100 |  600
 上着            |          500 |  800
```

**Aggregate functions as window functions can be used only in SELECT since WINDOW functions runs after WHERE, GROUP BY.**

## GROUPING operator
By using GROUP BY, you cannot express total_line(total_cost of each categories + total of these)

GROUPING is for solving this trouble.

Here's three ways to solve this.
- ROLLUP
- CUBE
- GROUPING SET

Let's see how those works!


### ROLLUP

```bash
select * from friends;
 profile_id |   color    |      name       | age
------------+------------+-----------------+-----
          1 | yellow     | yuto            |  24
          2 | red        | tosa            |  41
          6 | red        | herry           |  23
          5 | black      |                 |  20
          4 | black      | deji            |  20
          3 | yellow     | ilji            |  24

# Not using rollup
select color, round(avg(age),1) as avg_age
from friends
group by color;
   color    | avg_age
------------+---------
 black      |    20.0
 yellow     |    24.0
 red        |    32.0

# using rollup
select color, round(avg(age),1) as avg_age
from friends
group by rollup(color);
   color    | avg_age
------------+---------
# total of 3 records
            |    25.3
 black      |    20.0
 yellow     |    24.0
 red        |    32.0
(4 rows)

# add multiple columns on GROUP BY clause
select color, age, round(avg(age),1) as avg_age
from friends
group by rollup(color, age)
order by color;
   color    | age | avg_age
------------+-----+---------
 black      |  20 |    20.0
# total of all black
 black      |     |    20.0
 red        |  41 |    41.0
 red        |  23 |    23.0
#  total of all red
 red        |     |    32.0
 yellow     |  24 |    24.0
#  total of yellow
 yellow     |     |    24.0
#  total of all records
            |     |    25.3
```
the number of red records are more than other cuz two of red records have different age though yellow and black records have same age.

## cube
how to use cube is totally same as rollup.
However the number of output is more than that.

```bash
select color, age, round(avg(age),1) as avg_age
from friends
group by cube(color, age)
order by color;
   color    | age | avg_age
------------+-----+---------
# total of black
 black      |     |    20.0
# total of black and 20
 black      |  20 |    20.0
# total of red and 23
 red        |  23 |    23.0
# total of red
 red        |     |    32.0
# total of red and 41
 red        |  41 |    41.0
# total of yellow and 24
 yellow     |  24 |    24.0
# total of yellow
 yellow     |     |    24.0
# 5 rows below is from table below
# total of 23
            |  23 |    23.0
# total of 41
            |  41 |    41.0
# total of 24
            |  24 |    24.0
# total of 20
            |  20 |    20.0
#  total of all records 
            |     |    25.3
(12 rows)

# 5 rows below is added to the above table
select age, round(avg(age),1) as avg_age
from friends
group by cube(age);
 age | avg_age
-----+---------
     |    25.3
  41 |    41.0
  24 |    24.0
  20 |    20.0
  23 |    23.0
(5 rows)
```
## grouping function
this function is to identify if null is real null or superset null

if argument is superset null, that returns 1, if not that returns 0

Exmaples using this functions is below

```bash
select
  case when grouping(color) = 1
  then '全色合計'
  else color end 
  as test,
round(avg(age),1) as color
from friends
group by rollup(color);

    test    | color
------------+-------
 全色合計   |  25.3
 black      |  20.0
 yellow     |  24.0
 red        |  32.0
```




# 質問
## 【解決】ソートキー
dynamoDBでソートキーをGUIで設定した気がするが、それは何か。
毎回のSQL文の中で設定するわけじゃないのか？並び替える基準のものって、、

あ、毎回一緒か。並び替えるというか、何もしないと順番はランダムだからどれを基準にレコード並べるか指定する必要がある
→いやこれは違う。それなら順番入れ替わったり追加されるたびにかなり負荷かかることになる。



## 【解決】なぜトランザクション途中でも更新してしまう？
本来はトランザクションは更新されず、commit時にデータベースに反映される認識。それじゃないと、毎回更新のたびに負荷かかる。
参考: [【DB】同じトランザクション内でupdateとselectをしたときの結果値](https://oshiete.goo.ne.jp/qa/9803035.html)
**トランザクションを行っているユーザーからはそうは見えるが、実際に更新されるのはトランザクション完了した時点。**

トランザクション中に他のユーザーがselectで確認すると更新前の情報しか表示されない

## サブクエリと相関サブクエリの違い
相関サブクエリでしていることはサブクエリで代用できる気がする
→P171の一番下のコードはなくてもサブクエリとして通じない？？？

### サブクエリ
二重の計算を行う時
二段階以上の集計を行う場合
具体的には下のようなクエリです。学生一覧からクラスの平均在籍数と最小/最大在籍数を算出します。

```
SELECT AVG(`count`), MIN(`count`), MAX(`count`) FROM (
  SELECT COUNT(*) AS `count` FROM `students` GROUP BY `class`
) AS sub;
```

そもそもサブクエリはかテーブルのようなものを作ってオリジナルテーブルがそこからFROMで引っ張ってくる。だから=,>,<などを使う対象ではない。
対して下記の二つはスカラ(=,>,<)と共に使う。
### スカラサブクエリ
単純な計算で、これ全部のレコードの平均年齢は？とか。カテゴリごとはダメかな。WHEREで使うことが多い
=,>,<などと使う
### 相関サブクエリ
=,>,<などと使うときに、複数行返す場合。
スカラサブクエリは１行しか返せないのに複数行返すことになる時に使う。
GROUP BYして複数グループあるとき複数行返してしまう。

## 相関サブクエリの使い道


## VIEW vs subquery
何度も使うならVIEW, 一度だけならsubquery。

一度だけ使うときにsubqueryの理由は？速度？コード量？

## Postgres以外のDBMSは原子性と一貫性矛盾してる

## TRUNCATEは使う？
レコードを全件削除にはこれも使えて、DELETEより高速な処理で時間を短縮できるらしいが、使うことはあるのか？

## サブクエリを使う場面
他のテーブルとか？スカラサブクエリはWHEREでavgとか使いたい時に使うのわかるが。
```
--サブクエリ使う場合--
select first_name, last_name, count
from (select first_name, last_name, count(*) as count
from customer
group by first_name, last_name) as test;

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

--サブクエリ使わない場合--
select first_name, last_name, count(*)
from customer
group by first_name, last_name;

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
 田中3        | 太郎3        |     1
```


[SQLでサブクエリを上手に使う6パターン](https://medium.com/nyle-engineering-blog/sql%E3%81%A7%E3%82%B5%E3%83%96%E3%82%AF%E3%82%A8%E3%83%AA%E3%82%92%E4%B8%8A%E6%89%8B%E3%81%AB%E4%BD%BF%E3%81%866%E3%83%91%E3%82%BF%E3%83%BC%E3%83%B3-bae5ae35b962)で出てくるように

> 二段階以上の集計を行う場合
具体的には下のようなクエリです。学生一覧からクラスの平均在籍数と最小/最大在籍数を算出します。
```
SELECT AVG(`count`), MIN(`count`), MAX(`count`) FROM (
  SELECT COUNT(*) AS `count` FROM `students` GROUP BY `class`
) AS sub;
```

## 【重要】相関関数 on SELECT
各カラーごとの平均年齢を知りたい

```bash
# なぜこれがダメ？where追加してるからいいという認識
select *, (select avg(age)
                           from friends as S2
                           WHERE S1.profile_id = S2.profile_id
                           group by color) as sub
from friends as S1;
ERROR:  more than one row returned by a subquery used as an expression

select *, (select avg(age)
                           from friends as S2
                           WHERE S1.profile_id = S2.profile_id
                           ) as sub
from friends as S1;
# これはグループごとになっていないからやりたいことと違う

 profile_id |   color    |      name       | age |         sub
------------+------------+-----------------+-----+---------------------
          1 | yellow     | yuto            |  24 | 24.0000000000000000
          2 | red        | tosa            |  41 | 41.0000000000000000
          3 | yellow     | ilji            |  21 | 22.0000000000000000
          4 | jenny      | deji            |  32 | 32.0000000000000000
          3 | red        | herry           |  23 | 22.0000000000000000
(5 rows)


```

## EXISTSの方がINより早い証明ができていない
他ファイル参照
→NOT EXISTS, NOT INを使うのとデータもっと増やす

## どうやってbuffer shared hitをあげるのか？？
どうやって初期化パラメータファイル（init.ora）の値を変更し、データベース・バッファ・キャッシュのサイズを変更すればいいのか？


# 参考
[チューニングなどの事例](https://oreno-it.info/archives/category/oracle/oracle-sql)
[SQLを高速化するコツ・テクニック](https://style.potepan.com/articles/26070.html)
[SQL CASE](https://www.w3schools.com/sql/sql_case.asp)

# words
- clause: 節 like WHERE, SELECT, JOIN
- statement: select * from test; ([ref](https://stackoverflow.com/questions/15629550/difference-between-sql-statements-and-clause))