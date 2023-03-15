+++
title = "Hello World"
description = "Create your first pipeline"
weight = 1
+++

Here is a simple pipeline that prints a greeting and returns the greeting as a string:

```python
from kfp import dsl
from kfp import compiler


@dsl.component
def say_hello(name: str) -> str:
    hello_text = f'Hello, {name}!'
    print(hello_text)
    return hello_text


@dsl.pipeline
def hello_pipeline(person_to_greet: str) -> str:
    hello_task = say_hello(name=person_to_greet)
    return hello_task.output

compiler.Compiler().compile(addition_pipeline, 'pipeline.yaml')
```

Using the `dsl.component` decorator and the `dsl.pipeline`, we turn our Python functions into components and pipelines, respectively. Using the compiler, we can compile our pipeline to a hermetic pipeline YAML file, which can be executed by a KFP backend.

If you have a [KFP open source backend deployment and endpoint][installation], you can run the pipeline using the KFP SDK [`Client`][client]. We'll pass the argument `person_to_greet='World'`:

```python
from kfp.client import Client
client = Client(host='<MY-KFP-ENDPOINT>')
run = client.create_run_from_pipeline_package(
    'pipeline.yaml',
    arguments={
        'person_to_greet': 'World',
    },
)
```

Click the link printed by the client to view the pipeline run in the UI.

In the next few examples, you'll learn more about the core concepts of authoring pipelines and how to create more flexible, useful pipelines.

[installation]: /docs/components/pipelines/v2/installation/
[client]: https://kubeflow-pipelines.readthedocs.io/en/2.0.0b13/source/client.html#kfp.client.Client