+++
title = "Lightweight Python Components"
description = "Create a component from a self-contained Python function"
weight = 1
+++

The easiest way to get started authoring a component is by creating a Lightweight Python Component. We saw an example of the `say_hello` Python components in the [Hello World][hello-world-pipeline] pipeline. Let's take a look at another Lightweight Python Component that adds two numbers together:

```python
from kfp import dsl

@dsl.component
def add(a: int, b: int) -> int:
    return a + b
```

To create a lightweight Python components, simply define a Python function and decorate it with the `@dsl.component` decorator. There are a few requirements for this function:

1. The function must be hermetic.

    This means the function does not reference any symbols defined outside of the function body.

    For example, if I wish to use a constant, the constant must be defined inside the function:

    ```python
    @dsl.component
    def double(a: int) -> int:
        """Succeeds at runtime."""
        VALID_CONSTANT = 2
        return VALID_CONSTANT * a
    ```

    By comparison, the following is invalid and will fail at runtime:
    ```python
    INVALID_CONSTANT = 2
    
    @dsl.component
    def multiply(a: int) -> int:
        """Fails at runtime."""
        return INVALID_CONSTANT * a
    ```
    
    Imports must also be included in the function body:

    ```python
    @dsl.component
    
    def print_env():
        import os
        print(os.environ)
    ```


    This is, of course, a fairly constraining requirement, so we'll drop this constraint later when we look at the more flexible Containerized Python Components.

2. The function inputs and outputs must have type annotations.

    There are two high-level types of inputs and outputs in KFP: parameters and artifacts. The type of input/output will determine the annotation. The above examples use `int`, which is one of the valid parameter annotation types: `int`, `float`, `str`, `bool`, `typing.Dict`, and `typing.List`. We'll discuss artifacts and the associated type annotations later.

3. The function must be decorated with the `@dsl.component` decorator.
    
    This decorator transforms a Python function into a component that can be used within a pipeline.

    For a comprehensive list of `@dsl.component` decorator arguments, see the [DSL reference documentation][dsl-reference-documentation].


[data-passing]: /docs/components/pipelines/v2/author-a-pipeline/component-io
[dsl-reference-documentation]: https://kubeflow-pipelines.readthedocs.io/en/master/source/dsl.html
[python-docker-image]: https://hub.docker.com/_/python
[hello-world-pipeline]:
[type-annotations]: