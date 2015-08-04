Class 5
=======

Last time...

Indexing for Speed
------------------
Let's see how fast we can select a movie by exact title.

```
movielens=# \timing
Timing is on.

movielens=# select * from movies where title = 'Serpico (1973)';
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
movielens=# select count(rating) from ratings where movieid = 3735;
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
movielens=# create index ratings_movieid_idx on ratings (movieid);
CREATE INDEX
Time: 1082.549 ms
```

That took well over a second to create but now things should be a bit faster.

```
movielens=# select count(rating) from ratings where movieid = 3735;
 count 
-------
   495
(1 row)

Time: 0.686 ms
```

Yowza! That's quick! We're searching through all 1 million records in just over half a milisecond now! Compare that
to the 71 miliseconds from before and you can see we have a speedup of better than two orders of magnitude!
