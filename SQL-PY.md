# Working with SQL and Python

Like all other programming languages, Python supports communicating with databases. What follows explains the elements needed for Python to interact with a relational MySQL database.

## 0. Setting Up

You need three pieces to set up a database connection in Python:

1. Connection information
2. Libraries / packages
3. Connection string

### Connection Information

To connect to a remote MySQL database, you will need to specify a HOST, a database USER, a database PASSWORD, and the name of a specific DATABASE.

These can be populated as variables within your code. However, you should hold back at least the PASSWORD to be passed in as an environment variable, and **not** commit it directly to Git/GitHub.

```
DBHOST = "ds2022.cqee4iwdcaph.us-east-1.rds.amazonaws.com"
DBUSER = "admin"
DBPASS = os.getenv('DBPASS')
DB = "xxxxx"
```

### Libraries / packages

For this connection to work you need at least the `os` and `mysql.connector` packages:
```
import os
import json
import mysql.connector
```

### Connection String

Finally, bring these two elements together by creating a complete DB connection string, along with a cursor.

- The connection string opens the actual connection to the database.
- The cursor allows you to iterate through entries, or interact in other ways with the results of a connection.

```
db = mysql.connector.connect(user=DBUSER, host=DBHOST, password=DBPASS, database=DB)
cur=db.cursor()
```

## 1. SELECT Statements

Using all the pieces above, we can then build outo a simple example of a `SELECT` query.
```
query = "SELECT * FROM genres ORDER BY genreid;"
try:    
    cur.execute(query)
    headers=[x[0] for x in cur.description]
    results = cur.fetchall()
    json_data=[]
    for result in results:
        json_data.append(dict(zip(headers,result)))
    output = json.dumps(json_data)
    print(output)
except mysql.connector.Error as e:
    print("MySQL Error: ", str(e))
```

A few notes on the block above:

- It makes use of a static SQL query, and does not require a parameter as part of the SELECT statement.
- The `try` stanza attempts to execute the SQL query by using the cursor object.
- Since this particular block needs to reassemble the response in JSON, it fetches the `headers` so that the entire response can be formatted as a `dict` dictionary. This is not always desired.
- The results of the `.fetchall()` are passed into a new object, which can then be looped through one by one.
- The rest of the block is simply formatting and outputting the results into JSON.
- Normal error handling is used in case the connection or query has an error.


A simpler version of this query without any JSON formatting or dictionaries might look like this:

```
query = "SELECT * FROM genres ORDER BY genreid;"
try:    
    cur.execute(query)
    results = cur.fetchall()
    data=[]
    for result in results:
        print(result)
except mysql.connector.Error as e:
    print("MySQL Error: ", str(e))
```

Results in this output:
```
(1, 'Rock')
(2, 'Indie')
(3, 'Pop')
(4, 'Hiphop')
(5, 'Jazz')
(6, 'Country')
(7, 'Classical')
(8, 'OTHER')
```

## 2. INSERT Statements

Another common example performs an INSERT statement to inject data into a new database record. Here is a simplified version of that. Notice the similarities in setup with the SELECT statement above:

```
import os
import json
import mysql.connector

# database things
DBHOST = os.getenv('DBHOST')
DBUSER = os.getenv('DBUSER')
DBPASS = os.getenv('DBPASS')
DB = os.getenv('DB')
db = mysql.connector.connect(user=DBUSER, host=DBHOST, password=DBPASS, database=DB)
cur = db.cursor()

# Assuming you have some values from someplace:
TITLE = "Running to Stand Still"
ALBUM = "The Joshua Tree"
ARTIST = "U2"
YEAR = 1986
GENRE = 1

# try to insert the song into the database
try:
    add_song = ("INSERT INTO songs "
            "(title, album, artist, year, genre) "
            "VALUES (%s, %s, %s, %s, %s)")
    song_vals = (TITLE, ALBUM, ARTIST, YEAR, GENRE)
    cur.execute(add_song, song_vals)
    db.commit()

except mysql.connector.Error as err:
    db.rollback()
```

A few notes on the block above:

- Everything up to the `try` statement is fairly straightforward setup and populating some values to insert.
- Within `try`, the SQL statement is assembled as a typical `INSERT` statement, with string replacements (`%s`) for each value to be passed in.
- The `song_vals` array then calls five (5) values, in the right order to match the INSERT statement's order.
- Next, the `cur.execute()` method brings those two together to push specific values into the INSERT statement.
- Finally, the `db.commit()` call triggers the database transaction.
- In cases of error, the transaction is rolled back (reverted) so as to not leave any residue of a failed insertion.
