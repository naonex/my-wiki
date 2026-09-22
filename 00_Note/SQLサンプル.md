# deleteSQLサンプル

#### 複数カラムを比較＆集計から対象を割り出す

```sql
delete from TABLE1
where (CD, TIMESTAMP) IN
    (
        select CD, TIMESTAMP
        from TABLE1
        group by CD, TIMESTAMP
        having sum(AMOUNT) < 10000
    )
;
```
