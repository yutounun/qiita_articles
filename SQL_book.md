- [SQL ゼロから始めるデータベース操作](#sql-ゼロから始めるデータベース操作)
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
  - [集約した結果に条件を指定する](#集約した結果に条件を指定する)
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
- [質問](#質問)
  - [ソートキー](#ソートキー)
  - [TRUNCATEは使う？](#truncateは使う)
# SQL ゼロから始めるデータベース操作
これまでに知らなかった知識をメモしていく

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

## 集約した結果に条件を指定する

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
```
select shohin_mei, count(*)
from Shohin
group by shohin_mei
having shohin_mei = 'パンツ';

  shohin_mei   | count
---------------+-------
 パンツ        |     1
```

HAVINGでもこのように集約キー(shohin_mei)に対しては条件文を書ける

**ただし、条件文の時はHAVINGよりもWHEREを使うべき**

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
insert into Shohin values (
0005, '上着', 222,NULL, '2000-03-22' ),
(
0005, '上着', 222,NULL, '2000-03-22' ),
(
0005, '上着', 222,NULL, '2000-03-22' ),
(
0005, '上着', 222,NULL, '2000-03-22' );
```
### DEFAULT
```bash
create table Shohin(
shohin_id INT NOT NULL,
shohin_mei CHAR(10) NOT NULL,
hanbai_tanka INT,
# このようにDEFAULTを設定する
shiire_tanka INT DEFAULT 400,
torokubi DATE, PRIMARY KEY (shohin_id))
```

### 他のテーブルからコピー

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
```
UPDATE <table_name>
SET <column_name> = <式>
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

# 質問
## ソートキー
dynamoDBでソートキーをGUIで設定した気がするが、それは何か。
毎回のSQL文の中で設定するわけじゃないのか？並び替える基準のものって、、

あ、毎回一緒か。並び替えるというか、何もしないと順番はランダムだからどれを基準にレコード並べるか指定する必要がある
→いやこれは違う。それなら順番入れ替わったり追加されるたびにかなり負荷かかることになる。

## TRUNCATEは使う？
レコードを全件削除にはこれも使えて、DELETEより高速な処理で時間を短縮できるらしいが、使うことはあるのか？