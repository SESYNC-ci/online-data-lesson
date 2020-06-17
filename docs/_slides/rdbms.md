---
---

## Paging & Stashing

A common strategy that web service providers take to balance their
load is to limit the number of records a single API request can
return. The user ends up having to flip through "pages" with the API,
handling the response content at each iteration. Options for stashing
data are:

1. Store it all in memory, write to file at the end.
1. Append each response to a file, writing frequently.
1. Offload these decisions to database management software.

The [data.gov](https://www.data.gov) API provides a case in point. 
Data.gov is a service provided by the U.S. federal government to make data available
from across many government agencies. It hosts a catalog of raw data and of many other
APIs from across government.
Among the APIs catalogued by data.gov is the [FoodData Central](https://fdc.nal.usda.gov/) API.
The U.S. Department of Agriculture maintains a data system of nutrition information 
for thousands of foods. 
We might be interested in the relative nutrient content of different fruits.
{:.notes}

To repeat the exercise below at home, request an API key at
https://api.data.gov/signup/, and store it in a file named `api_key.py`
in your working directory. The file should contain the single line 
`API_KEY = your many digit key`.
{:.notes}

===

Load the `API_KEY` variable by importing it from the file you saved it in.



~~~python
> from api_key import API_KEY
~~~
{:title="Console" .input}


===

Run an API query for all foods with `"fruit"` in their name.



~~~python
import requests

api = 'https://api.nal.usda.gov/fdc/v1/'
path = 'foods/search'
query = {
    'api_key':API_KEY,
    'query':'fruit',
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
Python dictionary called `doc`. To inspect the return, we can list 
the dictionary keys.



~~~python
> list(doc.keys())
~~~
{:title="Console" .input}


~~~
['foodSearchCriteria', 'totalPages', 'foods', 'totalHits', 'currentPage']
~~~
{:.output}


===

We can print the value associated with the key `totalHits` to see
how many foods matched our search term, `"fruit"`.



~~~python
> doc['totalHits']
~~~
{:title="Console" .input}


~~~
17833
~~~
{:.output}


===

The purported claimed number of results is much larger than the length
of the `foods` array contained in this response. The query returned only the
first page, with 50 items.



~~~python
> len(doc['foods'])
~~~
{:title="Console" .input}


~~~
50
~~~
{:.output}


===

The following commands prepare Python to connect to a
database-in-a-file, and create empty tables in the database if they
do not already exist (meaning that it is safe to re-run after you have
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
will map to columns in a table. Then create a session engine.



~~~python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, Text, Numeric

Base = declarative_base()

class Food(Base):
    __tablename__ = 'food'
    
    id = Column(Integer, primary_key=True)
    name = Column(Text)
    sugar = Column(Numeric)
    
engine = create_engine('sqlite:///fruits.db')
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)
~~~
{:title="{{ site.data.lesson.handouts[1] }}" .no-eval .text-document}


===

For each fruit, we'll store its name and the amount of sugar
(grams of sugar per 100 grams of fruit) found in the API response.



~~~python
> fruit = doc['foods'].pop()
+ fruit['description']
~~~
{:title="Console" .input}


~~~
'Fruit peel, candied'
~~~
{:.output}


===

Extract the names and values of the first ten nutrients for the first item returned by the query.



~~~python
> [ (nutrient['nutrientName'], nutrient['value']) for nutrient in fruit['foodNutrients'][:9] ]
~~~
{:title="Console" .input}


~~~
[('Protein', 0.34), ('Total lipid (fat)', 0.07), ('Carbohydrate, by difference', 82.74), ('Energy', 322.0), ('Alcohol, ethyl', 0.0), ('Water', 16.7), ('Caffeine', 0.0), ('Theobromine', 0.0), ('Sugars, total including NLEA', 80.68)]
~~~
{:.output}

===

### Step 3: Connect (and Initialize)



~~~python
from schema import Session, Food

session = Session()
engine = session.bind
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

You could inspect the fruit database now using any sqlite3 client: you
would find one empty "food" table with fields "id", "name", and "sugar".

===

Add a new `pageSize` parameter to request `100` documents per page.




~~~python
query['pageSize'] = 100
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

In each request, advance the query parameter `pageNumber` by one. 
The first record retrieved will be `pageNumber * pageSize`. 
Insert the fruits (the key:value pairs stored in `values`) 
in bulk to the database with `engine.execute()`.

In each iteration of the loop, we use a list comprehension to
extract the value corresponding to the amount of sugar from each
of the foods in the page of results returned by the query.
{:.notes}



~~~python
table = Food.metadata.tables['food']
for i in range(0, 10):
    
    # advance page and query
    query['pageNumber'] = i 
    response = requests.get(api + path, params=query)
    page = response.json()
    fruits = page['foods']
    
    # save page with session engine
    values = [{'name': fruit['description'],
               'sugar': next(iter([ nutrient['value'] for nutrient in fruit['foodNutrients'] if nutrient['nutrientName'][0:5] == 'Sugar' ]), None) } for fruit in fruits]
               
    insert = table.insert().values(values)
    engine.execute(insert)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
<sqlalchemy.engine.result.ResultProxy object at 0x7f74f1f91630>
<sqlalchemy.engine.result.ResultProxy object at 0x7f74f33927f0>
<sqlalchemy.engine.result.ResultProxy object at 0x7f74f44e9860>
<sqlalchemy.engine.result.ResultProxy object at 0x7f74f47227b8>
<sqlalchemy.engine.result.ResultProxy object at 0x7f74f3453550>
<sqlalchemy.engine.result.ResultProxy object at 0x7f74f4729e48>
<sqlalchemy.engine.result.ResultProxy object at 0x7f74f44f60b8>
<sqlalchemy.engine.result.ResultProxy object at 0x7f74f47336a0>
<sqlalchemy.engine.result.ResultProxy object at 0x7f74f44f65f8>
<sqlalchemy.engine.result.ResultProxy object at 0x7f74f4733b00>
~~~
{:.output}


===

View the records in the database by reading
everything we have so far back into a `DataFrame`.



~~~python
import pandas as pd

df = pd.read_sql_table('food', engine)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

Don't forget to disconnect from your database!



~~~python
engine.dispose()
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}

