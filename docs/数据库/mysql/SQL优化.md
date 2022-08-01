- 确认只查询一条数据使用 `limit 1`

避免全表扫描

- 避免 `select *`
- 字段指定为 `not null`
- 把 IP 地址存成 UNSIGNED INT

```mysql
mysql> inet_aton('255.255.255.255'); # ip 转 数字
mysql> inet_ntoa(1231231123123); # 数字转 ip
```

- 隐式转换导致索引失效

field 定义类型为 VARCHAR(11);

错误：select * table where field=1;

正确：select * table where field='1';

- `where` 中对字段进行运算，如 `where field + 1 = 2`
- `where` 中字段作为函数参数，如 `where substr(field, 1, 1) = 2`
- `<>` 、`not in`、`not exist`、`!=`、`like '%_'` 导致索引失效
- 空值使用 `is null` 、`is not null`、`isnull()`
- limit 大数字导致速度慢，如 `limit 1000000,10`

