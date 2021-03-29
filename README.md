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


