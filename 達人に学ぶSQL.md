# goal
Writing down everything I learned takes quite long, so I'll just take easy memos on this doc.

# CASE
## IN vs EXIST
EXIST is better choice since it's quicker

## HAVING vs scala-subquery
- Use HAVING with GROUP BY
- use scala-subquery with others like where

# WINDOWS func
Windowns functions can also have name.

```bash
 sample_date | load_val
-------------+----------
 2018-02-01  |     1024
 2018-02-01  |     1024
 2012-03-21  |     3923
 2032-02-21  |     3822

select sample_date,
       min(sample_date)
           over (order by sample_date asc
            #   if rows 1 preceding, first latest_date will be 2012-03-21
              rows between 1 preceding and 1 preceding) as latest_date
from LoadSample

  sample_date | latest_date
-------------+-------------
 2012-03-21  | 
 2018-02-01  | 2012-03-21
 2018-02-01  | 2018-02-01
 2032-02-21  | 2018-02-01
```

**When you use same WINDOW functions, you can use WINDOW functions with name which combines multiple window func into one.**

While running WINDOW functions, SORT also runs internally.

Adding no options like PARTITION BY or ORDER BY resulted in getting total mount of records since window will be all records of that table.

# self JOIN

# NULL
Comparing null with something leads to 'unknown' in any situation.

OR: True > unknown > False

[(reference)](https://www.techscore.com/tech/sql/SQL6/06_03.html/#:~:text=%E8%87%AA%E5%B7%B1%E7%B5%90%E5%90%88%E3%81%A8%E3%81%AF%E5%90%8C%E3%81%98,%E5%90%8C%E3%81%98%E3%81%A0%E3%81%A8%E3%81%84%E3%81%86%E7%82%B9%E3%81%A7%E3%81%99%E3%80%82)

# Questions
- P10,11で言っているようなSUMを使わなければいけない理由がわからない
- Mind if I hight-light or add memo on the textbook?
- How come do I need min?? and why using MAX instead doesn't work??
  - using max also works.(checked) If the range of frame is one line, which function you choose doesn't matter.
```
select sample_date,
       min(sample_date)
           over (order by sample_date asc
              rows between 1 preceding and 1 preceding) as latest_date
from LoadSample
```


- 