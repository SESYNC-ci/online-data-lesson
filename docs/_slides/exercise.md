---
---

## Exercises

### Exercise 1

Identify the name of the census variable in the table of ACS variables whose
label includes "COUNT OF THE POPULATION". Next use the Census API to collect
the data for this variable, for every county in the U.S. state with FIPS code
'24', into a [pandas](){:.pylib} DataFrame.

### Exercise 2

Get an API key for Regulations.gov, and collect 3 "pages" of comments posted on
the "Revised Definition of 'Waters of the United States'". Over half a million
were received before the comment period closed on April 15^th^, 2019. Modify
`schema.py` to save the comments into a new SQLite file.
