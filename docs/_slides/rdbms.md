---
---

## Paging & Stashing

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
doc = (
    requests
    .get(api + path, params=query)
    .json()
)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

Extract data from the returned JSON object, which gets mapped to a
Python dictionary called `doc`.



~~~python
> doc['numItemsRecieved']
~~~
{:title="Console" .input}


~~~
{'label': 'Number of Comments Received', 'value': '2839046'}
~~~
{:.output}


===

Initiate a new API query for public submission (PS) comments and print
the dictionary keys in the response.



~~~python
query = {
    'dktid':doc['docketId']['value'],
    'dct':'PS',
    'api_key':API_KEY,
}
path = 'documents.json'
dkt = (
     requests
    .get(api + path, params=query)
    .json()
)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


To inspect the return, we can list the keys in the
parsed `dkt`.



~~~python
> list(dkt.keys())
~~~
{:title="Console" .input}


~~~
['documents', 'totalNumRecords']
~~~
{:.output}


===

The purported claimed number of results is much larger than the length
of the documents array contained in this response.



~~~python
> len(dkt['documents'])
~~~
{:title="Console" .input}


~~~
25
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
"Dear Ryan Zinke,\n\nOur national monuments and public lands and waters help define who we are as a nation by telling the story of our shared historical, cultural, and natural heritage. I am concerned that the recent Executive Order attempts to undermine our national monuments and to roll back protections of these public lands. Protected public lands are an important part of what makes America great. I strongly urge you to oppose any efforts to eliminate or shrink our national monuments.\n\nSince President Theodore Roosevelt signed the Antiquities Act into law in 1906, 16 Presidents - 8 Republicans and 8 Democrats - have used the authority granted by the act to safeguard public lands, oceans, and historic sites in order to share America's story with future generations. These national monument designations are broadly supported from coast to coast and provide a myriad of benefits to local communities, including economic boosts from tourism, places to enjoy the outdoors, clean air and water, protection for ecologically sensitive areas, and windows into our country's history.\n\nSending a signal that protections for our shared history, culture, and natural treasures are temporary would set a terrible precedent. National monuments have been shown to be tremendous drivers of the $887 billion outdoor recreation economy and businesses rely on the permanency of these protections when making decisions about investing in these communities.\n\nFrom Maine's magnificent Katahdin Woods to the colorful canyons of Utah's Grand Staircase-Escalante to the western history held in New Mexico's Organ Mountains-Desert Peaks, these landmarks, landscapes, and seascapes have value which far exceeds their physical features; they manifest the core democratic ideals of freedom, justice, and equality. They are our legacy to our children and our children's children, and a gift that belongs to all Americans.\n\nI am firmly opposed to any effort to revoke or diminish protections for our national monuments. I urge you to support our public lands and waters and recommend that our current national monuments remain as they are today.\n\nSincerely,\nNic Brooksher\n  Baton Rouge, LA 70808"
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
    
    # advance page and query
    query['po'] = i * query['rpp']
    response = requests.get(api + path, params=query)
    page = response.json()
    docs = page['documents']
    
    # save page with session engine
    values = [{'comment': doc['commentText']} for doc in docs]
    insert = table.insert().values(values)
    engine.execute(insert)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
<sqlalchemy.engine.result.ResultProxy object at 0x128383250>
<sqlalchemy.engine.result.ResultProxy object at 0x12837a790>
<sqlalchemy.engine.result.ResultProxy object at 0x12837ced0>
<sqlalchemy.engine.result.ResultProxy object at 0x12838b110>
<sqlalchemy.engine.result.ResultProxy object at 0x128386c50>
<sqlalchemy.engine.result.ResultProxy object at 0x128386b10>
<sqlalchemy.engine.result.ResultProxy object at 0x12838b290>
<sqlalchemy.engine.result.ResultProxy object at 0x12837a850>
<sqlalchemy.engine.result.ResultProxy object at 0x128386190>
<sqlalchemy.engine.result.ResultProxy object at 0x128390750>
<sqlalchemy.engine.result.ResultProxy object at 0x128396610>
<sqlalchemy.engine.result.ResultProxy object at 0x128394bd0>
<sqlalchemy.engine.result.ResultProxy object at 0x1265cef50>
<sqlalchemy.engine.result.ResultProxy object at 0x12837a290>
<sqlalchemy.engine.result.ResultProxy object at 0x12838f590>
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

