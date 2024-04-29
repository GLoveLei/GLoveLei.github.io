---
title: '一个sql查询排序小技'
date: '2019/04/21 14:59:21'
categories:
   - mysql
tags:
   - mysql
toc_level: 3
version: 3
---


```
SELECT CASE WHEN (num = 0) THEN '0' WHEN (age > 0 AND age <= 10) THEN '10'
WHEN (age > 100 AND age <=200) THEN '100' ELSE '>200' END as num
```

这样的情况，如果用别名排序的话，数据库字符串排序会变成乱序，导致难以浏览，所以可以在别名上加前缀  
，这样就可以通过首字母排序了，比如:

```
SELECT CASE WHEN (num = 0) THEN 'a. 0' WHEN (age > 0 AND age <= 10) THEN 'b. 10'
WHEN (age > 100 AND age <=200) THEN 'c. 100' ELSE 'd. >200' END as num
```

这样select出来的数据就会按照你设想的方式排序了

  

本文转自 [https://gaoleiv3e.cn/2019/04/21/Ssql-one/](https://gaoleiv3e.cn/2019/04/21/Ssql-one/)，如有侵权，请联系删除。