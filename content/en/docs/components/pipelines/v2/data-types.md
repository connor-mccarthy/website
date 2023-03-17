+++
title = "Data Types"
description = "Component and pipeline I/O types"
weight = 5
+++

There are two groups of types in KFP: parameters and artifacts. So far [Hello World pipeline][hello-world] and the examples in [Components][components] have demonstrated how to use input and output parameters.

Parameters and artifacts are indicated through component and pipeline function signatures and type annotations. KFP automatically tracks the way parameters and artifacts are passed between components and stores the data to using [ML Metadata][ml-metadata]. This enables out-of-the-box ML artifact lineage tracking and easily reproducible pipeline executions. Furthermore, KFP's strongly-typed components provide a data contract between components and between KFP and external systems.

### Parameters
Parameters are useful for passing small amounts of data between components and when the data does not represent a machine learning artifact such as a model, dataset, or more complex data type.

Specify parameter inputs and outputs using using built-in Python types annotations. KFP maps the Python type annotations used when authoring pipelines to the KFP types that will be shown in the UI according to the following table:

| Python object          | KFP type |
| ---------------------- | -------- |
| `str`                  | string   |
| `int`                  | number   |
| `float`                | number   |
| `bool`                 | boolean  |
| `typing.List` / `list` | object   |
| `typing.Dict` / `dict` | object   |

As with normal Python function arguments, input parameters can have default values, indicated in the standard way: `def func(my_string: str = 'default'):` 

Under the hood KFP passes all parameters to and from components by serializing them as JSON.

For [Lightweight Python Components][lightweight-python-components] and [Containerized Python Components][containerized-python-components], parameter serialization and deserialiation is invisible to the user; KFP handles this automatically. For [Container Components][container-component], input parameter deserialization is invisible to the user; KFP passes inputs to the component automatically. However, to create outputs from a Container Component, the user code in the container component must handle serializing the output parameters as described in [Container Components: Creating component outputs][container-component-outputs].

#### Input parameters
Using input parameters is very easy. Simply annotate your component function with the types and, optionally, defaults. As shown in the following comprehensive pipeline, this is the same for Python Components, Container Components, and Pipelines:

```python
@dsl.component
def python_comp(
    string: str = 'hello',
    integer: int = 1,
    floating_pt: float = 0.1,
    boolean: bool = True,
    dictionary: Dict = {'key': 'value'},
    array: List = [1, 2, 3],
):
    print(string)
    print(integer)
    print(floating_pt)
    print(boolean)
    print(dictionary)
    print(array)


@dsl.container_component
def container_comp(
    string: str = 'hello',
    integer: int = 1,
    floating_pt: float = 0.1,
    boolean: bool = True,
    dictionary: Dict = {'key': 'value'},
    array: List = [1, 2, 3],
):
    return dsl.ContainerSpec(
        image='alpine',
        command=['sh', '-c', """echo $0 $1 $2 $3 $4 $5 $6"""],
        args=[
            string,
            integer,
            floating_pt,
            boolean,
            dictionary,
            array,
        ])

@dsl.pipeline
def my_pipeline(
    string: str = 'Hey!',
    integer: int = 100,
    floating_pt: float = 0.1,
    boolean: bool = False,
    dictionary: Dict = {'key': 'value'},
    array: List = [1, 2, 3],
):
    python_comp(
        string='howdy',
        int=integer
        array=[4, 5, 6],
    )
    container_comp(
        string=string,
        integer=20,
        dictionary={'other key': 'other val'},
        boolean=boolean,
    )
```

#### Output parameters

For Python Components and pipelines, output parameters are indicated via return annotations:

```python
from kfp import dsl

@dsl.component
def my_comp() -> int:
    return 1

@dsl.pipeline
def my_pipeline() -> int:
    task = my_comp()
    return task.output
```

You can specify multiple output parameters using a `typing.NamedTuple` and referencing output parameters by name:

```python
from kfp import dsl
from typing import NamedTuple

@dsl.component
def my_comp() -> NamedTuple('outputs', [('my_int', int), ('my_string', str)]):
    outputs = NamedTuple('outputs', [('my_int', int), ('my_string', str)])
    return outputs(1, 'hello')

@dsl.pipeline
def my_pipeline() -> -> NamedTuple('pipeline_outputs', [('an_integer', int), ('a_string', str)]):
    task = my_comp()
    pipeline_outputs = NamedTuple('pipeline_outputs', [('an_integer', int), ('a_string', str)])
    return pipeline_outputs(task.outputs['my_int'], task.outputs['my_string'])
```


For Container Components, output parameters are indicated via a `dsl.OutputPath` annotation. At runtime, container components with this annotation will be automatically passed a system-generated path to which they should write the output.

```python
from kfp import dsl

@dsl.container_component
def say_hello(text_path: dsl.OutputPath(str)):
    return dsl.ContainerSpec(image='alpine', command=['echo Hello! > $0'], args=text_path)
```


[Container Components: Create component outputs][container-component-outputs] describes the `dsl.OutputPath annotation in more detail.

### Artifacts

Most machine learning pipelines aim to create one or more machine learning artifacts, such as a model, dataset, evaluation metrics, etc.

KFP provides first-class support for creating machine learning artifacts via the `dsl.Artifact` class and its subclasses. KFP maps these artifacts to their underlying [ML Metadata][ml-metadata] schema title.

| DSL object                    | Artifact schema title              |
| ----------------------------- | ---------------------------------- |
| `Artifact`                    | system.Artifact                    |
| `Dataset`                     | system.Dataset                     |
| `Model`                       | system.Model                       |
| `Metrics`                     | system.Metrics                     |
| `ClassificationMetrics`       | system.ClassificationMetrics       |
| `SlicedClassificationMetrics` | system.SlicedClassificationMetrics |
| `HTML`                        | system.HTML                        |
| `Markdown`                    | system.Markdown                    |

`Artifact`, `Dataset`, `Model`, and `Metrics` are the most frequently used artifact types. On the [KFP open source][kfp-open-source] UI, `ClassificationMetrics`, `SlicedClassificationMetrics`, `HTML`, and `Markdown` provide special UI rendering to make it easy to observe the contents of the artifact.

By default, an artifact is a reference to a cloud storage path within the bucket specified by the `@dsl.pipeline` decorator's [`pipeline_root` argument][dsl-pipeline]. [Non-default URIs][non-default-uris] describes another way to use artifacts.

When you use an input or output artifact annotation, the backend makes the contents of the cloud storage path available via the local filesystem, enabling components to read and create artifacts while only interacting with the local filesystem. You can access the artifact's local filesystem location via it's `.path` attribute. Artifacts also have `.uri`, `.name`, and `.metadata` attributes.

#### Input artifact

Artifact input annotations are always used alongside the `Input`/`Output` type markers to indicate their I/O type.

Input artifact annotations are included via an `Input[<Artifact class>]` annotation. The `count_rows` component demonstrates use of an input dataset by counting the number of rows in the `dataset` input:

```python
from kfp import dsl
from kfp.dsl import Dataset
from kfp.dsl import Input

@dsl.component
def count_rows(dataset: Input[Dataset]) -> int:
    with open(dataset.path) as f:
        lines = f.readlines()
    
    print('Information about the artifact:')
    print('Name:', dataset.name)
    print('URI:', dataset.uri)
    print('Metadata:', dataset.metadata)
    
    return len(lines)
```

Input artifacts should be treated as immutable. You should not try to modify the contents of `dataset.path` and any changes to `dataset.metadata` will not have an effect on the artifact's metadata in [ML Metadata][ml-metadata].

Input artifacts are used similarly Container Components. The following Container Component `container_count_rows` is equivalent to the previous Python component `count_rows`:

```python
from kfp import dsl

@dsl.container_component
def container_count_rows(dataset: Input[Dataset]):
    return dsl.ContainerSpec(image='alpine',
                             command=['sh', '-c', "wc -l $0 | awk '{ print $1 }'"],
                             args=[dataset.path])
```

Input artifacts are also used similarly in pipelines:

```python
from kfp import dsl

@dsl.pipeline
def my_pipeline(dataset: Input[Dataset]):
    count_rows(dataset=dataset)
```

#### Output artifact

Output artifacts are used similarly to input artifacts, but instead use the `Output` type marker. As a component author, you are responsible for creating the artifact at the artifact at the artifact's `.path` or `.uri` destination. You may also update the artifact's `.metadata`.

```python
from kfp import dsl
from kfp.dsl import Dataset
from kfp.dsl import Output

@dsl.component
def create_dataset(dataset: Output[Dataset]):
    with open(dataset.path, 'w') as f:
        f.writelines(['dataset row 1', 'dataset row 2'])

    dataset.metadata['dataset_type'] = 'dummy'
```

<!-- TODO: setting metadata -->

<!-- TODO -->
<!-- #### Lists of artifacts -->
<!-- #### Non-default URIs -->



<!-- TODO: how to use -->

[hello-world]: TODO
[components]: TODO
[lightweight-python-components]: TODO
[containerized-python-components]: TODO
[container-component]: TODO
[container-component-outputs]: TODO
[ml-metadata]: https://github.com/google/ml-metadata
[kfp-open-source]: TODO
[dsl-pipeline]: TODO
[non-default-uris]: TODO