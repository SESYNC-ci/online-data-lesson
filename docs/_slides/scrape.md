---
---

## Requests

That "http" at the beginning of the URL for a possible data source is
a protocol---an understanding between a client and a server about how
to communicate. The client does not have to be a web browser, so long
as it knows the protocol. After all, [servers exist to
serve](https://xkcd.com/869/).

===

The [requests](){:.pylib} package provides a simple interface to
issueing HTTP requests and handling the response.



~~~python
import requests

response = requests.get('https://xkcd.com/869')
response
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
<Response [200]>
~~~
{:.output}


===

The response is still binary, it takes a browser-like parser to
translate the raw content into an HTML
document. [BeautifulSoup](){:.pylib} does a fair job, while making no
attempt to "render" a human readable page.



~~~python
from bs4 import BeautifulSoup

doc = BeautifulSoup(response.text, 'lxml')
'\n'.join(doc.prettify().splitlines()[0:10])
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
'<!DOCTYPE html>\n<html>\n <head>\n  <link href="/s/7d94e0.css" rel="stylesheet" title="Default" type="text/css"/>\n  <title>\n   xkcd: Server Attention Span\n  </title>\n  <meta content="IE=edge" http-equiv="X-UA-Compatible"/>\n  <link href="/s/919f27.ico" rel="shortcut icon" type="image/x-icon"/>\n  <link href="/s/919f27.ico" rel="icon" type="image/x-icon"/>'
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
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
[<img alt="Server Attention Span" src="//imgs.xkcd.com/comics/server_attention_span.png" title="They have to keep the adjacent rack units empty. Otherwise, half the entries in their /var/log/syslog are just 'SERVER BELOW TRYING TO START CONVERSATION *AGAIN*.' and 'WISH THEY'D STOP GIVING HIM SO MUCH COFFEE IT SPLATTERS EVERYWHERE.'"/>]
~~~
{:.output}


===

It makes sense to query by CSS if the content being scraped always appears
the same in a browser; stylesheets are separate from delivered content.



~~~python
img = doc.select_one('#comic > img')
img['title']
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
"They have to keep the adjacent rack units empty. Otherwise, half the entries in their /var/log/syslog are just 'SERVER BELOW TRYING TO START CONVERSATION *AGAIN*.' and 'WISH THEY'D STOP GIVING HIM SO MUCH COFFEE IT SPLATTERS EVERYWHERE.'"
~~~
{:.output}


===

## Was that so bad?

Pages designed for humans are increasingly harder to parse programmatically.

- Servers provide different responses based on client "metadata"
- JavaScript often needs to be executed by the client
- The HTML `<table>` is drifting into obscurity (mostly for the better)

===

## HTML Tables

Sites with easilly accessible html tables nowadays may be specifically
geared toward non-human agents. The US Census provides some
documentation for their data services in a massive such table:

<https://api.census.gov/data/2017/acs/acs5/variables.html>

===



~~~python
import pandas as pd

vars = (
  pd
  .read_html('https://api.census.gov/data/2017/acs/acs5/variables.html')
  .pop()
)
vars.head()
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
          Name            Label  ...   Group Unnamed: 8
0       AIANHH        Geography  ...     NaN        NaN
1       AIHHTL        Geography  ...     NaN        NaN
2        AIRES        Geography  ...     NaN        NaN
3         ANRC        Geography  ...     NaN        NaN
4  B00001_001E  Estimate!!Total  ...  B00001        NaN

[5 rows x 9 columns]
~~~
{:.output}


===

We can use our data manipulation tools to search this unwieldy
documentation for variables of interest



~~~python
idx = (
  vars['Label']
  .str
  .contains('Median household income')
)
vars.loc[idx, ['Name', 'Label']]
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
               Name                                              Label
11214   B19013_001E  Estimate!!Median household income in the past ...
11215  B19013A_001E  Estimate!!Median household income in the past ...
11216  B19013B_001E  Estimate!!Median household income in the past ...
11217  B19013C_001E  Estimate!!Median household income in the past ...
11218  B19013D_001E  Estimate!!Median household income in the past ...
11219  B19013E_001E  Estimate!!Median household income in the past ...
11220  B19013F_001E  Estimate!!Median household income in the past ...
11221  B19013G_001E  Estimate!!Median household income in the past ...
11222  B19013H_001E  Estimate!!Median household income in the past ...
11223  B19013I_001E  Estimate!!Median household income in the past ...
11932   B19049_001E  Estimate!!Median household income in the past ...
11933   B19049_002E  Estimate!!Median household income in the past ...
11934   B19049_003E  Estimate!!Median household income in the past ...
11935   B19049_004E  Estimate!!Median household income in the past ...
11936   B19049_005E  Estimate!!Median household income in the past ...
19332   B25099_001E           Estimate!!Median household income!!Total
19333   B25099_002E  Estimate!!Median household income!!Total!!Medi...
19334   B25099_003E  Estimate!!Median household income!!Total!!Medi...
19643   B25119_001E  Estimate!!Median household income in the past ...
19644   B25119_002E  Estimate!!Median household income in the past ...
19645   B25119_003E  Estimate!!Median household income in the past ...
~~~
{:.output}

