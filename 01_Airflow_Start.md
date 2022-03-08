## Why Airflow
Typical Usecase: data pipeline to trigger ever day of 10PM
- Downloading Data(Request API) -> Processing Data(Spark) -> Storing Data(Intsert/Update database)

## What is Airflow
Apache Airflow is an ope source platform to programmatically author, schedule, and monitor workflows

Benefits: dynamic, scalability, ui, extensibility(if you want to integrate airflow with a new tool, you dont need to wait until airflow upgrade, you can create the plugin added to your app)

Key componenets:
- Web server: flask server with Gunicorn serving the UI
- Scheduler: daemon in charge of scheduling workflows
- Metastore: database where metadata are stored
- Executor: claass defining how your tasks should be executed
- Worker: process/subprocess executing your task

DAG: no loop!
- The work (tasks), and the order in which work should take place (dependencies), written in Python.


Operator
- Operators are the main building blocks of Airflow DAGs. They are classes that encapsulate logic to do a unit of work. 
- When you create an instance of an operator in a DAG and provide it with it's required parameters, it becomes a task. Many tasks can be added to a DAG along with their dependencies.
- Like: action operator, transfer operator, sensor operator

Task/Task Instance
- Task: Defines work by implementing an operator, written in Python.
- Task Instance: An instance of a task - that has been assigned to a DAG and has a state associated with a specific DAG run (i.e for a specific execution_date).

Workflow: a dialogue with operators, with tasks, with dependencies

Airflow is not a data streaming solution neither a data processing framework

## How Airflow works
One node architecture

![image](pics/one_node_architecture.png)

Multi nodes architecture

![image](pics/multi_node_architecture.png)

How it works:
- Add dag.py to folder dags, which will be parsed by web server and scheduler
- then schechuler create a dag run object, which has a status of running in metasotre
- next, the first task to executre i your data pipelein is scheduled
- when the task is ready to be triggered in your data pipeline, you have a task instance object
- once it's created, it will be sent to executer by the scheduler
- executer runs task, updates task isntacne object, the status of task instance object in metasotre
- once dag run complete, web server updates the UI that you're able t see that ok

![image](pics/how_it_works.png)

## Install airflow 2.0
- Install airflow vm, In local vscode, set up remote ssh to connect to vm. Connect to vm where a new vscode show up and you'd need to enter password
- Create a sandbox where you're gonna to set up airflow and python packages(avoid messing u with alread installed python packages)
- Inside sandbox, pip install apache-airflow == version and constraint python librarie, which is important because otherwise as soon as python dependency gets updated, you'll end up with trouble
- Airflow db init
- Airflow webserver -> local:8080 -> Airflow UI


