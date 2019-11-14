# MySQL操作

## 基本操作

### 插入
```sql
insert into world 
(name,continent,area,population,gdp)
values('Angola',' Africa',1246700,20609294,100990000 );
```

### 更新
![](https://github.com/Dannymeng/picture/blob/master/1571305719\(1\).png?raw=true)
将world表中满足population>37000000的条件的area变成555，否则变成666
```sql
update world 
   set area = case 
   when population>37000000 then 555 
   else 666 end;
```

或者

```sql
update world set area = if(population>37000000,555,666);
```



