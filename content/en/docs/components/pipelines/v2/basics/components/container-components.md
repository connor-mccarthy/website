+++
title = "Container Components"
description = "Create a component from an arbitrary container"
weight = 4
+++

In KFP, each task execution corresponds to a container execution. This means that all components, even Python Components, are defined by an `image`, `command`, and `args`.

Python Components are unique because they abstract the definition of the container away from the user, making it convenient to construct components that use pure Python. Under the hood, the KFP SDK sets the `image`, `commands`, and `args` to the values needed to execute the Python component for the user.

By comparison, Container Components enable the component authors to set the `image`, `command`, and `args` directly. This makes it possible to author components that execute shell scripts, use other languages and binaries, etc., all from within the KFP Python SDK.

Let's start with a simple `say_hello` component and gradually make it more complex until it is equivalent to our `say_hello` component from the [Hello World][hello-world-pipeline]:

```python
from kfp import dsl

@dsl.container_component
def say_hello():
    return dsl.ContainerSpec(image='alpine', command=['echo'], args=['Hello'])
```

Container Components always use the `dsl.container_component` decorator and return a `dsl.ContainerSpec` object. `dsl.ContainerSpec` accepts three arguments: `image`, `command`, and `args`.

The component above runs the command `echo` with the argument `Hello`. We'll see the string `Hello` in the logs.

Container components can be used inside of pipelines just like Python components:

```python
from kfp import dsl
from kfp import compiler

@dsl.pipeline
def hello_pipeline():
    say_hello()

compiler.Compiler().compile(addition_pipeline, 'pipeline.yaml')
```

### Container Component inputs
More realistically, we want to say hello to someone in particular. Let's modify `say_hello` so that it accepts an input argument `person_to_greet`.

The `echo` command accepts only one argument (a single string), so there are two ways to do this:

The first option is to use an f-string to combine the `'Hello'` and the name into a single string:

```python
from kfp import dsl

@dsl.container_component
def say_hello(name: str):
    return dsl.ContainerSpec(image='alpine', command=['echo'], args=[f'Hello, {name}!'])
```

The second option is to tell the shell to read the commands from a string and pass the name as an argument:

```python
from kfp import dsl

@dsl.container_component
def say_hello(name: str):
    return dsl.ContainerSpec(image='alpine', command=['sh', '-c', 'echo Hello, $0!'], args=[name])
```

We'll continue with the second option, since it's more general.

Include parameters in your Container Component function declares the components interface. In this case, the component has one input parameter `name` and no output parameters.

When you run the component with the argument `name='World'`, you'll see the `'Hello, World!'` in the logs.

### Container Component outputs
 
Unlike Python functions, containers do not have a return value. To return the string `'Hello, World!'` as an output of the component, we can write it to a file that KFP will treat as the output file for `say_hello`. We can ask for the path to this file using a `dsl.OutputPath` annotation of type `str`:

```python
from kfp import dsl

@dsl.container_component
def say_hello(name: str, greeting: dsl.OutputPath(str)):
    return dsl.ContainerSpec(image='alpine', command=['sh', '-c',
    """RESPONSE="Hello, $0!"
    && echo $RESPONSE
    && cat $RESPONSE > $1""",],
    args=[name, greeting])
```

When you use a `dsl.OutputPath` annotation, your component will be provided with a string path to which you should write the output parameter as JSON. Note that this string path is independent from the type argument provided to `dsl.OutputPath`; a parameter annotated `dsl.OutputPath(int)` will still be provided a string path argument, to which you should create an output of type `int`.

The component above usage results in a string output named `greeting`.

### Using in a pipeline

Now that our `say_hello` Container Component has an input and an output, we can pass pipeline inputs to the component and return the component's output as a pipeline output, similar to how we did in [Hello World][hello-world-pipeline] pipeline:

```python
from kfp import dsl
from kfp import compiler

@dsl.pipeline
def hello_pipeline(person_to_greet: str) -> str:
    hello_task = say_hello(name=person_to_greet)
    return hello_task.outputs['greeting']

compiler.Compiler().compile(addition_pipeline, 'pipeline.yaml')
```

There are two main differences from the original [Hello World][hello-world-pipeline] pipeline. First, we don't need to pass an input to `say_hello`'s `greeting` parameter. We only need to pass inputs to input parameters and `greeting` is an output parameter. Second, `hello_task`'s output is a named output, we will access it via `hello_task.outputs['greeting']`, unlike `hello_task.output` in the previous example.

[hello-world-pipeline]: