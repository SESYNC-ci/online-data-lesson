---
---

## Response Stashing

A common strategy that web service providers take to balance their
load, is to limit the number of records a single API request can
return. The user ends up having to flip through "pages" with the API,
handling the response content at each iteration. Options for stashing
data are:

1. Store it all in memory, write to file at the end.
1. Append each response to a file, writing frequently.
1. Offload these decisions to database management software.

To repeat the exercise below at home, request an API key at
https://api.data.gov/signup/, and store it in an adjacent `api_key.py`
file with the single variable `API_KEY = your many digit key`.
{:.notes}

===

The "data.gov" API provides a case in point. Take a look at the
[request for comments](https://www.regulations.gov/docket?D=DOI-2017-0002)
posted by the US Department of Interior about Bears Ear National
Monument. The document received over two million comments, all
accessible through [Regulations.gov](https://www.regulations.gov).

===

```{python title="{{ site.handouts[0] }}"}
import requests
from api_key import API_KEY

api = 'https://api.data.gov/regulations/v3/'
path = 'document.json'
query = {
    'documentId':'DOI-2017-0002-0001',
    'api_key':API_KEY,
    }
response = requests.get(
    api + path,
    params=query)
```

===

Extract data from the returned JSON object, which gets mapped to a
Python dictionary called `doc`.

```{python title="{{ site.handouts[0] }}"}
doc = response.json()
print('{}: {}'.format(
    doc['numItemsRecieved']['label'],
    doc['numItemsRecieved']['value'],
))
```

===

Initiate a new API query for public submission (PS) comments and print
the dictionary keys in the response.

```{python title="{{ site.handouts[0] }}"}
query = {
    'dktid': doc['docketId']['value'],
    'dct': 'PS',
    'api_key': API_KEY,
    }
path = 'documents.json'
response = requests.get(
    api + path, params=query)
dkt = response.json()
```

To inspect the return, we can list the keys in the
parsed `dkt`.

```{python}
list(dkt.keys())
```

===

The purported claimed number of results is much larger than the length
of the documents array contained in this response.


```{python title="{{ site.handouts[0] }}"}
print('Number received: {}\nTotal number: {}'
    .format(
        len(dkt['documents']),
        dkt['totalNumRecords'],
))
```

===

The following commands prepare Python to connect to a
database-in-a-file, and creates empty tables in the database if they
do not already exist (i.e. it is safe to re-run after you have
populated the database).

===

### Step 1: Boilerplate

The SQLAlchemy package has a lot of features, and
requires you to be very precise about how to get started.

```{python title="{{ site.handouts[0] }}"}
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine

Base = declarative_base()
```

===

### Step 2: Table Definition

Define the tables that are going to live in the database
using Python classes. For each class, its attributes
will map to columns in a table.

```{python title="{{ site.handouts[0] }}"}
from sqlalchemy import Column, Integer, Text

class Comment(Base):
    __tablename__ = 'comment'
    
    id = Column(Integer, primary_key=True)
    comment = Column(Text)
```

===

For each document, we'll just store the "commentText" found in the API
response.

```{python}
doc = dkt['documents'].pop()
doc['commentText']
```
===

### Step 3: Connect (and Initialize)

```{python title="{{ site.handouts[0] }}"}
engine = create_engine('sqlite:///BENM.db')
Session = sessionmaker(bind=engine)

Base.metadata.create_all(engine)
```

===

You could inspect the BENM database now using any sqlite3 client: you
would find one empty "comment" table with fields "id" and "comment".

===

Add a new `rpp` parameter to request `100` documents per page.


```{python title="{{ site.handouts[0] }}"}
query['rpp'] = 10
```

===

In each request, advance the query parameter `po` to the number of the
record you want the response to begin with. Insert the documents (the
key:value pairs stored in `values`) in bulk to the database with
`engine.execute()`.


```{python title="{{ site.handouts[0] }}"}
for i in range(0, 15):
    query['po'] = i * query['rpp']
    print(query['po'])
    response = requests.get(api + path, params=query)
    page = response.json()
    docs = page['documents']
    values = [{'comment': doc['commentText']} for doc in docs]
    insert = Comment.__table__.insert().values(values)
    engine.execute(insert)
```

===

View the records in the database by reading
everyting we have so far back into a `DataFrame`.

```{python title="{{ site.handouts[0] }}"}
df = pd.read_sql_table('comment', engine)
```

===

Don't forget to disconnect from your database!

```{python title="{{ site.handouts[0] }}"}
engine.dispose()
```