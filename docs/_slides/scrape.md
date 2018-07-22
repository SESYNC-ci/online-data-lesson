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



===

The response is still binary, it takes a browser-like
parser to translate the raw content into an HTML document. [BeautifulSoup](){:.pylib} does
a fair job, while making no attempt to "render" a human readable page.













