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



~~~python
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
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

Extract data from the returned JSON object, which gets mapped to a
Python dictionary called `doc`.



~~~python
doc = response.json()
print('{}: {}'.format(
    doc['numItemsRecieved']['label'],
    doc['numItemsRecieved']['value'],
))
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
Number of Comments Received: 2839046
~~~
{:.output}


===

Initiate a new API query for public submission (PS) comments and print
the dictionary keys in the response.



~~~python
query = {
    'dktid': doc['docketId']['value'],
    'dct': 'PS',
    'api_key': API_KEY,
    }
path = 'documents.json'
response = requests.get(
    api + path, params=query)
dkt = response.json()
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


To inspect the return, we can list the keys in the
parsed `dkt`.



~~~python
> list(dkt.keys())
~~~
{:title="Console" .input}


~~~
['totalNumRecords', 'documents']
~~~
{:.output}


===

The purported claimed number of results is much larger than the length
of the documents array contained in this response.




~~~python
print('Number received: {}\nTotal number: {}'
    .format(
        len(dkt['documents']),
        dkt['totalNumRecords'],
))
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
Number received: 25
Total number: 783340
~~~
{:.output}


===

The following commands prepare Python to connect to a
database-in-a-file, and creates empty tables in the database if they
do not already exist (i.e. it is safe to re-run after you have
populated the database).

===

### Step 1: Boilerplate

The SQLAlchemy package has a lot of features, and
requires you to be very precise about how to get started.



~~~python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
~~~
{:title="{{ site.data.lesson.handouts[1] }}" .no-eval .text-document}


===

### Step 2: Table Definition

Define the tables that are going to live in the database
using Python classes. For each class, its attributes
will map to columns in a table.



~~~python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, Text

Base = declarative_base()

class Comment(Base):
    __tablename__ = 'comment'
    
    id = Column(Integer, primary_key=True)
    comment = Column(Text)
    
engine = create_engine('sqlite:///BENM.db')
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)
~~~
{:title="{{ site.data.lesson.handouts[1] }}" .no-eval .text-document}


===

For each document, we'll just store the "commentText" found in the API
response.



~~~python
> doc = dkt['documents'].pop()
+ doc['commentText']
~~~
{:title="Console" .input}


~~~
"I am appalled that our treasured national parks and monuments, like the Bears Ears National Monument, are up for review at all. Bears Ears is one of our nation's newest monuments -- the American people are very lucky to now call this ancient site covering an expanse of 1.3 million acres a public resource protected for future generations. The monument protects ancient sites that are sacred to the Native American tribes in southern Utah's red-rock country. Utah is greatly enriched by the Bears Ears National Monument. Bears Ears National Monument also provides incredible spaces for outdoor activities-- it is one of best places in the world for rock climbing and bouldering. These public lands need to stay in public hands. No president has EVER attempted to abolish a national monument, and an attack on one park is an attack on all our parks. Secretary Zinke, I am adamantly opposed to any effort to eliminate or diminish protections for Bears Ears or any other national monument, and I urge you to support our public lands and waters and recommend that our current national monuments remain protected."
~~~
{:.output}

===

### Step 3: Connect (and Initialize)



~~~python
from schema import Session, Comment
session = Session()
engine = session.bind
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

You could inspect the BENM database now using any sqlite3 client: you
would find one empty "comment" table with fields "id" and "comment".

===

Add a new `rpp` parameter to request `100` documents per page.




~~~python
query['rpp'] = 10
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

In each request, advance the query parameter `po` to the number of the
record you want the response to begin with. Insert the documents (the
key:value pairs stored in `values`) in bulk to the database with
`engine.execute()`.



~~~python
table = Comment.metadata.tables['comment']
for i in range(0, 15):
    query['po'] = i * query['rpp']
    print(query['po'])
    response = requests.get(api + path, params=query)
    page = response.json()
    docs = page['documents']
    values = [{'comment': doc['commentText']} for doc in docs]
    insert = table.insert().values(values)
    engine.execute(insert)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
0
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbc56518d0>
10
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbca08fd68>
20
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbc55da080>
30
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbca09c4e0>
40
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbc55e12e8>
50
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbca09ce48>
60
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbc5651080>
70
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbca09cc50>
80
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbc90f9f28>
90
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbd1dc9a90>
100
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbca094d68>
110
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbca09c7f0>
120
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbca0a0668>
130
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbca0a4d30>
140
<sqlalchemy.engine.result.ResultProxy object at 0x7fcbca0a4c88>
~~~
{:.output}


===

View the records in the database by reading
everyting we have so far back into a `DataFrame`.



~~~python
df = pd.read_sql_table('comment', engine)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

Don't forget to disconnect from your database!



~~~python
engine.dispose()
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}

