## Introduction
- How to make your DAG cleaner
  - reading file A, B, C ... will have seperate task for each, how can we group thoese repetitive tasks togther and share same functionality in order to make your DAG cleaner
- How to choose one task or another
  - like, the next task will depend on the previous task, if user subscribe, they'll not get email, else they will get an email?
- How to exchange data between tasks?
  - next task will use the file from previous task
## Minimising DAG with SubDAGs
Turn off load_examples in .cfg file to remove all the DAG example on Airflow website

Example for this section(parallel.py): group task2 and task3 into one task by using subDag

``` python
from airflow.operators.subdag import SubDagOperator
from subdags.subdag_parallel_dag import subdag_parallel_dag

processing = SubDagOperator(
  task_id = 'processing_tasks',
  subdag=subdag_parallel_dag('parallel_dag','processing_tasks',default_args)
)

task_1>> processing >>task4
```

in order to do this, create a new folder called subdags, and a new file subdag_parallel_dag.py
- One very important thing here: you create a new dag with a dag id corresponding to the parent dag id with the child id, the default args must be the same as parent and you have to return the dag with the tasks that you want to group together
```
from airflow import DAG
from airflow.operator.bash import BashOperator
def subdag_parallel_dag(parent_dag_id, child_dag_id, default_args):
  with DAG(dag_id=f'{parent_dag_id}.{child_dag_id}', default_args) as dag:
    task_2 = BashOperator(
      task_id = 'task_2',
      bash_command='sleep 3'
  )
  
    task_3 = BashOperator(
      task_id = 'task_3',
      bash_command='sleep 3'
  )
  
  return dag
```

SubDAGs are not encouraged for use, three reasons:
- deadlocks: that means you might not be able to execute any more tasks in your instance at some point
- complex
- subdag has its own executer, in that case, with the subject that we have just added, we use the sequential executor by defalt even if in the configuration of airflow we have two local executor. So if you're able to execute multiple tasks in parallel, you won't be able to do taht inside subdag.

## TaskGrousp 





