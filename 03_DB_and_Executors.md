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
  
  task_3 = BashOperator(
    task_id = 'task_3',
    bash_command='sleep 3'
  )
  
  task_4 = BashOperator(
    task_id = 'task_4',
    bash_command='sleep 3'
  )
  
task1 >> [task2, task3] >> task4
```

Check DAG in Web UI

## Start scaling with local executor
Problems need to solve:
- What about if you want to execute multiple tasks in parallel?
- What about if you want to start scaling airflow as you will use i tin production

Two things:
- Change database by anothe one as sqllite doesnt' allow multiple writes in the same time
  - Use PostgresSQL which allows you to have multiple reads as well as multiple writes and it scales really well
- Change executer fro the sequential executer to local executer
  - Local executer allows you execute multiple tasks in parallel on the same machine. As soo as you execute a task, a set process is created and that task is executed inside that subprocess

Install postgres:
```console
sudo apt update
sudo apt install postgresql
sudo -u postgres psql
ALTER USER postgres PASSWORD 'postgres';

pip install 'apche-airflow[postgres]'## connect airflow with postgres
```
In file airflow.cfg: change sql_alchemy_conn => postgresql+psycopg2://postgres:postgres@localhost/postgres

``` console
airflow db check
```

Change executor
- In file airflow.cfg: change executor => LocalExecutor

Initialize postgres database

``` console
airflow db init
airflow users create -u admin -p admin -r admin -f Admin -l admin -e adming@airflow.com
airflow webserver
airflow scheduler
```

Check UI -> in parallel_dag we created last time, run it, we could see task 2 and task 3 are running at same time in tree chart and gantt chart

Next thing we want to solve: be able to execute an infinite number of tasks?

## Scale to the infinity with the Celery Executor
Inside of local executor, we have a queue, task is pushed inside queue, task then in pulled out and executed insid of a subprocess. However instead of a subprocess, we have a worker which pulls out the task from queue. As we can see we have a dedicated worke now which means we hav more resources and ultimately you're able to execute as many tasks as you need. The we just need to add a new worker, a new machine, and you can spead the tasks among those
There're tw executor can do the Scale: celery executer and community executer

Celery Executor:
Task are execute in workers(machines). The more workers, the more tasks can be executed.

How it works:
![image](pics/celery_work.png)

- Node3: instead of having queue inside of executor like we said above, that queue is outside of it. Which is a 3rd party tool and in that case it's redis where is an in-memory database and we can use as a queue system or a Message broker syste with the set of executer. Each time you execute a task, that task is pushed first in redis and then worke or machine will pull task from redis in order to execute it. 
- Change work_concurrency paramter to allow more tasks to be executed in worker

Takeaways:
- The executive order allows you to execute your tasks on multiple machines, whereas with local executor, you're able t execute your tasks on a single machine
- You have to install an external tool corresponding to the queue where your tasks will be pushed and pulled out by the workers, in this case, it's redis
- Each worker corresponds to a machine and so each machine has an instance running where the task will be executed. Therefore, you have to make sure that all of your machines share the same dependencies. For example, if you have a task interacting with AWS, you have to make sure these three work all install AWS module. Super important!!

Finally, you're able to execute as many tasks as you want with airflow.

## Scaling Airflow with the Celery Executor in action
```console
pip install 'apache-airflow[celery]'

sudo apt update
sudo apt install redis-server
sudo nano /etc/redis/redis.conf
```
then modify: supervised no => supervised systemd i redis.conf

```console
sudo systemctl restart redis.service
sudo systemctl status redis.service
```

then in airflow.cfg:
- change executor = CeleryExecutor
- broker_url = redis://localhost:6379/0
- result_backed = postgresql+psycopg2://postgres:postgres@localhost/postgres

If we want to interact with Redis from airflow, we need to install the extra package redis
```console
pip install 'apache-airflow[redis]'
```

flower is a UI where allow you to monitor your workers, the machine that will be used by airflow and then are executer wher your tasks will be executed. To start flower, open a new terminal:
```console
source sandbox/bin/activate
airflow celery flower
```
open localhost/5555 > flower UI

How can we add new machine, add a new walker to our airflow instance?
```console
source sandbox/bin/activate
airflow celery workder
```
- if you have error about result backend, need to back to .cfg file, change - result_backed = db+postgresql//postgres:postgres@localhost/postgres

Verify in Flower UI if we have a new worker?

Restart webserver and scheduler, then check Airflow UI, turn on parallel_dag see how celery executor works

## Concurrency, the parameters you must know
Parallelism: max number of tasks that you can execute in parallel for your entire airflow instance
- .cfg
- now run paralle_dag, we can see task2 and task3 cannot be run together as we expect

dag_concurrency: max number of tasks that you can execute in parallel for given DAG across all its diagrams. This number will naturally be limited by dag_concurrency. If you have 1 worker and want it to match your deployment’s capacity, worker_concurrency = parallelism.
- and in parallel_dag.py the DAG func, change catchup=True
- if set concurrency = 1 only applies to this specific DAG

max_active_runs_per_dag: the max number of dag that can be parallel for a given dag
- max_active_runs applies to this specific DAG
## Concurrency in practice


