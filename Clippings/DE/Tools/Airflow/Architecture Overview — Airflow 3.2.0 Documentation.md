---
title: "Architecture Overview — Airflow 3.2.0 Documentation"
source: "https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/overview.html"
author:
published:
created: 2026-04-19
description:
tags:
  - "clippings"
---
## Architecture Overview

Airflow is a platform that lets you build and run *workflows*. A workflow is represented as a [Dag](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html) (a Directed Acyclic Graph), and contains individual pieces of work called [Tasks](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html), arranged with dependencies and data flows taken into account.

![An example Airflow Dag, rendered in Graph](https://airflow.apache.org/docs/apache-airflow/stable/_images/edge_label_example.png)

A Dag specifies the dependencies between tasks, which defines the order in which to execute the tasks. Tasks describe what to do, be it fetching data, running analysis, triggering other systems, or more.

Airflow itself is agnostic to what you’re running - it will happily orchestrate and run anything, either with high-level support from one of our providers, or directly as a command using the shell or Python [Operators](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/operators.html).

## Airflow components

Airflow’s architecture consists of multiple components. The following sections describe each component’s function and whether they’re required for a bare-minimum Airflow installation, or an optional component to achieve better Airflow extensibility, performance, and scalability.

### Required components

A minimal Airflow installation consists of the following components:

- A [scheduler](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/scheduler.html), which handles both triggering scheduled workflows, and submitting [Tasks](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html) to the executor to run. The [executor](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/executor/index.html), is a configuration property of the *scheduler*, not a separate component and runs within the scheduler process. There are several executors available out of the box, and you can also write your own.
- A *Dag processor*, which parses Dag files and serializes them into the *metadata database*. More about processing Dag files can be found in [Dag File Processing](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/dagfile-processing.html)
- A *webserver*, which presents a handy user interface to inspect, trigger and debug the behaviour of Dags and tasks.
- A folder of *Dag files*, which is read by the *scheduler* to figure out what tasks to run and when to run them.
- A *metadata database*, usually PostgreSQL or MySQL, which stores the state of tasks, Dags and variables.
	Setting up a metadata database is described in [Set up a Database Backend](https://airflow.apache.org/docs/apache-airflow/stable/howto/set-up-database.html) and is required for Airflow to work.

### Optional components

Some Airflow components are optional and can enable better extensibility, scalability, and performance in your Airflow:

- Optional *worker*, which executes the tasks given to it by the scheduler. In the basic installation worker might be part of the scheduler not a separate component. It can be run as a long running process in the [CeleryExecutor](https://airflow.apache.org/docs/apache-airflow-providers-celery/stable/celery_executor.html "(in apache-airflow-providers-celery v3.18.0)"), or as a POD in the [KubernetesExecutor](https://airflow.apache.org/docs/apache-airflow-providers-cncf-kubernetes/stable/kubernetes_executor.html "(in apache-airflow-providers-cncf-kubernetes v10.16.0)").
- Optional *triggerer*, which executes deferred tasks in an asyncio event loop. In basic installation where deferred tasks are not used, a triggerer is not necessary. More about deferring tasks can be found in [Deferrable Operators & Triggers](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/deferring.html).
- Optional folder of *plugins*. Plugins are a way to extend Airflow’s functionality (similar to installed packages). Plugins are read by the *scheduler*, *Dag processor*, *triggerer* and *webserver*. More about plugins can be found in [Plugins](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/plugins.html).

## Deploying Airflow components

All the components are Python applications that can be deployed using various deployment mechanisms.

They can have extra *installed packages* installed in their Python environment. This is useful for example to install custom operators or sensors or extend Airflow functionality with custom plugins.

While Airflow can be run in a single machine and with simple installation where only *scheduler* and *webserver* are deployed, Airflow is designed to be scalable and secure, and is able to run in a distributed environment - where various components can run on different machines, with different security perimeters and can be scaled by running multiple instances of the components above.

The separation of components also allow for increased security, by isolating the components from each other and by allowing to perform different tasks. For example separating *Dag processor* from *scheduler* allows to make sure that the *scheduler* does not have access to the *Dag files* and cannot execute code provided by *Dag author*.

Also while single person can run and manage Airflow installation, Airflow Deployment in more complex setup can involve various roles of users that can interact with different parts of the system, which is an important aspect of secure Airflow deployment. The roles are described in detail in the [Airflow Security Model](https://airflow.apache.org/docs/apache-airflow/stable/security/security_model.html) and generally speaking include:

- Deployment Manager - a person that installs and configures Airflow and manages the deployment
- Dag author - a person that writes Dags and submits them to Airflow
- Operations User - a person that triggers Dags and tasks and monitors their execution

## Architecture Diagrams

The diagrams below show different ways to deploy Airflow - gradually from the simple “one machine” and single person deployment, to a more complex deployment with separate components, separate user roles and finally with more isolated security perimeters.

The meaning of the different connection types in the diagrams below is as follows:

- **brown solid lines** represent *Dag files* submission and synchronization
- **blue solid lines** represent deploying and accessing *installed packages* and *plugins*
- **black dashed lines** represent control flow of workers by the *scheduler* (via executor)
- **black solid lines** represent accessing the UI to manage execution of the workflows
- **red dashed lines** represent accessing the *metadata database* by all components

### Basic Airflow deployment

This is the simplest deployment of Airflow, usually operated and managed on a single machine. Such a deployment usually uses the LocalExecutor, where the *scheduler* and the *workers* are in the same Python process and the *Dag files* are read directly from the local filesystem by the *scheduler*. The *webserver* runs on the same machine as the *scheduler*. There is no *triggerer* component, which means that task deferral is not possible.

Such an installation typically does not separate user roles - deployment, configuration, operation, authoring and maintenance are all done by the same person and there are no security perimeters between the components.

![../_images/diagram_basic_airflow_architecture.png](https://airflow.apache.org/docs/apache-airflow/stable/_images/diagram_basic_airflow_architecture.png)

If you want to run Airflow on a single machine in a simple single-machine setup, you can skip the more complex diagrams below and go straight to the section.

### Distributed Airflow architecture

This is the architecture of Airflow where components of Airflow are distributed among multiple machines and where various roles of users are introduced - *Deployment Manager*, **Dag author**, **Operations User**. You can read more about those various roles in the [Airflow Security Model](https://airflow.apache.org/docs/apache-airflow/stable/security/security_model.html).

In the case of a distributed deployment, it is important to consider the security aspects of the components. The *webserver* does not have access to the *Dag files* directly. The code in the `Code` tab of the UI is read from the *metadata database*. The *webserver* cannot execute any code submitted by the **Dag author**. It can only execute code that is installed as an *installed package* or *plugin* by the **Deployment Manager**. The **Operations User** only has access to the UI and can only trigger Dags and tasks, but cannot author Dags.

The *Dag files* need to be synchronized between all the components that use them - *scheduler*, *triggerer* and *workers*. The *Dag files* can be synchronized by various mechanisms - typical ways how Dags can be synchronized are described in [Manage Dag files](https://airflow.apache.org/docs/helm-chart/stable/manage-dag-files.html "(in helm-chart v1.21.0)") of our Helm Chart documentation. Helm chart is one of the ways how to deploy Airflow in K8S cluster.

![../_images/diagram_distributed_airflow_architecture.png](https://airflow.apache.org/docs/apache-airflow/stable/_images/diagram_distributed_airflow_architecture.png)

### Separate Dag processing architecture

In a more complex installation where security and isolation are important, you’ll also see the standalone *Dag processor* component that allows to separate *scheduler* from accessing *Dag files*. This is suitable if the deployment focus is on isolation between parsed tasks. While Airflow does not yet support full multi-tenant features, it can be used to make sure that **Dag author** provided code is never executed in the context of the scheduler.

![../_images/diagram_dag_processor_airflow_architecture.png](https://airflow.apache.org/docs/apache-airflow/stable/_images/diagram_dag_processor_airflow_architecture.png)

Note

When Dag file is changed there can be cases where the scheduler and the worker will see different versions of the Dag until both components catch up. You can avoid the issue by making sure Dag is deactivated during deployment and reactivate once finished. If needed, the cadence of sync and scan of Dag folder can be configured. Please make sure you really know what you are doing if you change the configurations.

## Workloads

A Dag runs through a series of [Tasks](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html), and there are three common types of task you will see:

- [Operators](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/operators.html), predefined tasks that you can string together quickly to build most parts of your Dags.
- [Sensors](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/sensors.html), a special subclass of Operators which are entirely about waiting for an external event to happen.
- A [TaskFlow](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/taskflow.html) -decorated `@task`, which is a custom Python function packaged up as a Task.

Internally, these are all actually subclasses of Airflow’s `BaseOperator`, and the concepts of Task and Operator are somewhat interchangeable, but it’s useful to think of them as separate concepts - essentially, Operators and Sensors are *templates*, and when you call one in a Dag file, you’re making a Task.

## Control Flow

[Dags](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html) are designed to be run many times, and multiple runs of them can happen in parallel. Dags are parameterized, always including an interval they are “running for” (the [data interval](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dag-run.html#data-interval)), but with other optional parameters as well.

[Tasks](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/tasks.html) have dependencies declared on each other. You’ll see this in a Dag either using the `>>` and `<<` operators:

```
first_task >> [second_task, third_task]
fourth_task << third_task
```

Or, with the `set_upstream` and `set_downstream` methods:

```
first_task.set_downstream([second_task, third_task])
fourth_task.set_upstream(third_task)
```

These dependencies are what make up the “edges” of the graph, and how Airflow works out which order to run your tasks in. By default, a task will wait for all of its upstream tasks to succeed before it runs, but this can be customized using features like [Branching](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html#concepts-branching), [LatestOnly](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html#concepts-latest-only), and [Trigger Rules](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html#concepts-trigger-rules).

To pass data between tasks you have three options:

- [XComs](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/xcoms.html) (“Cross-communications”), a system where you can have tasks push and pull small bits of metadata.
- Uploading and downloading large files from a storage service (either one you run, or part of a public cloud)
- TaskFlow API automatically passes data between tasks via implicit [XComs](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/xcoms.html)

Airflow sends out Tasks to run on Workers as space becomes available, so there’s no guarantee all the tasks in your Dag will run on the same worker or the same machine.

As you build out your Dags, they are likely to get very complex, so Airflow provides several mechanisms for making this more sustainable, example [TaskGroups](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html#concepts-taskgroups) let you visually group tasks in the UI.

There are also features for letting you easily pre-configure access to a central resource, like a datastore, in the form of [Connections & Hooks](https://airflow.apache.org/docs/apache-airflow/stable/authoring-and-scheduling/connections.html), and for limiting concurrency, via [Pools](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/pools.html).

## User interface

Airflow comes with a user interface that lets you see what Dags and their tasks are doing, trigger runs of Dags, view logs, and do some limited debugging and resolution of problems with your Dags.

![../_images/dags.png](https://airflow.apache.org/docs/apache-airflow/stable/_images/dags.png)

It’s generally the best way to see the status of your Airflow installation as a whole, as well as diving into individual Dags to see their layout, the status of each task, and the logs from each task.

Was this entry helpful?