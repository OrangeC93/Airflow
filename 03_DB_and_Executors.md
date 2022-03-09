## Introduction
- How to config Airflow for executing multiple tasks?
- Waht are the important parameters to know
- Waht are the different executors for scaling Airflow

## Default configuration
```console
airflow config get-value core sql_alchemy_conn  
sqlite:///home/airflow/airflow.db ## sqllite: execute one task after another 
airflow config get-value core executor
SequentialExecutor  ## SequentialExecuto execute one task after another 
```

Under folder dags, create parallel_dag.py
```python
from airflow import DAG
from airflow.operators.bash import BashOperator

from datetime import datetime

default_args = {
  'start_date': datetime(2020,1,1)
}

with DAG('parallel_dag', schedule_interval='@daily', default_args=default_args):
  task_1 = BashOperator(
    task_id = 'task_1',
    bash_command='sleep 3'
  )
  
  task_2 = BashOperator(
    task_id = 'task_2',
    bash_command='sleep 3'
  )
  
  task_2 = BashOperator(
    task_id = 'task_2',
    bash_command='sleep 3'
  )
  
  task_4 = BashOperator(
    task_id = 'task_4',
    bash_command='sleep 3'
  )
  
task1 >> [task2, task3] >> task4
```

Check DAG in Web UI
