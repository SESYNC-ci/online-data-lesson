---
---

## Aquiring Online Data

Data can be available on the web in many different forms. The difficulty you
will have aquiring that data for running analyses depends on which of three
approaches the data source requires.

===

## Scraping 🙁

If a web browser can read HTML and JavaScript and display a human readable page,
why can't you right a program (a "bot") to read HTML and JavaScript and store the
data?

===

## Web Service or API 😉

An Application Programming Interface (API, as opposed to GUI) that is compatible
with passing data around the internet using HTTP (Hyper-text Transfer Protocol).
This is not the fastest protocol for moving large datasets, but it is universal
(it underpins web browsers, after all).

===

## API Wrapper 😂

Major data providers can justify writing a package, specific to your
language of choice (e.g. Python or R), that facilitates accessing the
data they provide through a web service. Sadly ... not all do so.