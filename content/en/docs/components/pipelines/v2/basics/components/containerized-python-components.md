+++
title = "Containerized Python Components"
description = "Create a component from a "
weight = 3
+++

The following assumes a basic familiarity with [Lightweight Python Components](lightweight-python-components).

Containerized Python Components extend [Lightweight Python Components](lightweight-python-components) by relaxing the constraint that Lightweight Python Components be hermetic. Unlike for Lightweight Python Component functions, Containerized Python Component functions can depend on symbols defined outside of the function's definition, imports from outside of the function definition, code in adjacent Python modules, etc. To achieve this, the KFP SDK provides a convenient way to package your Python code into a container.

To demonstrate Containerized Python Components, we'll modify our `add` component from [Lightweight Python Components](lightweight-python-components) example:

```python
from kfp import dsl

@dsl.component
def add(a: int, b: int) -> int:
    return a + b
```

## 1. Source code setup
We'll start by creating an empty `src/` directory to contain all of our code:

```txt
src/
```

Next, we'll add the following simple module `src/math.py` with one helper function:

```python
# src/math.py
def add_numbers(num1, num2):
    return num1 + num2
```

Lastly, we'll move our component to `src/my_component.py` and modify it to use the helper function:

```python
# src/my_component.py
from kfp import dsl
from .math import add_numbers

@dsl.component
def add(a: int, b: int) -> int:
    return add_numbers(num1, num2)
```

`src` now looks like this:

```txt
src/
├── my_component.py
└── math.py
```

## 2. Modify the dsl.component decorator

In the next step, we're going to build the contents of `src/` into a container image. To do this, we need to make one change to the `dsl.component` decorator usage in `src/my_component.py` to indicate the name of the image to build via the `target_image` argument:

```python
@dsl.component(
    target_image='gcr.io/my-project/my-component:v1',
)
```

## 3. Build the Containerized Python Component
Now that your code is in a standalone directory and you've specified a target image, you can conveniently build an image using the [`kfp component build`][kfp-component-build] CLI command:

```sh
kfp component build src/ --component-filepattern my_component.py
```


[kfp-component-build]: https://kubeflow-pipelines.readthedocs.io/en/master/source/cli.html#kfp-component-build