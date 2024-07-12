+++
title = 'SQLite UDF in Python'
date = 2016-02-22T00:00:00-04:00
categories = 'Tutorial'
tags = ['Python', 'Pandas', 'SQLite']
+++

From [my StackOverflow answer](https://stackoverflow.com/a/35544470/83144) on the
subject of parsing SQLite string dates:

Python makes it pretty easy to add a user-defined function to a SQLite query. Let's demonstrate this by populating a table full of text reprensentations of dates, and then use a simple Python UDF we write here called `date_parse` to attempt to parse the text date into a real date.

First, we need to do some basic setup. Let's do some standard Python imports firsts. We'll use Python's build-in SQLite support, as well as it's built-in date parser. We also have ensured `pip install pandas` was run so that we can use Pandas to display our demo output nicely in Jupyter Lab.

```python
#!/usr/bin/env python3
import sqlite3
from dateutil import parser
import pandas as pd
```

Now, let's make a `populate_db` function that will take a connection to a SQLite database with a table called `txtdates` containing various text reprensentations of dates.

```python
def populate_db(con):
    ''' Setup some values to parse '''
    cur = con.cursor()

    # Make a table
    sql = '''
    CREATE TABLE txtdates (
        "id" integer primary key,
        "date" text
    );
    '''
    cur.execute(sql)

    # Fill the table
    dates = [
        '2/1/03',
        '03/2/04',
        '4/03/05',
        '05/04/06',
        '6/5/2007',
        '07/6/2008',
        '8/07/2009',
        '09/08/2010',
        '2-1-03',
        '03-2-04',
        '4-03-05',
        '05-04-06',
        '6-5-2007',
        '07-6-2008',
        '8-07-2009',
        '09-08-2010',
        '31/12/20',
        '31-12-2020',
        'BOMB!',
        None
    ]
    params = [(x,) for x in dates]
    cur.executemany(''' INSERT INTO txtdates ("date") VALUES (?); ''', params)

```

Now, we're ready to make our User-Defined Function (UDF) called `date_parse`.

```python
def date_parse(s):
    ''' SQL UDF to convert a string to a date '''
    try:
        t = parser.parse(s, parser.parserinfo(dayfirst=True))
        return t.strftime('%Y-%m-%d')
    except:
        return None
```

And finally, let's put it all together. Create an in-memory SQLite database, populate
the database with our test date data, add our Python UDF, and query the parsed results.

```python
# Create an in-memory SQLite database
with sqlite3.connect(":memory:") as con:
    # Initialize the database
    populate_db(con)

    # Tell SQLite about our Python UDF
    con.create_function("date_parse", 1, date_parse)

    # Construct a SQL SELECT
    query = '''
      SELECT "id"
           , "date"
           , date_parse("date") as parsed
        FROM txtdates
    ORDER BY 3
    ;
    '''

    # Pull and display the data
    df = pd.read_sql(query, con=con)

# Show results
df
```

| id  | date       | parsed     |
| --- | ---------- | ---------- |
| 1   | 2/1/03     | 2003-01-02 |
| 2   | 03/2/04    | 2004-02-03 |
| 3   | 4/03/05    | 2005-03-04 |
| 4   | 05/04/06   | 2006-04-05 |
| 5   | 6/5/2007   | 2007-05-06 |
| 6   | 07/6/2008  | 2008-06-07 |
| 7   | 8/07/2009  | 2009-07-08 |
| 8   | 09/08/2010 | 2010-08-09 |
| 9   | 2-1-03     | 2003-01-02 |
| 10  | 03-2-04    | 2004-02-03 |
| 11  | 4-03-05    | 2005-03-04 |
| 12  | 05-04-06   | 2006-04-05 |
| 13  | 6-5-2007   | 2007-05-06 |
| 14  | 07-6-2008  | 2008-06-07 |
| 15  | 8-07-2009  | 2009-07-08 |
| 16  | 09-08-2010 | 2010-08-09 |
| 17  | 31/12/20   | 2020-12-31 |
| 18  | 31-12-2020 | 2020-12-31 |
| 19  | BOMB!      | None       |
| 20  | None       | None       |

Now we can see how all the parsable date forms came out in YYYY-MM-DD form. This isn't something that SQLite has an easy built-in way to do, so using a Python UDF saves the day!
