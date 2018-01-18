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

Inspect [this URL](https://api.census.gov/data/2015/acs5?get=NAME,AIANHH&for=county&in=state:24#irrelevant) in your browser.

In a RESTful web service, the already universal system for
transferring data over the internet, known as HTTP is half of the
interface. All you really need is documentation for how to construct
the URL in a standards compliant way that the service will accept.
{:.notes}

===

| Section           | Description                                                             |
| `https://`        | **scheme**                                                              |
| `api.census.gov`  | **authority**, or simply host if there's no user authentication         |
| `/data/2015/acs5` | **path** to a resource within a hierarchy                               |
| `?`               | beginning of the **query** component of a URL                           |
| `get=NAME,AIANHH` | first query parameter                                                   |
| `&`               | query parameter separator                                               |
| `for=county`      | second query parameter                                                  |
| `&`               | query parameter separator                                               |
| `in=state:*`      | third query parameter                                                   |
| `#`               | beginning of the **fragment** component of a URL                        |
| `irrelevant`      | the fragment is a client side pointer, it isn't even sent to the server |

===


~~~python
path = 'https://api.census.gov/data/2015/acs5'
query = {
  'get': 'NAME,AIANHH',
  'for': 'county',
  'in': 'state:24',
}
response = requests.get(path, params=query)
response
~~~
{:.input}
~~~
Out[1]: <Response [200]>
~~~
{:.output}



===

## Interpretting the response

The response from the API is a bunch of 0s and 1s, but part of the
HTTP protocol is to include a "header" with information about how
to decode the body of the response.

===

Most REST APIs return in the "body" on of these:

- Javascript Object Notation (JSON)
  - a UTF-8 encoded string of key-value pairs, where values may be lists
  - e.g. `{'a':24, 'b': ['x', 'y', 'z']}`
- eXtensible Markup Language (XML)
  - hierarchy of `<tag></tag>`s that do the same thing

===

The header from Census says the content type is JSON.


~~~python
for k, v in response.headers.items():
    print('{}: {}'.format(k, v))
~~~
{:.input}
~~~
Server: Apache-Coyote/1.1
Cache-Control: max-age=60, must-revalidate
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET,POST
Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept
Content-Type: application/json;charset=utf-8
Transfer-Encoding: chunked
Date: Thu, 18 Jan 2018 01:21:00 GMT
Strict-Transport-Security: max-age=31536000
~~~
{:.output}



===


~~~python
data = pd.read_json(response.content)
data
~~~
{:.input}
~~~
Out[1]: 
                                   0       1      2       3
0                               NAME  AIANHH  state  county
1          Allegany County, Maryland    None     24     001
2      Anne Arundel County, Maryland    None     24     003
3         Baltimore County, Maryland    None     24     005
4           Calvert County, Maryland    None     24     009
5          Caroline County, Maryland    None     24     011
6           Carroll County, Maryland    None     24     013
7             Cecil County, Maryland    None     24     015
8           Charles County, Maryland    None     24     017
9        Dorchester County, Maryland    None     24     019
10        Frederick County, Maryland    None     24     021
11          Garrett County, Maryland    None     24     023
12          Harford County, Maryland    None     24     025
13           Howard County, Maryland    None     24     027
14             Kent County, Maryland    None     24     029
15       Montgomery County, Maryland    None     24     031
16  Prince George's County, Maryland    None     24     033
17     Queen Anne's County, Maryland    None     24     035
18       St. Mary's County, Maryland    None     24     037
19         Somerset County, Maryland    None     24     039
20           Talbot County, Maryland    None     24     041
21       Washington County, Maryland    None     24     043
22         Wicomico County, Maryland    None     24     045
23        Worcester County, Maryland    None     24     047
24          Baltimore city, Maryland    None     24     510
~~~
{:.output}



===

## API Keys & Limits

Most servers request good behavior, others enforce it.

- Size of single query
- Rate of queries (calls per second, or per day)
- User credentials specified by an API key

===

## From the Census Bureau

[**What Are the Query Limits?**](https://www.census.gov/data/developers/guidance/api-user-guide.Query_Components.html)

>You can include up to 50 variables in a single API query and can make up to 500 queries per IP address per day...
>Please keep in mind that all queries from a business or organization having multiple employees might employ a proxy service or firewall. This will make all of the users of that business or organization appear to have the same IP address.
