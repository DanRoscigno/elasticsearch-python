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

## Switch to using an API Key
Up above the authentication to Elasticsearch was done by using the `elastic`
username and password.  Using API Keys are a best practice, see the [ApiKey docs](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-create-api-key.html) to learn about creating API Keys, and the [security privilege docs](https://www.elastic.co/guide/en/elasticsearch/reference/7.11/security-privileges.html) to understand which privileges are needed.  In the example below, the cluster `monitor` privilege is assigned to give read-only access to determine the cluster state etc.  The index privileges `create_index`, `write`, and `read` allow the creation of the index, write, update, delete, and read from that index.  The privilege `manage` is added to enable index refreshes.  If you are having a difficult time finding the right combination of privileges for your custom application you can get detailed logging by enabling [audit logging](https://www.elastic.co/guide/en/elasticsearch/reference/current/enable-audit-logging.html) on Elasticsearch.  

```
POST /_security/api_key
{
  "name": "python_example",
  "role_descriptors": {
    "python_example_writer": {
      "cluster": ["monitor"],
      "index": [
        {
          "names": ["test-index"],
          "privileges": ["create_index", "write", "read", "manage"]
        }
      ]
    }
  }
}
```

Output is:
```
{
  "id" : "nx9OEngBZE2R_LhmKJgG",
  "name" : "python_example",
  "api_key" : "zMWKady5T4OxaJWAuXCgBg"
}
```

Edit the configuration file (`example.ini`) created earlier and add the above ID and API Key.  You should remove the username and password once you have tested the API key, and change the `elastic` password using the Cloud Console.
`example.ini`:
```
[DEFAULT]
cloud_id = i-o-optimized-deployment:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
user = elastic
password = xxxxxxxxxxxxxxxxxxxxxxxx
apikey_id = nx9OEngBZE2R_LhmKJgG
apikey_key = zMWKady5T4OxaJWAuXCgBg
```

Now the API Key can be used in place of the `http_auth` username and password.  The connection becomes:

```
es = Elasticsearch(
    cloud_id=config['DEFAULT']['cloud_id'],
    api_key=(config['DEFAULT']['apikey_id'], config['DEFAULT']['apikey_key']),
)
```
