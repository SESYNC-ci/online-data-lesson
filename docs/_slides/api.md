---
---

## Web Services

The US Census Burea provides access to its vast stores of demographic
data over the Web via their API at <https://api.census.gov>.

===

The **I** in **GUI** is for interface---its the same in **API**, where buttons
and drop-down menus are replaced by functions and object attributes.

Instead of interfacing with a user, this kind of **i**nterface is suitable for
use in **p**rogramming another software **a**pplication. In the case of the
Census, the main component of the application is some relational database
management system. There probabably are several GUIs designed for humans to
query the Census database; the Census API is meant for communication between
your program (i.e. script) and their application.
{:.notes}

===

Inspect [this URL](https://api.census.gov/data/2015/acs5?get=NAME&for=county&in=state:24#irrelephant){:target="_blank"} in your browser.

In a web service, the already universal system for
transferring data over the internet, known as HTTP is half of the
interface. All you really need is documentation for how to construct
the URL in a standards compliant way that the service will recognize.
{:.notes}

===

| Section | Description |  
|---+---|
| `https://`        | **scheme** |
| `api.census.gov`  | **authority**, or simply domain if there's no user authentication |
| `/data/2015/acs5` | **path** to a resource within a hierarchy |
|---+---|
| `?`          | beginning of the **query** component of a URL |
| `get=NAME`   | first query parameter |
| `&`          | query parameter separator |
| `for=county` | second query parameter |
| `&`          | query parameter separator |
| `in=state:*` | third query parameter |
|---+---|
| `#`          | beginning of the **fragment** component of a URL |
| `irrelevant` | a document section, it isn't even sent to the server |

===



~~~python
path = 'https://api.census.gov/data/2017/acs/acs5'
query = {
  'get': 'NAME,B19013_001E',
  'for': 'tract:*',
  'in': 'state:24',
}
response = requests.get(path, params=query)
response
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
<Response [200]>
~~~
{:.output}


===

## Response Header

The response from the API is a bunch of 0s and 1s, but part of the
HTTP protocol is to include a "header" with information about how
to decode the body of the response.

===

Most REST APIs return as the "content" either:

1. Javascript Object Notation (JSON)
  - a UTF-8 encoded string of key-value pairs, where values may be lists
  - e.g. `{'a':24, 'b': ['x', 'y', 'z']}`
1. eXtensible Markup Language (XML)
  - a nested `<tag></tag>` hierarchy serving the same purpose

===

The header from Census says the content type is JSON.



~~~python
response.headers['Content-Type']
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
'application/json;charset=utf-8'
~~~
{:.output}


===

## Response Content

Use a JSON reader to extract a Python object. To read it into
a Panda's `DataFrame`, use Panda's `read_json`.



~~~python
data = pd.read_json(response.content)
data.head()
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
                                                0            1  ...       3       4
0                                            NAME  B19013_001E  ...  county   tract
1  Census Tract 105.01, Wicomico County, Maryland        68652  ...     045  010501
2  Census Tract 5010.02, Carroll County, Maryland        75069  ...     013  501002
3  Census Tract 5077.04, Carroll County, Maryland        88306  ...     013  507704
4  Census Tract 5061.02, Carroll County, Maryland        84810  ...     013  506102

[5 rows x 5 columns]
~~~
{:.output}


===

## API Keys & Limits

Most servers request good behavior, others enforce it.

- Size of single query
- Rate of queries (calls per second, or per day)
- User credentials specified by an API key

===

From the Census FAQ [What Are the Query Limits?](https://www.census.gov/data/developers/guidance/api-user-guide.Query_Components.html):

>You can include up to 50 variables in a single API query and can make
>up to 500 queries per IP address per day...  Please keep in mind that
>all queries from a business or organization having multiple employees
>might employ a proxy service or firewall. This will make all of the
>users of that business or organization appear to have the same IP
>address.
