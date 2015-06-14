SQL Course
==========
These notes cover a beginning SQL course using PostgreSQL.

Setup
-----
The class assumes you are using Mac OS X. We will run our own local PostgreSQL server so we can 
build and break things without worrying about doing any real damage.

Start by getting the Mac OS X PostgreSQL package from:

http://www.enterprisedb.com/products-services-training/pgdownload

Once downloaded, click on the resulting file to mount it and double click to start the installer.
You may be asked to enter your password so the application has the privileges it needs to install
properly. Accept the defaults as you install. You will also be asked to supply a password for the
database superuser. Use something simple that you won't forget.

When everything completes, you will see an option to run Stack Builder. You don't need to do that
at this point. Stack Builder allows you to install additional components that you won't need just
yet.

Startup
-------
Under "Applications", you should now have a PostgreSQL folder. In there you will find pgAdmin III.
Run that bad boy. This is the frontend we are going to use to interact with our database. Remember,
the database engine is installed and running the the background on your computer at all times. It
doesn't have a window or menu item, its just a "deamon" back there in the background silently doing
your bidding. Your Mac is a unix system. In the unix world, "daemon" is the fancy name for programs 
that run in the background.

New Database
------------
The window that comes up should already have a link to your PostgreSQL install under "Servers". 
Double click that and supply your password. That should open up to show what databases you have
installed on your system. By default, you should have a database called "postgres". Let's NOT mess
with that one as it is there for administrative reasons.

Instead, let's control-click on "Databases (1)" and give our new database the name "sql-class". Don't
worry about filling anything else out. Just click "OK" and you'll have your new database.

Test Your Database
------------------
The last thing we need to do is a quick test to make sure things are working. We're going to do a bunch 
of things without much explanation, but hang in there. We'll go back over it in more detail in a bit.

Start by highlighting your new "sql-class" database and clicking the "SQL" magnafying glass up at the 
top of the window between the garbage pail and the thing that looks like an empty Excell sheet. You 
should get a new window that pops up with a "SQL Editor" tab open. This is where we are going to do
most of our work. While you can do many things through the graphical interface, we'll lean on SQL in 
this window to do pretty much everything.

First up, we're going to create a table in our database called "people" and put some data in there. Enter
the following into the "SQL Editor" window:

```
CREATE TABLE people (
  id             SERIAL,
  first_name     VARCHAR NOT NULL,
  last_name      VARCHAR NOT NULL,
  birthday       TIMESTAMP NOT NULL,
  favorite_color VARCHAR NOT NULL
);
```

This is going to create a table called "people" that we can use to store some information. Up at the top 
of the window right next to the little magnafying glass, you will see three little green arrows. Click the 
first of those arrows. This will execute what we have entered into the "SQL Editor" window and show us the
results in the "Output pane" below.

If all goes well, you should see "Query returned successfully with no result in (some number of) ms." The 
number shows how many "milliseconds" your command took to execute. Creating this simple table is generally
very quick.

Next we're going to put some sample data in this table so we have something to play with. Enter the following 
lines:

```
INSERT INTO people (first_name, last_name, birthday, favorite_color) VALUES ('Elizabeth', 'Bennet', '1992-02-13', 'yellow');
INSERT INTO people (first_name, last_name, birthday, favorite_color) VALUES ('Jane', 'Bennet', '1990-05-16', 'yellow');
INSERT INTO people (first_name, last_name, birthday, favorite_color) VALUES ('Fitzwilliam', 'Darcy', '1987-02-01', 'blue');
INSERT INTO people (first_name, last_name, birthday, favorite_color) VALUES ('Charles', 'Bingley', '1987-05-16', 'red');
```

Press the green arrow to run that and if all goes well, you should get something like "Query returned successfully:
one row affected, (some number of) ms execution time."

Now we have a table with some data in it. The last step is to read our people out of the database. We'll do that 
by entering this:

```
SELECT * FROM people;
```

Down in the "Data Output" pane, you should see all your people listed out. If you go this far, "Woohoo!!" you have
a PostgreSQL database up and running and you have created a database with a table and some data and you have 
SELECTed stuff out of it! Well done.

Extra Fun
---------
Before we finish though, let's have the database do something with our data. Try these statements one by one and
watch what we get as results:

```
SELECT * FROM people ORDER BY birthday ASC;
```

See that? They are listed in ASCending ORDER BY their birthdays. Try this:

```
SELECT first_name, last_name, birthday FROM people ORDER BY birthday DESC;
```

We can pick only the columns we want to show. But notice we also inverted the list when we ORDERed BY birthday 
in DESCending order.

Now try this:

```
SELECT first_name, last_name, birthday, AGE(birthday) FROM people;
```

Cool, huh? Now we are getting a little bit ahead of ourselves. We'll end here and next time we'll go into this
in a bit more detail.
