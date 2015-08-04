Class 5
=======

Last time...

Indexing for Speed
------------------
You can see how fast queries run by looking at the bottom of the query window. (If you happen to be using the 
CLI version of the PostgreSQL client, you would have to turn timing on like this)
```
movielens=# \timing
Timing is on.
```

OK, let's see how fast it is to select a movie by exact title:
```
movielens=# SELECT * FROM movies WHERE title = 'Serpico (1973)';
 movieid |     title      |   genres    
---------+----------------+-------------
    3735 | Serpico (1973) | Crime|Drama
(1 row)

Time: 0.512 ms
```

Wow, fairly quickly, less than one milisecond!

There are around 3,000 rows in that table. Let's see what happens when we select from a table with 1 million
rows in it.

```
movielens=# SELECT count(rating) FROM ratings WHERE movieid = 3735;
 count 
-------
   495
(1 row)

Time: 71.895 ms
```

Well, still really fast but it took 71 ms for me. 100 ms would be one tenth of a second so still pretty fast
but quite a bit slower in computer time. The reason for this is the database is doing a sequential scan through 
the entire table to make sure it doesn't miss any records.

Wouldn't it be better to do something more like the phone book where all the 'A's are grouped together and all
the 'B's grouped together so if you wanted something starting with an 'A' you don't have to look through all the
'B's? We can do something like that by adding an INDEX to the movieid column in the ratings table.

```
movielens=# CREATE INDEX ratings_movieid_idx ON ratings (movieid);
CREATE INDEX
Time: 1082.549 ms
```

That took well over a second to create but now things should be a bit faster.

```
movielens=# SELECT count(rating) FROM ratings WHERE movieid = 3735;
 count 
-------
   495
(1 row)

Time: 0.686 ms
```

Yowza! That's quick! We're searching through all 1 million records in just over half a milisecond now! Compare that
to the 71 miliseconds from before and you can see we have a speedup of better than two orders of magnitude!

Now let's count the number of movies userid 1234 rated:

```
movielens=# SELECT count(movieid) FROM ratings WHERE userid = 1234;
 count 
-------
    24
(1 row)

Time: 70.262 ms
```

Whoops, slow again. What happened to our INDEX? Interesting, huh? It doesn't work in this query. Why is that?

Well, we're looking things up against userid and our INDEX is for the ratingid column. Let's ask the database
to show us how it intends to execute this SELECT by adding EXPLAIN before everything:

```
movielens=# EXPLAIN SELECT count(movieid) FROM ratings WHERE userid = 1234;
                            QUERY PLAN                             
-------------------------------------------------------------------
 Aggregate  (cost=18874.07..18874.08 rows=1 width=4)
   ->  Seq Scan on ratings  (cost=0.00..18873.61 rows=184 width=4)
         Filter: (userid = 1234)
(3 rows)

Time: 7.693 ms
```

Let's look at the query plan. It starts with "aggregate" and then "seq scan" and then "filter". You can see rough 
cost estimates on each row as well. Down at the lowest level, we have a filter on userid. To do this, we have to 
do a sequential scan across the ratings table which is going to cost us anywhere from 0.00 to 18,873.61 "units". 
That's a big range and the most costly thing in this query. The top aggregate line basically takes the worst case
scenario (because it knows it has to search all records in the table) and we are left with a consistantly high
cost of 18,874.07 to 18,874.08. Pretty high.

Let's try it.

```
movielens=# SELECT count(movieid) FROM ratings WHERE userid = 1234;
 count 
-------
    24
(1 row)

Time: 70.262 ms
```

Yep, we're right back to 70 miliseconds. This is happening because the columns in the WHERE clause don't have 
INDEXes so we have to do a sequential scan.

Let's look at the previous select where we do have an INDEX.

```
movielens=# EXPLAIN SELECT count(rating) FROM ratings WHERE movieid = 3735;
                                        QUERY PLAN                                        
------------------------------------------------------------------------------------------
 Aggregate  (cost=954.83..954.84 rows=1 width=4)
   ->  Bitmap Heap Scan on ratings  (cost=6.64..954.12 rows=286 width=4)
         Recheck Cond: (movieid = 3735)
         ->  Bitmap Index Scan on ratings_movieid_idx  (cost=0.00..6.57 rows=286 width=0)
               Index Cond: (movieid = 3735)
(5 rows)

Time: 0.295 ms
```

Whoah, whoah! A little bit of crazy town going on there, huh? What's this "Bitmap Index Scan" thing, huh?

That's our INDEX. And look at the cost - anywhere from 0.00 to 6.57. Much better, right?! We'll add in a 
heap scan of the ratings table which aggregates all those index scans and we're still slightly under 1,000
for cost. Much better than 18,874.

OK, let's create a second index on the movieid column:

```
movielens=# CREATE INDEX ratings_userid_idx ON ratings (userid);
CREATE INDEX
Time: 752.129 ms
```

Took some time but we should be good. Let's try the SELECT again.

```
movielens=# SELECT count(rating) FROM ratings WHERE movieid = 3735;
 count 
-------
   495
(1 row)

Time: 0.673 ms
```

Much better. I love it, right?

OK, cool. But there is one other thing that an INDEX can do that is helpful to us. In this case, we will
never have two ratings from the same person for the same movie. Right? Kind of makes sense, doesn't it?
You wouldn't want to skew the results by letting one guy add 80 ratings to a movie. Too much representation
from that person - we don't want that.

So we can use our INDEX to also constrain duplicates. Its like in the phone book. There should never be two
people with exactly the same name AND exactly the same address. So in this case, we can make an INDEX across
two columns in a table and make sure the pair of them remain UNIQUE.

```
movielens=# CREATE UNIQUE INDEX ratings_userid_movieid_idx ON ratings (userid, movieid);
CREATE INDEX
Time: 895.579 ms
```

And let's try it:

```
movielens=# SELECT count(rating) FROM ratings WHERE movieid = 3735 and userid = 1010;
 count 
-------
     1
(1 row)

Time: 0.380 ms
```

BOOM! Lightning quick and guarenteed never to be a duplicate.

Wait, what was that? Well, let's try it. What happens when we try to add a second rating for user 1010
for the movieid 3735 now?

```
movielens=# INSERT INTO ratings (userid, movieid, rating, timestamp) VALUES (1010, 3735, 1, 975561787);
ERROR:  duplicate key value violates unique constraint "ratings_userid_movieid_idx"
DETAIL:  Key (userid, movieid)=(1010, 3735) already exists.
Time: 8.474 ms
```

Whoops! Didn't work, did it? Coll, huh? The database is guarenteeing some data integrity here. Nobody will
be able to INSERT even a single second rating for any movie no matter what. Nice!

What We Got
-----------

So we were able to create an INDEX that will make our lookups really fast. We also got a side benefit because
we decided to make the pair of fields "movieid" and "userid" UNIQUE. Cool.

* Remember, INDEXes are used when columns with INDEXes show up in the WHERE clause. So if you are trying 
to make an INDEX on something to make a query faster, look at the WHERE clause in the SELECT you are
trying to do and make sure there are INDEXes for the columns there. Use the EXPLAIN command to see what
the query optimizer is picking.

Use of INDEXes is up to the query optimizer and it relies on some estimated statistics that the system
occasionally updates. If you create table and suddenly fill it with lots of stuff before the system has
had a chance to update its planner statistics, it may choose a sequential scan over your INDEX. In that
case, run a VACUUM ANALYZE to force an update. As this is done automatically occasionally, it isn't usually
necessary to do this but it is good to know it is there.
