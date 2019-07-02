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

The [census](){:.pylib} package is a user contributed suite of tools
that streamline access to the API.



~~~python
from census import Census

key = None
c = Census(key, year=2016)
c.acs5
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
<census.core.ACS5Client object at 0x7fcbfec612e8>
~~~
{:.output}


===

Compared to using the API directly via the [requests](){:.pylib} package:

**Pros**
- More concise code, quicker development
- Package documentation (if present) is usually more user friendly than API documentaion.
- May allow seemless update if API changes

**Cons**
- No guarantee of updates
- Possibly limited in scope

===

Query the Census ACS5 survey for the variable `B19001_001E` and each
entity's `NAME`.



~~~python
variables = ('NAME', 'B19013_001E')
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

The [census](){:.pylib} package converts the JSON string into a Python
dictionary. (No need to check headers.)



~~~python
response = c.acs5.state_county_tract(
    variables,
    '24',
    Census.ALL,
    Census.ALL
    )
response[0]
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


~~~
{'state': '24', 'NAME': 'Census Tract 1, Allegany County, Maryland', 'tract': '000100', 'county': '001', 'B19013_001E': 42292.0}
~~~
{:.output}


===

The Pandas `DataFrame()` constructor will accept the list of
dictionaries as the sole argument, taking column names from "keys".



~~~python
df = pd.DataFrame(response)
mask = df['B19013_001E'] == -666666666.0
df = df.loc[~mask, :]
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}


===

The [seaborn](){:.pylib} package provides some nice, quick visualizations.



~~~python
import seaborn as sns

sns.boxplot(
  data = df,
  x = 'county',
  y = 'B19013_001E',
)
~~~
{:title="{{ site.data.lesson.handouts[0] }}" .text-document}
![ ]({% include asset.html path="images/census/unnamed-chunk-5-1.png" %})
{:.captioned}
