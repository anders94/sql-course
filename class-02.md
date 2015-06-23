Class 2
=======
Last time we got a database going and did some SELECTs. This time we'll take a little closer
look at how the database was created and see what other kinds of information we could store.

Refresher
---------
We created a table in the database with the following SQL:

```
CREATE TABLE people (
  id             SERIAL,
  first_name     VARCHAR NOT NULL,
  last_name      VARCHAR NOT NULL,
  birthday       TIMESTAMP NOT NULL,
  favorite_color VARCHAR NOT NULL
);
```

It reads fairly well from the top if you look closely. "CREATE TABLE people" followed by a bunch
of stuff in parentheses. We're going to create a table called "people" defined by the stuff in the
parentheses. What is in the parentheses is a comma seperated list of names (that we made up) and
data types. (that are pre-defined by the database) It isn't required, but by convention we are 
typing all the pre-defined stuff in capitals and the made up stuff in lower case. This makes it 
easier to scan through some text and instantly know, "that's SQL" or "we made that up."

So the names that we made up for the columns in this table are "id", "first_name", "last_name", etc.
Why is it called "first_name" rather than "first name"? We are only allowed to use one string of
non-space characters so we can't use the space and have two words there. We connect them with an _
to satisfy this requirement because it looks almost like a space.

The other "defined by the database" stuff there says what type of data this column will have and
possibly other "constraints" on that column such as "NOT NULL", or in other words "not empty". Let's
take for example "VARCHAR" which is a shorthand for "VARIABLE CHARACTER" or a column that can accept
a variably long string of text. Another data type you could use there would be "CHAR" or "CHARACTER".
As you might guess, CHAR fields have to be an exact number of characters long. There are many 
different data types that can be used for columns. We'll take a brief look at them next.

Database defined terms, like CREATE or TIMESTAMP, can't be used as column names. You can imagine how 
the computer might not be able to figure out if you were intending one of these words as a name or
not if they weren't "reerved" by the database. These are called "reserved words". It is generally
very easy to stay away from them though so it isn't a big loss.

Data Types
----------
Columns in your table can hold data of many types. For example, you can add numbers, words (using 
VARCHAR for example) or dates. (as in TIMESTAMP) There are many other types that hold less common
things, like UUID for universally unique identifiers, INET for Inernet addresses and networks, BOOLEAN
for "yes / no" type information and even GEOGRAPHY for latitude / longitude type information. We're
going to skip the complicated stuff and focus on the common stuff.

Here's a list of common data types and example content:

* INTEGER - 52
* DECIMAL - 3.1415
* VARCHAR - "Mr. Incredible"
* TEXT - "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum."
* TIMESTAMP - "2015-06-01 06:32:14.489"
* DATE - "2015-06-01"
* TIME - "05:14:22.952"
* INTERVAL - "3:14:08.631"
* BOOLEAN - "true"

The full list for PostgreSQL is available here: http://www.postgresql.org/docs/9.4/static/datatype.html

We glossed over the SERIAL type - that is an INTEGER that automatically gets the next unused number
if one isn't specified. This is handy when you don't know what number we're at and just want to pick
the next number for your data and be sure it is unique.

Why would you want to do that? Grabbing a unique number for each row of data you throw into a database
is a great way to have something simple and unique to refer to that row of data. It is kind of like
assigning a unique number to each player on a soccer field. You don't have to remember all the names,
and there may be two players names Anne on the field. Calling them by number is more clear.

A New Table
-----------
Let's create a new table to demonstrate how these unique numbers can be used. Let's say our people have
bank accounts.

```
CREATE TABLE accounts (
  id        SERIAL,
  people_id INTEGER NOT NULL,
  balance   DECIMAL NOT NULL
);
```

There are a few things to notice in this new table. Firstly, each account we create also has a SERIAL id 
number. We might call this the "account number" if we were a bank. Just like our "people" table, this 
will start at 1 and incrament each time we INSERT data into this table.

Secondly, that people_id column refers to the id column in the people table. Got that? Yeah, this is where
it gets interesting. The people_id column says who owns this account. We're going to put the "id" from the 
people table in there to show who owns this account. Let's put some data in there.

```
INSERT INTO accounts (people_id, balance) VALUES (1, 100.00);
INSERT INTO accounts (people_id, balance) VALUES (2, 250.00);
INSERT INTO accounts (people_id, balance) VALUES (3, 100.00);
INSERT INTO accounts (people_id, balance) VALUES (3, 500.00);
INSERT INTO accounts (people_id, balance) VALUES (3, 390.00);
INSERT INTO accounts (people_id, balance) VALUES (4, 800.00);
```

Looking at this data here, we should have Elizabeth with $100 and Jane with $250. But look closely at what
person number 3 has. That's Darcy. Not only does he have $100 in one account, but two others with $500 and
$390 respectively. He's got some coin. And of course our friend Bingley has a single account with $800 in it.

Let's do some simple selects on this table. Let's find out how much money they all have together.

```
SELECT SUM(balance) FROM accounts;
```

See that? Now what about Darcy? How much does he have across all his accouts?

```
SELECT SUM(balance) FROM accounts WHERE people_id = 3;
```

Boom!

But we have to know that people_id 3 here stands for Darcy, right? Let's make it a little more clear by tying
the two tables together.

```
SELECT p.first_name, p.last_name, a.balance
FROM accounts a
  JOIN people p ON a.people_id = p.id;
```

Whoa, whoa! That gives us what we want, but what is all that JOIN stuff?
