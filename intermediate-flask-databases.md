Title: Intermediate Flask
Blurb: So you can spin up a quick Flask app and run it in dev mode. What do now?

These are some things to do to get you moving into the intermediate Flask realm.

## Binds (SQLAlchamy)
What if you are using SQLAlchemy and the database is in a bit of a mess? It might be spread across a few different schema, or connections, or even different databases entirely. You want the `binds` option. With this you can transparently connect to different databases in whatever configuration and query them as if they were a single interface.

### Get configured 
You are probably used to seeing this line: `SQLALCHEMY_DATABASE_URI`. This is the connection URI that you run all your queries against. It represents a single connection to a single database and serves as the default connection used for all queries to all models.

`SQLALCHEMY_BINDS` is a dictionary that holds a record of connection URIs and a label for them to be referenced by.

<pre>
<code>
... more config above...
SQLALCHEMY_DATABASE_URI = "my_connection_uri"
...more config below...
</code>
</pre>

Instead or in addition to this we can define the dictionary `SQLALCHEMY_BINDS`

<pre>
<code>
... more config above...
SQLALCHEMY_BINDS = {
	'alpha': 'my_connection_uri_for_alpha',
	'alpha_readonly': 'my_connection_uri_for_alpha_readonly',
	'prime': 'my_connection_uri_for_prime',
	... more bind uri's if you have them ...
}
...more config below...
</code>
</pre>

### OK, so how do I use it?
When you define a model you have a few meta options you can set. One of them is the `__bind_key__` option. By default this is `None` and if it is `None` then the `SQLALCHEMY_DATABASE_URI` URI is used. This is the default way that models are used:

<pre>
<code>
from app import db # This is Flask-SqlAlchemy db

... more models above...
class Person(db.Model):
	__bind_key__ = None # This is the default case, uses `SQLALCHEMY_DATABASE_URI`

	... columns defined below ...

...more models below...
</code>
</pre>
 
If you set `__bind_key__` though, any query to that table will transparently use that connection:

<pre>
<code>
from app import db # This is Flask-SqlAlchemy db

... more models above...

class Person(db.Model):
	__bind_key__ = 'prime'
	... columns defined below ...

...more models below...
</code>
</pre>

As useful as this is already, there are a few more things you can do to help yourself.


<pre>
<code>
from app import db # This is Flask-SqlAlchemy db

class Person(db.Model):
	__bind_key__ = 'alpha'
	... columns and functions defined below ...


class PersonReadOnly(Person):
	__bind_key__ = 'alpha_readonly'
	# NO models or functions, they are all inherited
</code>
</pre>

The `PersonReadOnly` class has all the same columns and function of the `Person` class but processed through the different URI. You can use this to protect yourself by having different URIs connect to the same database but with different connection permissions, avoiding situations where someone has access to the database in a way that you do not wish them to.

<pre>
<code>
# Potentially issue where connection has more permissions than it needs
unsafer = db.session.execute(db.select(Person)).scalars()

# Only select permission, inserts fail permissions
safer = db.session.execute(db.select(PersonReadOnly)).scalars() 
</code>
</pre>

Is it more work to setup and maintain? Yes. Is it really useful. You betcha.

### Cross `__bind__` joins
Lets setup a `one-to-many` relationship with the `Person` class and a `Book` class. This is assuming that you are operating on one DB (PostgreSQL in this case.)

<pre>
<code>
class Person(db.Model):
    __bind_key__ = "alpha"
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)
    books = db.relationship('Book', backref='person')
    
    def __repr__(self):
        return f"{self.name}"


class Book(db.Model):
    __bind_key__ = "prime"
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)
    person_id = db.Column(db.Integer, db.ForeignKey(Person.id))

    def __repr__(self):
        return f"{self.name}"
</code>
</pre>

Notice that the `Person` class is using the `alpha` bind and the `Book` class is using the `prime` bind.

<pre>
<code>
INSERT INTO person (name) VALUES
	 ('Tim'),
	 ('Tom'),
	 ('Mazie'),
	 ('Nadine'),
	 ('Augustine');

INSERT INTO book (name, person_id) VALUES
	 ('Once upon a time',2),
	 ('Twice too many',1),
	 ('Three times the charm',3),
	 ('Go forth and conquor',3),
	 ('Five little soldiers',4),
	 ('Six and out',4),
	 ('Seventh heaven',5),
	 ('Doneights',5),
	 ('Sweet nimes',2),
	 ('Tenth ducket',3),
	 ('Eleventh duck',2),
	 ('Twelve Monkeys',1);
</code>
</pre>

If we do a join we can request the `Person`s books (pre-seeded the DB earlier).

<pre>
<code>
>>> person = db.session.execute(db.select(Person)).scalar()
>>> person
Tim
>>> person.books
[Twelve Monkeys, Twice too many]
</code>
</pre>

So far so good. Lets get a bit silly:

<pre>
<code>
... more config above...
SQLALCHEMY_BINDS = {
	'alpha': 'PostgreSQL_URI',
	'prime': 'MySQL_URI',
}
...more config below...
</code>
</pre>

Cheeky!

Not so fast though. The referential integrity will catch up to you here as the `Book` class is is defining a reference to the `Person` class which is a physically different database. If we drop that requirement and try again...

<pre>
<code>
# Same query as before
>>> person = db.session.execute(db.select(Person)).scalar()
>>> person
Tim
>>> person.books
[Twelve Monkeys, Twice too many]
</code>
</pre>

Hooray! Wait...how did that work? They are physically different?!?

The answer comes down to `lazy`.

An expanded way to define the `Person` -> `books` relationship is to do it like this `books = db.relationship('Book', lazy='select', backref='person')`. The `lazy` parameter describes how the database tries to join the two relations.

#### 'select' / True

This is the default (as seen above).  When the query is executed, a second query is run to get the data.

<pre>
<code>
>>> from flask_sqlalchemy import record_queries as rq
>>> rq.get_recorded_queries()
[
	_QueryInfo(
		statement='SELECT person.id, \nFROM person',
		=[{}],
		start_time=238562.375763577,
		end_time=238562.376351878, location='<unknown>'
	),
	_QueryInfo(
		statement='SELECT book.id AS book_id, book.name AS book_name, book.person_id AS book_person_id \nFROM book \nWHERE %s = book.person_id',
		parameters=[(1,)],
		start_time=238562.49037108,
		end_time=238562.497730226,
		location='<unknown>'
	)
]
</code>
</pre>

This allows the query to use the `Book` class and it's `bind` to make things work as expected. There is still no enforced (at the DB level) referential integrity but it 'works'.

#### 'joined' / False

Load the relationship using a `join` query:

<pre>
<code>
sqlalchemy.exc.ProgrammingError: (psycopg2.errors.UndefinedTable) relation "book" does not exist
LINE 2: FROM person LEFT OUTER JOIN book AS book_1 ON person.id = bo...
                                    ^

[SQL: SELECT person.id, person.name, book_1.id AS id_1, book_1.name AS name_1, book_1.person_id
FROM person LEFT OUTER JOIN book AS book_1 ON person.id = book_1.person_id]
</code>
</pre>

The join here does not respect the `bind` and attempt to use the MySQL URI and instead uses the `Person` bind and subsequently can not find the table. It is a nonstarter.

#### 'subquery'

Same as above, but uses a sub-query.

<pre>
<code>
sqlalchemy.exc.ProgrammingError: (MySQLdb.ProgrammingError) (1146, "Table primedb.person' doesn't exist")

[SQL: SELECT book.id AS book_id, book.name AS book_name, book.person_id AS book_person_id, anon_1.person_id AS anon_1_person_id
FROM (SELECT person.id AS person_id
FROM person) AS anon_1 INNER JOIN book ON anon_1.person_id = book.person_id]
</code>
</pre>

This is basically the error as above; the two dbs cant cooperate.

### PostgreSQL specifics

What usually gets produced is to use PostgreSQL `schema`s to separate disparate sections of an overall otherwise related system (in MySQL land, a different database is a 'schema'.) There are a few things that have come up that might be an issue.

#### Search path

When a schema is used you need to be explicit about when and how it is used. The easiest and most straightforward is to set the schema name:

<pre>
<code>
class Person(db.Model):
    __bind_key__ = "alpha"
    __table_args__ = {"schema":"schema_name"}

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)
    books = db.relationship('Book', backref='person')
    
    def __repr__(self):
        return f"{self.name}"
 
</code>
</pre>

The `__table_args__` and `__bind_key__` are independent of each other and can be combined or excluded as needed. Remember that the `__bind_key__` only records how to connect to a DB, not to do once there.

This ties in to the other way to sort this out with the search path: `SHOW search_path;`

If you have two schema, with tables of the same name present in them, which should be the one to use in a query? PostgreSQLs answer is to explicitly define it. By default though the search space is rather limited: 

<pre>
<code>
SHOW search_path;

 search_path
--------------
 "$user", public
</code>
</pre>

Here, `public` is the default, available on creation schema that you all know and love. `$user` is a schema that (might not exist yet) matching the role's name.

PostgreSLQ will attempt to look for tables in the schemas 

This path is updatable in pretty much any combination you want: the database generally, a role generally, a combination of the two or you can even edit the default. It is advisable to add in new paths for two reasons: first is to not muck with the default behaviour that other users and databases are expecting and two so that you can recover from mistakes easily later on. The way that modification per users and databases is to add in the specific at the top of the list and match you way down to the default at the bottom. Much better control, maintainability and fewer surprises for other developers.

It is still a bit of a surprise though that will cause problems unless you are really explicit so I recommend the use of `__table_args__ = {"schema":"schema_name"}` every time.

You can see all the current settings by running:

<pre>
<code>
SELECT r.rolname, d.datname, rs.setconfig
FROM   pg_db_role_setting rs
LEFT   JOIN pg_roles      r ON r.oid = rs.setrole
LEFT   JOIN pg_database   d ON d.oid = rs.setdatabase;
</code>
</pre>

You can set it for a database:
<pre>
<code>
ALTER DATABASE \<database\_to\_alter\> set search_path = prime,alpha;
</code>
</pre>

You can set it for a role:
<pre>
<code>
ALTER ROLE \<role\_to\_alter\> set search_path = prime,alpha;
</code>
</pre>

You can set it for a role on a database:
<pre>
<code>
ALTER ROLE \<role\_to\_alter\> IN DATABASE \<database\_to\_alter\> set search_path = prime,alpha;
</code>
</pre>

Undo it via:
<pre>
<code>
ALTER ROLE \<role\_to\_alter\> RESET search\_path;
</code>
</pre>

### MySQL and SQLITE specifics

Dunno, do not use them enough except to say that MySQL considers a different database to be a different schema.

## Have fun

Hope this helped :)