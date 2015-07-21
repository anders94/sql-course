Class 4
=======
Last time we looked at 

A New Table
-----------

This time we're going to create a new table for the movie genres. If you look at the movies 
in the movie table:

```
SELECT * FROM movies LIMIT 10;
```

you will see the "genres" column containing some genres that each movie might fall into. But 
how many genres are there?

Let's do a little bit of string manipulation. We'll read in the genres column but we won't just
directly print what is in there. Instead, we will use a handy function that "splits" the string up 
into parts based on some character. In our case, the "pipe" character (|) is used to deliniate 
the different genres. MovieId number 1 is Toy Story. Let's read the genres for Toy Story.

```
SELECT genres FROM movies WHERE movieid = 1;
```

We should get this string:

```
Animation|Children's|Comedy
```

Not very easy to see, is it?

Let's take that string and pass it through the regexp_split_to_table function:

```
SELECT regexp_split_to_table(genres, E'\\|') FROM movies WHERE movieid = 1;
```

See that? Much nicer, huh?

OK, that's a start, but how many distinct genres are there? We can start to dump all the genres 
like this:

```
SELECT regexp_split_to_table(genres, E'\\|') FROM movies LIMIT 25;
```

but you'll notice a bunch of duplicates there. Obviously more than one movie has the same genre.

Thankfully, we can find just the DISTINCT genres - no duplicates.

```
SELECT DISTINCT regexp_split_to_table(genres, E'\\|') FROM movies;
```

You'll see we only have 18 across the entire set of movies. Isn't that interesting? Just 18 different
ones. Let's find out how many action movies we have:

```
SELECT COUNT(*) FROM movies WHERE genres like '%Action%';
```

Seems like there are 503 action movies. That means the word "Action" has been copied 503 times so it shows
up in each of those rows, right? Not very efficient, is it?

We're going to abstract away the genres so they take less space and are easier to work with. Let's start 
by creating a table to hold our 18 genres. Only this time, I'm not going to give you the SQL to generate 
this table. We're going to start here:


```
CREATE TABLE genres (
    ????????
);
```

but I'm going to leave it up to you to define the table. Remember, the only thing we need here is name of
the genre and some way to identify it by a number. I might suggest two columns then, one called genreid
and the other called genre.
