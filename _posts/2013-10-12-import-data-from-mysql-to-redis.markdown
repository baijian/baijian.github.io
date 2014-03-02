---
layout: post
title: "import-data-from-mysql-to-redis"
date: 2013-10-12 01:17
comments: true
categories: 
---

Today I need to import severial millions of data to redis from mysql, If you meet
with this question, how will you do it. Below is what I do it, from a bad method to
a right method.

First I write a api, this api will first query data from redis, if not exists then query 
from mysql and save it to redis, then I will load all distinct ids from mysql, then use these
ids to curl the api, after all requests are finished, all data will be set into redis, ofcourse
I put all logic to the api. At last I found each request should query mysql and each set to redis is based
on `HTTP`, so the speed is so slow even I execute the curl in parallel.So it is a bad idea. 

<!--more-->

Then I think for a while, I search for redis documents, I found [this](http://redis.io/topics/mass-insert).
It tell us how to insert mass data into redis quickly.

Ok, first give a simple example:

```
$ echo -ne "*3\r\n\$3\r\nSET\r\n\$3\r\nkeyr\n\$3\r\nval\r\n" | redis-cli --pipe
```

What you should pay attention to is `-ne` and `\$`, and paste the command above to your linux terminal,
It will execute normal.

Then how to import mass data from mysql to redis, ok give a sql and a command, It is enough to import severial
millions of data from mysql to redis in less than severial miniutes.

```sql
SELECT CONCAT( 
    "*3\r\n",
    '$',LENGTH(redis_cmd),'\r\n',redis_cmd,'\r\n',
    '$',LENGTH(redis_key),'\r\n',redis_key,'\r\n',
    '$',LENGTH(redis_val),'\r\n',redis_val,'\r'
) FROM (
    SELECT 'SET' as redis_cmd,
    id as redis_key,
    name as redis_val
    FROM table_example 
) AS table
```

```
$ mysql -hlocalhost -uusername -ppassword  dbname \
  --skip-column-names --raw < data.sql | redis-cli --pipe
```

Ok, just a sql and a command and just cost less than severial miniutes, the problem is solved.
In summary, when meet a new problem, you should search some official documents first, It will
help us think of good methods to solve our problems.Do not do quickly without thinking for a while ^-^.

