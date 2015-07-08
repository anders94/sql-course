Class 3
=======
Last time we looked at joining tables together just like zipping two different sheets in
Excel together using a common column. This makes it easy to get at the related data in one 
table when you are selecting something from the other table. The whole idea here is to 
not have multiple copies of the same data all over the place in your table. If you need to
update something, you know it lives in only one place. You don't have to scan through
everything for other copies of the same thing to update. That's the whole idea of a 
relational database management system, or RDBMS. SQL is the language we use to interact with
RDBMSes.

Movies
------

This time we're going to have a bit of fun. We'll start by creating a whole new set of tables
in a new database. We're going to throw a bunch of movies and their ratings into our new tables
and do some SELECTs on them to find out interesting things about what people think about movies.

In pgAdmin III, right click on "Databases" and select "New Database...". Set the "Name" to 
"movies" and leave everything else as-is. Click "OK" at the bottom of that window to create 
the database.

Now we have a new database called "movies" where we can create a new set of tables to store
our movie data.

With your new "movies" database selected, click the little SQL editor icon at the top of the
screen top open the SQL editor we have been using. You should get a nice empty editing window.

Now comes the tricky part. We're going to execute a SQL script that creates a few tables and 
dumps an enormous pile of data into them. All in all, this SQL is 94 megabytes of data!

Right click and select "Save as..." on this [MovieLens](https://raw.githubusercontent.com/anders94/sql-course/master/datasets/movielens.sql) 
dataset and save it somewhere on your computer. Don't just view this in your browser - it is
not a small file! Save it - it will be easier this way!

Let's go back to our blank SQL editor window. Select "File: Open..." and go find that 
"movielens.sql" file you just downloaded. It is a huge file - it will take time to load up in 
the SQL monitor.

Be careful now! Don't click the green play button - we don't want to inject everything just yet.

At the top of that file, you will see some table creation SQL. Let's look at that.

```
CREATE TABLE movies (
    movieid integer NOT NULL,
    title character varying,
    genres character varying
);
```

Not surprisingly, all of our movies will go in this table - some 3,000 of them! We'll have an 
"id" field that uniquely refers to this movie, the title of the movie, like "Goonies, The (1985)".
There is also a column for "genre" which contains a set of genres deliniated by a | character. For
example, "Adventure|Children's|Fantasy" in the case of Goonies. All in all there will be 3,883 
movies.

```
CREATE TABLE users (
    userid integer NOT NULL,
    gender character varying(1),
    age_category integer,
    occupationid integer,
    zip character varying
);
```

Here's our users. They have an "id", a gender like "M" or "F", and age_category, an occupation and 
a zipcode. We'll look more at that later. All in all we will have 6,040 distinct users.

```
CREATE TABLE ratings (
    userid integer,
    movieid integer,
    rating integer,
    "timestamp" bigint
);
```

Yowza! Here's the cool stuff. Each row in our "ratings" table will identify a user by id, a
movie by its id, assign it a numerical rating and add a timestamp for when this rating was 
assigned. All in all we have over one million ratings! Match that, Excel!

(Excel can't have more than 100,000 rows in a single table. This is more than 10x that limit!)

```
CREATE TABLE age_categories (
    age_category integer NOT NULL,
    age_range character varying
);

CREATE TABLE occupations (
    occupationid integer NOT NULL,
    occupation character varying
);
```

We'll get into these tables later but the names should be fairly self evident.

Scrolling down a little bit, you will see an insane number of INSERT commands. They will
look like this:

```
INSERT INTO movies (movieid, title, genres) VALUES (1, 'Toy Story (1995)', 'Animation|Children''s|Comedy');
INSERT INTO movies (movieid, title, genres) VALUES (2, 'Jumanji (1995)', 'Adventure|Children''s|Fantasy');
INSERT INTO movies (movieid, title, genres) VALUES (3, 'Grumpier Old Men (1995)', 'Comedy|Romance');
INSERT INTO movies (movieid, title, genres) VALUES (4, 'Waiting to Exhale (1995)', 'Comedy|Drama');
...
```

Now for the coup de grâce. Let's hit that little green play button. This will take a while.
The system is going to create all the tables above and then INSERT all the data into them.
If all goes well, you should be left without an error in the window below.

The next step is very important. So you don't accidentially re-run the same queries and mess
everything up, select all the text in the SQL window and delete it out of there! You want to 
be left with a nice blank SQL window before we move forward.

More to come...
---------------

Still working on this!
