[![Build Status](https://travis-ci.org/babbel/floto.svg?branch=master)](https://travis-ci.com/babbel/floto)

# floto
floto is a task orchestration tool based on AWS SWF (Simple Workflow Service) written in Python. It uses Python 3 and boto3, the AWS SDK for Python.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Introduction](#introduction)
- [Defining the Workflow's Logics](#defining-the-workflows-logic)
  - [Decider](#decider)
  - [Decider Specifications](#decider-specifications)
    - [JSON Representation of Decider Specifications](#json-representation-of-decider-specifications)
  - [Activity Tasks and Timers](#activity-tasks-and-timers)
    - [Retry Strategies of Activity Tasks](#retry-strategies-of-activity-tasks)
    - [Activity Task Inputs](#activity-task-inputs)
    - [Task IDs](#task-ids)
- [Activity Worker](#activity-worker)
  - [Activity Worker Heartbeats](#activity-worker-heartbeats)
- [Inputs and Results](#inputs-and-results)
- [Decider Daemon](#decider-daemon)
  - [Start Decider Daemon](#start-decider-daemon)
  - [Signal Child Workflow Execution](#signal-child-workflow-execution)
- [floto's simple SWF API](#flotos-simple-swf-api)
  - [Interface to SWF](#interface-to-swf)
  - [Start the Workflow](#start-the-workflow)
  - [Register Domains, Workflow Type and Activity Type](#register-domains-workflow-type-and-activity-type)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
## Introduction
The <a href="https://aws.amazon.com/swf/" target="_blank">AWS Simple Workflow Service</a> allows to 
manage distributed applications in a scalable, resilient and fault tolerant way.
It minimizes the complexity by decoupling the execution logic from the application worker. The 
Deciders, which handle the execution logic and the worker are stateless and therefore fault 
tolerant. Whenever something goes wrong the Deciders and worker can be restarted and pick up their 
work where they left off. Furthermore several Deciders and worker of the same kind can be run at 
the same time without interference of the workflow execution or result which again leads to 
higher resilience and scalability. Every step of a workflow execution is recorded by SWF and the 
history of events is provided to the Deciders when they are about to schedule tasks.

The process of implementing a SWF workflow can be somewhat tedious if you want to e.g. 
handle complex execution logic and treat task failures and time-outs.
floto solves this problem by providing a Python package which allows you to easily define the 
execution logic and activity worker.
For the impatient we provide a ["Getting started example"](examples/hello_world.py) of a
simple workflow.
The example shows the definition of a simple workflow with a single task. The task is defined and passed to the Decider. Furthermore an activity is defined so that the worker is able to executes the activity function on request. The Decider and the worker are started and the workflow execution is initiated. The single steps to define the components necessary to execute a workflow are discussed in more detail in the next sections.

## Defining the Workflow's Logics
The business logic of your distributed application is handled by so called Deciders. Deciders act on events like workflow start, task completion or task failure and schedule tasks that are to be executed. The logic itself is defined by a list of tasks. The tasks are then [passed to the decider](#inputs_and_results).
Let's get started with a simple example of three activities as depicted in figure 1.  In this example ActivityA and ActivityB are scheduled after the workflow start. ActivityC is executed once they are completed.

![alt tag](docs/images/decider_spec_01.png)

The Decider implements the application's business logic. The following code defines the execution logic as depicted in figure 1. In this example ``ActivityA`` and ``ActivityB`` are scheduled after the workflow start. ``ActivityC`` is executed once they are completed.


```python
from floto.specs.task import ActivityTask, DeciderSpec
from floto.specs import DeciderSpec
from floto.decider import Decider

activity_task_a = ActivityTask(name='ActivityA', version='v1')
activity_task_b = ActivityTask(name='ActivityB', version='v1')
activity_task_c = ActivityTask(name='ActivityC', version='v1', requires=[activity_task_a, activity_task_b])

decider_spec = DeciderSpec(domain='your_domain',
                           task_list='your_decider_task_list',
                           default_activity_task_list='your_activity_task_list',
                           activity_tasks=[activity_task_a, activity_task_b, activity_task_c])

Decider(decider_spec=decider_spec).run()
```
### Tasks
#### Activity Task
#### Generator
#### ChildWorkflow
#### Timer
#### Retry Strategy
