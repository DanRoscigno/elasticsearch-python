# elasticsearch-python

Docs:
https://elasticsearch-py.readthedocs.io/en/v7.12.0/

Blog:
https://www.elastic.co/blog/elasticsearch-python-client-now-supports-asyncio

```
python -m pip install elasticsearch
python -m pip install elasticsearch[async]
```

## Create setup.py:
```
# Elasticsearch 7.x
elasticsearch>=7.0.0,<8.0.0
```

# Deploy Elastic Cloud

## Copy cloud_id username (elastic) and password for user

# Create config file `example.ini`

See https://docs.python.org/3/library/configparser.html
```
[DEFAULT]
cloud_id = i-o-optimized-deployment:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
user = elastic
password = xxxxxxxxxxxxxxxxxxxxxxxx
```

# Sample

This sample comes from the docs, only the use of `configparser` has been added.

```
from datetime import datetime
from elasticsearch import Elasticsearch
import configparser

config = configparser.ConfigParser()
config.read('example.ini')

es = Elasticsearch(
    cloud_id=config['DEFAULT']['cloud_id'],
    http_auth=(config['DEFAULT']['user'], config['DEFAULT']['password']),
)

doc = {
    'author': 'kimchy',
    'text': 'Elasticsearch: cool. bonsai cool.',
    'timestamp': datetime.now(),
}
res = es.index(index="test-index", id=1, body=doc)
print(res['result'])

res = es.get(index="test-index", id=1)
print(res['_source'])

es.indices.refresh(index="test-index")

res = es.search(index="test-index", body={"query": {"match_all": {}}})
print("Got %d Hits:" % res['hits']['total']['value'])
for hit in res['hits']['hits']:
    print("%(timestamp)s %(author)s: %(text)s" % hit["_source"])
```
