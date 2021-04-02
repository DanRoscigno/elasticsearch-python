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
  "name": "nodejs_example",
  "role_descriptors": {
    "nodejs_example_writer": {
      "cluster": ["monitor"],
      "index": [
        {
          "names": ["game-of-thrones"],
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
  "name" : "nodejs_example",
  "api_key" : "zMWKady5T4OxaJWAuXCgBg"
}
```

To use the above key in your Node.js code, concatenate the `id` and `api_key`
values (with a `:` separator) and pipe the string to `base64` like so:
```
echo -n 'nx9OEngBZE2R_LhmKJgG:zMWKady5T4OxaJWAuXCgBg' | base64
```

Output is:
```
bng5T0VuZ0JaRTJSX0xobUtKZ0c6ek1XS2FkeTVUNE94YUpXQXVYQ2dCZw==
```

Edit the configuration file created earlier and add the above API Key.  You can have more than one set of configurations, below is one for Cloud and one for self managed:
`config/default.json`:
```
{
  "elastic-cloud": {
    "cloudID": "deploymentname:deploymentconnectiondetails",
    "username": "elastic",
    "password": "longpassword",
    "apiKey": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=="
  },
  "elastic-self-managed": {
    "node": "https://192.168.33.26:9200",
    "username": "elastic",
    "password": "changeme",
    "apiKey": "ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZzzzzzzzzz=="
  }
}
```

Now the API Key can be used in place of the username and password.  The client connection becomes:

```
const client = new Client({
  cloud: {
    id: elasticConfig.cloudID
  },
  auth: {
    apiKey: elasticConfig.apiKey
  }
})
```
