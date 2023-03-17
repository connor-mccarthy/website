+++
title = "Container Components"
description = "Create a component from an arbitrary container"
weight = 4
+++

In KFP, each task execution corresponds to a container execution. This means that all components, even Python Components, are defined by an `image`, `command`, and `args`.

Python Components are unique because they mostly abstract the definition of the container away from the user, making it convenient to construct components that use pure Python. Under the hood, the KFP SDK sets the `image`, `commands`, and `args` to the values needed to execute the Python component for the user.

By comparison, Container Components enable component authors to set the `image`, `command`, and `args` directly. This makes it possible to author components that execute shell scripts, use other languages and binaries, etc., all from within the KFP Python SDK.

### Simple container component

This demonstratino starts with a simple `say_hello` component and gradually modifies it until it is equivalent to our `say_hello` component from the [Hello World Pipeline example][hello-world-pipeline]. Here is a simple Container Component:

```python
from kfp import dsl

@dsl.container_component
def say_hello():
    return dsl.ContainerSpec(image='alpine', command=['echo'], args=['Hello'])
```

To create a Container Components, use the `dsl.container_component` decorator and create a function that returns a `dsl.ContainerSpec` object. `dsl.ContainerSpec` accepts three arguments: `image`, `command`, and `args`. The component above runs the command `echo` with the argument `Hello`.

Container components can be used in pipelines just like Python components:

```python
from kfp import dsl
from kfp import compiler

@dsl.pipeline
def hello_pipeline():
    say_hello()

compiler.Compiler().compile(addition_pipeline, 'pipeline.yaml')
```

If you run this pipeline, you'll see the string `Hello` in `say_hello`'s task execution logs.

### Using component inputs
In a more realistic example, `say_hello` will accept arguments. You can modify `say_hello` so that it accepts an input argument `person_to_greet`:

```python
from kfp import dsl

@dsl.container_component
def say_hello(name: str):
    return dsl.ContainerSpec(image='alpine', command=['echo'], args=[f'Hello, {name}!'])
```

The parameters and annotations in the Container Component function declare the component's interface. In this case, the component has one input parameter `name` and no output parameters.

When you compile this component, `name` will be replaced with a placeholder. At runtime, this placeholder is replaced with the actual value for `name` provided to the `say_hello` component.

Another way to implement this component is to use `sh -c` to read the commands from a single string and pass the name as an argument. This approach tends to be more flexible.

```python
from kfp import dsl

@dsl.container_component
def say_hello(name: str):
    return dsl.ContainerSpec(image='alpine', command=['sh', '-c', 'echo Hello, $0!'], args=[name])
```

When you run the component with the argument `name='World'`, you'll see the `'Hello, World!'` in the logs.

### Creating component outputs
 
Unlike Python functions, containers do not have a standard mechanism for returning values. To enable Container Components to have outputs, KFP provides a mechanism for declaring component outputs by writing to a file in the container.

To return an output string from the say `say_hello` component, you can add an "output" paramter to the function with a `dsl.OutputPath(str)` annotation:

```python
@dsl.container_component
def say_hello(name: str, greeting: dsl.OutputPath(str)):
    ...
```

This component now has one input parameter named `name` typed `str` and one output parameter named `greeting` also typed `str`. The `greeting` annotation is wrapped in `dsl.OutputPath` because it is a "request" to the KFP BE to provide the **path** to which the component code should write the **output** `greeting`. When your component runs, the backend will auto-generate a storage path inside the container and provide it to your component. This file is copied from inside the container to remote storage. **Note:** You will never provide explicitly provide output arguments to components when constructing your pipeline.

### Using in a pipeline

Here's the updated `say_hello` component with the commands and args filled it, and used in a pipeline:

```python
from kfp import dsl
from kfp import compiler

@dsl.container_component
def say_hello(name: str, greeting: dsl.OutputPath(str)):
    """Log a greeting and return it as an output."""

    return dsl.ContainerSpec(image='alpine',
                             command=['sh',
                                      '-c',
                                      """RESPONSE="Hello, $0!"
                                      && echo $RESPONSE
                                      && cat $RESPONSE > $1"""],
                             args=[name, greeting])

@dsl.pipeline
def hello_pipeline(person_to_greet: str) -> str:
    # greeting argument is never provided explicitly!
    hello_task = say_hello(name=person_to_greet)
    return hello_task.outputs['greeting']

compiler.Compiler().compile(addition_pipeline, 'pipeline.yaml')
```

This should look very similar to the [Hello World pipeline][hello-world-pipeline], but with one key difference: since `greeting` is a named output parameter, we access it and return it from the pipeline using `hello_task.outputs['greeting']`, instead of `hello_task.output`. Named outputs and data passing is discussed in more detail in [Pipelines][pipelines].

[hello-world-pipeline]: TODO
[pipelines]: TODO