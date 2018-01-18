---
---

## Requests

That "http" at the beginning of the URL for a possible data source is
a protocol&mdash;an understanding between a client and a server about
how to communicate. The client does not have to be a web browser, so
long as it knows the protocol.

[Servers exist to serve, after all.](https://xkcd.com/869/)

===

The [requests](){:.pylib} package provides a simple interface to
issueing HTTP requests and handling the response.


~~~python
import requests

response = requests.get('https://xkcd.com/869')
response
~~~
{:.input}
~~~
Out[1]: <Response [200]>
~~~
{:.output}



===

The response is still binary, it takes a browser-like
parser to translate the raw content into an HTML document. [BeautifulSoup](){:.pylib} does
a fair job, while making no attempt to "render" a human readable page.


~~~python
from bs4 import BeautifulSoup

doc = BeautifulSoup(response.text, 'lxml')
doc
~~~
{:.input}
~~~
<!DOCTYPE html>
<html>
 <head>
  <script>
   (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-25700708-7', 'auto');
~~~
{:.output}



===

Searching the document for desired content is the hard part. This search
uses a CSS query, to find the image below a section of the document with
attribute `id = comic`.


~~~python
img = doc.select('#comic > img')
img
~~~
{:.input}
~~~
Out[1]: [<img alt="Server Attention Span" src="//imgs.xkcd.com/comics/server_attention_span.png" title="They have to keep the adjacent rack units empty. Otherwise, half the entries in their /var/log/syslog are just 'SERVER BELOW TRYING TO START CONVERSATION *AGAIN*.' and 'WISH THEY'D STOP GIVING HIM SO MUCH COFFEE IT SPLATTERS EVERYWHERE.'"/>]
~~~
{:.output}



===

It makes sense to query by CSS if the content being scraped always appears
the same in a browser; stylesheets are separate from delivered content.


~~~python
img[0]['title']
~~~
{:.input}
~~~
Out[1]: "They have to keep the adjacent rack units empty. Otherwise, half the entries in their /var/log/syslog are just 'SERVER BELOW TRYING TO START CONVERSATION *AGAIN*.' and 'WISH THEY'D STOP GIVING HIM SO MUCH COFFEE IT SPLATTERS EVERYWHERE.'"
~~~
{:.output}



===

## Range of complexity

Pages designed for humans are increasingly harder to parse programmatically.

- Servers provide different responses based on client "metadata"
- Javascript often needs to be executed by the client
- The HTML `<table>` is drifting into obscurity (mostly for the better)

===

## HTML Tables

Sites with easilly accessible html tables nowadays may be specifically geared toward
non-human agents. The US Census provides some documentation for their
data services in a massive such table:

<http://api.census.gov/data/2015/acs5/variables.html>

===


~~~python
import pandas as pd

# oh, no! the census broke!
#acs1_variables = pd.read_html('https://api.census.gov/data/2016/acs/acs1/profile/variables.html')

failed_banks = pd.read_html(
  'https://www.fdic.gov/bank/individual/failed/banklist.html')
~~~
{:.input}


<!--
===


~~~python
acs5_variables = acs5_variables[0]
acs5_variables.head()
~~~
{:.input}


===


~~~python
rows = acs5_variables['Concept'].str.contains('Household Income', na = False)
acs5_variables.loc[rows,]
~~~
{:.input}

-->
