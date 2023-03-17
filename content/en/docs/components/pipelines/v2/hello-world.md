+++
title = "Hello World Pipeline"
description = "Create your first pipeline"
weight = 2
+++

Here is a simple pipeline that prints a greeting:

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

The `dsl.component` decorator and the `dsl.pipeline` turn your type-annotated Python functions into components and pipelines, respectively. The KFP SDK compiler compiles the domain-specific language (DSL) objects to a hermetic pipeline YAML file.

You can submit the YAML file to a KFP-conformant backend for execution. If you have already installed a [KFP open source backend deployment][installation] and [obtained the endpoint][get-endpoint] for your deployment, you can submit the pipeline for execution using the KFP SDK [`Client`][client]. We'll pass the argument `person_to_greet='World'`:

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

Click the link printed by the client to view the pipeline run in the UI to inspect the pipeline, which prints and returns `'Hello, world!'`.

In the next few examples, you'll learn more about the core concepts of authoring pipelines and how to create more expressive, useful pipelines.

[installation]: /docs/components/pipelines/v2/installation/
[client]: https://kubeflow-pipelines.readthedocs.io/en/2.0.0b13/source/client.html#kfp.client.Client
[get-endpoint]: TODO