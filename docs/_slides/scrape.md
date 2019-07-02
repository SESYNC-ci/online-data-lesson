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
print('\n'.join(doc.prettify().splitlines()[0:10]))
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
<!DOCTYPE html>
<html>
 <head>
  <link href="/s/b0dcca.css" rel="stylesheet" title="Default" type="text/css"/>
  <title>
   xkcd: Server Attention Span
  </title>
  <meta content="IE=edge" http-equiv="X-UA-Compatible"/>
  <link href="/s/919f27.ico" rel="shortcut icon" type="image/x-icon"/>
  <link href="/s/919f27.ico" rel="icon" type="image/x-icon"/>
~~~
{:.output}


===

Searching the document for desired content is the hard part. This search
uses a CSS query, to find the image below a section of the document with
attribute `id = comic`.



~~~python
img = doc.select('#comic > img').pop()
img
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
<img alt="Server Attention Span" src="//imgs.xkcd.com/comics/server_attention_span.png" title="They have to keep the adjacent rack units empty. Otherwise, half the entries in their /var/log/syslog are just 'SERVER BELOW TRYING TO START CONVERSATION *AGAIN*.' and 'WISH THEY'D STOP GIVING HIM SO MUCH COFFEE IT SPLATTERS EVERYWHERE.'"/>
~~~
{:.output}


===

It makes sense to query by CSS if the content being scraped always appears
the same in a browser; stylesheets are separate from delivered content.



~~~python
from textwrap import fill

print(fill(img['title'], width = 42))
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
They have to keep the adjacent rack units
empty. Otherwise, half the entries in
their /var/log/syslog are just 'SERVER
BELOW TRYING TO START CONVERSATION
*AGAIN*.' and 'WISH THEY'D STOP GIVING HIM
SO MUCH COFFEE IT SPLATTERS EVERYWHERE.'
~~~
{:.output}


===

## Was that so bad?

Pages designed for humans are increasingly harder to parse programmatically.

- Servers provide different responses based on client "metadata"
- Javascript often needs to be executed by the client
- The HTML `<table>` is drifting into obscurity (mostly for the better)

===

## HTML Tables

Sites with easilly accessible html tables nowadays may be specifically
geared toward non-human agents. The US Census provides some
documentation for their data services in a massive such table:

<http://api.census.gov/data/2015/acs5/variables.html>

===



~~~python
import pandas as pd

acs5_variables = pd.read_html(
    'https://api.census.gov/data/2016/acs/acs5/variables.html'
    )
vars = acs5_variables[0]
vars.head()
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
          Name  ... Values
0       AIANHH  ...    NaN
1      AIHHTLI  ...    NaN
2       AITSCE  ...    NaN
3         ANRC  ...    NaN
4  B00001_001E  ...    NaN

[5 rows x 9 columns]
~~~
{:.output}


<!--
failed_banks = pd.read_html(
  'https://www.fdic.gov/bank/individual/failed/banklist.html')
-->

===

We can use our data manipulation tools to search this unwieldy
documentation for variables of interest



~~~python
rows = (
    vars['Label']
    .str.contains(
        'household income',
        na = False,
        )
    )
for idx, row in vars.loc[rows].iterrows():
    print('{}:\t{}'.format(row['Name'], row['Label']))
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
B19013_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19013A_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19013B_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19013C_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19013D_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19013E_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19013F_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19013G_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19013H_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19013I_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19025_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19025A_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19025B_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19025C_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19025D_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19025E_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19025F_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19025G_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19025H_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19025I_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19049_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Total
B19049_002E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Householder under 25 years
B19049_003E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Householder 25 to 44 years
B19049_004E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Householder 45 to 64 years
B19049_005E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Householder 65 years and over
B19050_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19050_002E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Householder under 25 years
B19050_003E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Householder 25 to 44 years
B19050_004E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Householder 45 to 64 years
B19050_005E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Householder 65 years and over
B19202_001E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19202A_001E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19202B_001E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19202C_001E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19202D_001E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19202E_001E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19202F_001E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19202G_001E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19202H_001E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19202I_001E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19214_001E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19215_001E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Total (dollars)
B19215_002E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder!!Total (dollars)
B19215_003E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder!!Living alone!!Total (dollars)
B19215_004E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder!!Living alone!!Householder 15 to 64 years (dollars)
B19215_005E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder!!Living alone!!Householder 65 years and over (dollars)
B19215_006E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder!!Not living alone!!Total (dollars)
B19215_007E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder!!Not living alone!!Householder 15 to 64 years (dollars)
B19215_008E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder!!Not living alone!!Householder 65 years and over (dollars)
B19215_009E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder!!Total (dollars)
B19215_010E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder!!Living alone!!Total (dollars)
B19215_011E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder!!Living alone!!Householder 15 to 64 years (dollars)
B19215_012E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder!!Living alone!!Householder 65 years and over (dollars)
B19215_013E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder!!Not living alone!!Total (dollars)
B19215_014E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder!!Not living alone!!Householder 15 to 64 years (dollars)
B19215_015E:	Estimate!!Median nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder!!Not living alone!!Householder 65 years and over (dollars)
B19216_001E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)
B19216_002E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder (dollars)
B19216_003E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder (dollars)!!Living alone (dollars)
B19216_004E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder (dollars)!!Living alone (dollars)!!Householder 15 to 64 years (dollars)
B19216_005E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder (dollars)!!Living alone (dollars)!!Householder 65 years and over (dollars)
B19216_006E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder (dollars)!!Not living alone (dollars)
B19216_007E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder (dollars)!!Not living alone (dollars)!!Householder 15 to 64 years (dollars)
B19216_008E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Male householder (dollars)!!Not living alone (dollars)!!Householder 65 years and over (dollars)
B19216_009E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder (dollars)
B19216_010E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder (dollars)!!Living alone (dollars)
B19216_011E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder (dollars)!!Living alone (dollars)!!Householder 15 to 64 years (dollars)
B19216_012E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder (dollars)!!Living alone (dollars)!!Householder 65 years and over (dollars)
B19216_013E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder (dollars)!!Not living alone (dollars)
B19216_014E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder (dollars)!!Not living alone (dollars)!!Householder 15 to 64 years (dollars)
B19216_015E:	Estimate!!Aggregate nonfamily household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Female householder (dollars)!!Not living alone (dollars)!!Householder 65 years and over (dollars)
B25071_001E:	Estimate!!Median gross rent as a percentage of household income
B25092_001E:	Estimate!!Median selected monthly owner costs as a percentage of household income in the past 12 months!!Total
B25092_002E:	Estimate!!Median selected monthly owner costs as a percentage of household income in the past 12 months!!Housing units with a mortgage
B25092_003E:	Estimate!!Median selected monthly owner costs as a percentage of household income in the past 12 months!!Housing units without a mortgage
B25099_001E:	Estimate!!Median household income!!Total
B25099_002E:	Estimate!!Median household income!!Total!!Median household income for units with a mortgage
B25099_003E:	Estimate!!Median household income!!Total!!Median household income for units without a mortgage
B25119_001E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Total
B25119_002E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Owner occupied (dollars)
B25119_003E:	Estimate!!Median household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Renter occupied (dollars)
B25120_001E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)
B25120_002E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Owner occupied
B25120_003E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Owner occupied!!Housing units with a mortgage
B25120_004E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Owner occupied!!Housing units without a mortgage
B25120_005E:	Estimate!!Aggregate household income in the past 12 months (in 2016 inflation-adjusted dollars)!!Renter occupied
~~~
{:.output}

