---
---

## Exercises

### Exercise 1

Identify the name of the census variable in the table of ACS variables whose
label includes "COUNT OF THE POPULATION". Next, use the Census API to collect
the data for this variable, for every county in the U.S. state with FIPS code
'24', into a [pandas](){:.pylib} DataFrame.

### Exercise 2

Request an [API key for data.gov], which will enable you to access the FoodData
Central API. Use the API to collect 3 "pages" of food results matching a search 
term of your choice. Modify `schema.py` to save the names and sugar contents of the
foods into a new SQLite file.

[API key for data.gov]: https://api.data.gov/signup/