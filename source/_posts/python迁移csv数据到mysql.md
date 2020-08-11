---
title: python迁移csv数据到mysql
date: 2019-02-03 12:10:47
tags:
- python
categories: 
- python
---
<!-- more -->
## 背景

csv一般以逗号分隔字段，这个从sqlite3导出的grafana数据库以 | 分隔字段。。做法是逐条读取csv的数据，拼接成sql语句，通过MySQLdb库执行数据库操作。

在执行的过程中发现有某些csv数据被意外截断的情况，是因为某些字段存储的是json字符串，中间的逗号和分号造成字符串意外分成两个或者多个，所以还需要循环拼接一下才能还原出完整的sql语句。

在插入的时候发现主键总是出现状况，首先是变量类型，其次是值会重复，于是直接加了个i变量充当主键。。虽说暴力了点但是it's works。

目前还有的问题是在导入某个字段有超级长的json，里面还包含转义字符串的时候，会错误，本来想通过在拼接的时候用三对引号包裹每个字段的方式解决，但是没成功。。


```python
# -*- coding: UTF-8 -*-
import csv
import sys
import MySQLdb

csv_file = sys.argv[1]
tables = sys.argv[2]

conn = MySQLdb.connect(host='mysql.rds.aliyuncs.com', user='grafana', passwd='121321', db='grafana_config')
cursor = conn.cursor()

with open(csv_file) as f:
    f_csv = csv.reader(f)
    create_table_sql = 'create table if not exists ' + tables
    seq = ','
    i = 1
    for row in f_csv:
        i = i + 1
        rows = ''
        for piece in row:
            rows += piece + ','

        total_str = rows[0:-1].split('|')
        insert_sql = "insert into " + tables + " values("
        values = []
        for item in total_str:
            values.append("'" + str(item) + "'")
        del values[0]
        value_str = seq.join(values)
        insert_sql += str(i) + ',' + value_str +  ')'
        print(insert_sql)
        cursor.execute(insert_sql)
        conn.commit()

cursor.close()
conn.close()
f.close()

```