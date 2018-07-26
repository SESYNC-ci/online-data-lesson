---
---

## REST API

The US Census Burea provides access to its vast stores of demographic
data via their API at <https://api.census.gov>.

===

The **I** in **API** is all the buttons and dials on the same kind of
black box you need a **GUI** for (it's the same **I**).  Instead of
interfacing with a user, those buttons and dials are meant for another
software application.

In the case of the Census, the main component of the application is
some relational database management system. There probabably are
several **GUI**s designed for humans to query the Census database; the
Census API is meant for communication between your program
(i.e. script) and their application.
{:.notes}

===

Inspect [this URL](https://api.census.gov/data/2015/acs5?get=NAME&for=county&in=state:24#irrelephant){:target="_blank"} in your browser.

In a RESTful web service, the already universal system for
transferring data over the internet, known as HTTP is half of the
interface. All you really need is documentation for how to construct
the URL in a standards compliant way that the service will accept.
{:.notes}

===

| Section           | Description                                                             |
|-------------------+-------------------------------------------------------------------------|
| `https://`        | **scheme**                                                              |
| `api.census.gov`  | **authority**, or simply host if there's no user authentication         |
| `/data/2015/acs5` | **path** to a resource within a hierarchy                               |
|-------------------+-------------------------------------------------------------------------|
| `?`               | beginning of the **query** component of a URL                           |
| `get=NAME`        | first query parameter                                                   |
| `&`               | query parameter separator                                               |
| `for=county`      | second query parameter                                                  |
| `&`               | query parameter separator                                               |
| `in=state:*`      | third query parameter                                                   |
|-------------------+-------------------------------------------------------------------------|
| `#`               | beginning of the **fragment** component of a URL                        |
| `irrelevant`      | the fragment is a client side pointer, it isn't even sent to the server |

===

```{python title="{{ site.handouts[0] }}"}
path = 'https://api.census.gov/data/2016/acs/acs5'
query = {
  'get':'NAME,B19013_001E',
  'for':'tract:*',
  'in':'state:24',
}
response = requests.get(path, params=query)
response
```

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

```{python title="{{ site.handouts[0] }}"}
for k, v in response.headers.items():
    print('{}: {}'.format(k, v))
```

===

## Response Content

Use a JSON reader to extract a Python object. To read it into
a Panda's `DataFrame`, use Panda's `read_json`.

```{python title="{{ site.handouts[0] }}"}
data = pd.read_json(response.content)
data.head()
```

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