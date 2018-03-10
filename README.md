# Indigestion
Recursive sitemap crawler, with regex matching and ability to define your own parser function.

## Simple (Silly) Example
```python3
from ingestion_engine import Ingestion
from bs4 import BeautifulSoup


class ExampleIngestion(Ingestion):

    def parser(self, response):
        url = response.url
        status = response.status_code
        soup = BeautifulSoup(response.text)
        h1 = soup.find('title')
        if h1:
            h1 = h1.get_text()
        print(url, h1)
        with open('example.csv', 'a') as silly_example:
            silly_example.write('"{}","{}","{}"\n'.format(url, status, h1))


if __name__ == '__main__':
    c = ExampleIngestion('https://adaptworldwide.com/sitemap_index.xml', '.+')
    c.digest()
```
