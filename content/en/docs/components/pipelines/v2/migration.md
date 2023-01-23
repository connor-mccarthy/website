+++
title = "Migrating from v1"
description = "v1 to v2 migration instructions and breaking changes"
weight = 1

+++

## Terminology
* **SDK v1**: The `1.x.x` versions of the KFP SDK. `1.8` is the greatest [minor version](https://semver.org/#:~:text=MINOR%20version%20when%20you%20add%20functionality%20in%20a%20backwards%20compatible%20manner) of the v1 KFP SDK that will be released.
* **SDK v1 v2-namespace**: Refers to the v1 KFP SDK's `v2` module (i.e., `from kfp.v2 import *`), which permits access to v2 authoring syntax and IR YAML compilation from the v1 SDK. You should assume that references to the v1 SDK *do not* refer to the v2-namespace unless explicitly stated.
* **SDK v2**: The `2.x.x` versions of the KFP SDK. Uses the v2 authoring syntax and compiles to IR YAML.


## Non-breaking changes
Except in some rare cases, KFP SDK v2 is backward compatible with KFP SDK v1 v2-namespace user code.

The following documents non-breaking changes where we suggest you migrate your code to the "New usage", even though the "Previous usage" will still work with warnings.

### Import namespace

KFP SDK v1 v2-namespace imports (`from kfp.v2 import *`) should be converted to imports from the primary namespace (`from kfp import *`).

**Change:** Remove the `.v2` module from any KFP SDK v1 v2-namespace imports.

<table>
<tr>
<th>Previous usage</th>
<th>New usage</th>
</tr>
<tr>
<td>
  
```python
from kfp.v2 import dsl
from kfp.v2 import compiler

@dsl.pipeline(name='my-pipeline')
def pipeline():
  ...

compiler.Compiler().compile(...)
```
  
</td>
<td>

```python
from kfp import dsl
from kfp import compiler

@dsl.pipeline(name='my-pipeline')
def pipeline():
  ...

compiler.Compiler().compile(...)
```

</td>
</tr>
</table>

#### output_component_file parameter

In KFP SDK v2, components are [compilable](https://www.kubeflow.org/docs/components/pipelines/v2/author-a-pipeline/components/#compile-save-a-component) and [loadable](https://www.kubeflow.org/docs/components/pipelines/v2/author-a-pipeline/components/#load-a-component) in the same ways you compile and load pipelines.

The approach of compiling your pipeline via the `@dsl.component` decorator's `output_component_file` parameter is deprecated in KFP v2. If you choose to still use this parameter, your pipeline will be compiled to [IR YAML](https://www.kubeflow.org/docs/components/pipelines/v2/compile-a-pipeline/#ir-yaml) instead of v1 component YAML.

**Change:** Remove uses of `output_component_file`. Replace with a call to the compiler's [`.compile`](mpiler.html#kfp.compiler.Compiler.compile) method.


<table>
<tr>
<th>Previous usage</th>
<th>New usage</th>
</tr>
<tr>
<td>
  
```python
from kfp.v2.dsl import component

@component(output_component_file='my_component.yaml')
def my_component(input: str):
   ...
```
  
</td>
<td>

```python
from kfp.dsl import component
from kfp import compiler

@component()
def my_component(input: str):
   ...

compiler.Compiler().compile(my_component, 'my_component.yaml')
```

</td>
</tr>
</table>

#### Pipeline package file extension
The KFP compiler will compile your pipeline according to the extension provided to the compiler (`.yaml` or `.json`).

In KFP SDK v2, YAML is the default serialization format for [IR](https://www.kubeflow.org/docs/components/pipelines/v2/compile-a-pipeline/#ir-yaml).

**Change:** Convert compilation code that uses the a `.json` extension to use a `.yaml` extension.

<table>
<tr>
<th>Previous usage</th>
<th>New usage</th>
</tr>
<tr>
<td>
  
```python
from kfp.v2 import compiler
# .json extension, deprecated format
compiler.Compiler().compile(pipeline, package_path='my_pipeline.json')
```
  
</td>
<td>

```python
from kfp import compiler
# .yaml extension, preferred format
compiler.Compiler().compile(pipeline, package_path='my_pipeline.yaml')
```

</td>
</tr>
</table>

### Breaking changes
#### Remove primary namespace
Objects imported from the v1 namespace are no longer supported by the v2 compiler. KFP SDK v2 makes the KFP SDK v1 v2-namespace authoring styles the primary authoring style.

#### Drop support for Python 3.6
KFP SDK v2 supports Python >=3.7 and <3.11.

#### CLI output change
The [KFP CLI](https://www.kubeflow.org/docs/components/pipelines/v2/cli/) is more consistent, readable, and parsable. Existing output parsers may fail to parse the new output.


#### .after referencing upstream task in a ParallelFor sub-DAG

The following pipeline expression canont be compiled in KFP SDK v2:

```python
with dsl.ParallelFor(...):
    t1 = comp()
t2 = comp().after(t1)
```

This usage was primarily used by KFP SDK v1 users who implemented a custom ParallelFor fan-in. Fan-in from ParallelFor is not supported in KFP SDK v1. Support for ParallelFor fan-in is planned for KFP v2.

