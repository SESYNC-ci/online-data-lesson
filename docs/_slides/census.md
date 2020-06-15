---
---

## Specialized Packages

The third tier of access to online data is much preferred, if it
exists: a dedicated package in your programming language's repository
([PyPI](http://pypi.python.org) or [CRAN](http://cran.r-project.org)).

- Additional guidance on query parameters
- Returns data in native formats
- Handles all "encoding" problems

===

The [census](){:.pylib} package is a user-contributed suite of tools
that streamline access to the API.



~~~python
from census import Census

key = None
c = Census(key, year=2017)
c.acs5
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
<census.core.ACS5Client object at 0x7f734d50c128>
~~~
{:.output}


===

Compared to using the API directly via the [requests](){:.pylib} package:

**Pros**
- More concise code, quicker development
- Package documentation (if present) is usually more user-friendly than API documentaion.
- May allow seamless update if API changes

**Cons**
- No guarantee of updates
- Possibly limited in scope

===

Query the Census ACS5 survey for the variable `B19001_001E` (median annual household income,
in dollars) and each entity's `NAME`.



~~~python
variables = ('NAME', 'B19013_001E')
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

This code pulls the variables `NAME` and `B19001_001E` from all census tracts and all
counties in the state with ID `24` (Maryland). The [census](){:.pylib} package converts the JSON string 
into a Python dictionary. (No need to check headers.) 



~~~python
response = c.acs5.state_county_tract(
    variables,
    state_fips='24',
    county_fips=Census.ALL,
    tract=Census.ALL,
)
response[0]
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
{'NAME': 'Census Tract 105.01, Wicomico County, Maryland', 'tract': '010501', 'state': '24', 'county': '045', 'B19013_001E': 68652.0}
~~~
{:.output}


===

The Pandas `DataFrame()` constructor will accept the list of
dictionaries as the sole argument, taking column names from "keys". 
This code also removes values less than zero.



~~~python
df = (
  pd
  .DataFrame(response)
  .query("B19013_001E >= 0")
)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

The [seaborn](){:.pylib} package provides some nice, quick visualizations. Here
we create boxplots showing the income distribution among census tracts within
each county in Maryland.



~~~python
import seaborn as sns

sns.boxplot(
  data = df,
  x = 'county',
  y = 'B19013_001E',
)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .no-eval .text-document}

