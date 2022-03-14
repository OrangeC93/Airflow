## Introduction
How to extend features and functionalities

How to plugin system works

How to create your own operator

## Plugin System
Airflow is not only powerful because you can code your data propellants using Python, but also because you can extend its functionalities and features so that at the end you will get to know, for instance, fitting with your needs. How to customize in airflow?
- Customize operators: so either you create operators, extending functionalities of existing operators or you can create your own operators

What about if you have some tools with which you are interacting with from your data pipelines and you would like to monitor from your every instance which one is on and which one is off? you could definitely do that by adding a new view.
- In your every instance, you can customize the user interface of airflow as much as you need.

Then a new tool came out and you want to interact with that new tool.
- Hooks

How
- Create plugin folder
  - AirflowPluginClass, complicated
  - In airflow 2.0: AirflowPluginClass is only use for customizing UI, instead, create Regular python modules inside of the folder plugins, Then you put the files corresponding to your plugin, for example, your new operator. You will be able to import it directly from your data pipelines

Lazy loaded: Plugins are lazy, loaded, which means whenever you add a new plugin in your every instance, you have to start it, otherwise it won't work.

## Creating a hook interacting with Elasticsearch
Goal: create a hook to interact with ElasticSearch and a operator to transfer data fro Congress to ElasticSearch

Create plugin/elasticsearch_plugin/hooks/elastic_hook.py
```
from airflow.hooks.base import BaseHook

from elasticsearch import Elasticsearch

class ElasticHook(BaseHook):
  
  def __init__(self, conn_id='elasticsearch_default',*args, **kwargs):
    super().__init__(*args,**kwargs)
    conn = self.get_connection(conn_id)
    
    conn_config = {}
    hosts = []
    
    if conn.host:
      hosts = conn.host.split(',')
    if conn.port:
      conn_config['port'] = int(conn.port)
    if conn.login:
      conn_config['http_auth'] == (conn.login, conn.password)
      
    self.es = Elasticsearch(host, **conn_config)
    self.index = conn.schema
    
    
  def info(self):
    return self.es.info()
  
  def set_index(self, index):
    self.index = index
  
  
  # With that method. We are able to add document, we are able to add data.
  def add_doc(self, index, doc_type, doc):
    self.set_index(index)
    res = self.es.index(index=index, doc_type=doc_type, doc=doc)
    return res
    
```

Then create a elasticsearch_dag.py

```python
import airflow impor DAG
from elasticsearch_plugin.hooks.elestic_hook import ElasticHook
from airflow.operators.python import PythonOperator
from datetime import datetime

default_args = {
  'start_date': datetime(2020,1,1)
}

def _print_es_info():
  hook = ElasticHook()
  print(hook.info())

with DAG('elasticsearch_dag', schedule_interval='@daily',
         default_args=default_args, catchup=False) as dag:
         
         print_es_info = PythonOperator(
          task_id = 'print_es_info',
          python_callable=_print_es_info
         )
```

Test in UI

## Creating the PostgresToElasticOperator
