+++
title = "Containerized Python Components"
description = "Create a component from a "
weight = 3
+++

The following assumes a basic familiarity with [Lightweight Python Components](lightweight-python-components).

Containerized Python Components extend [Lightweight Python Components](lightweight-python-components) by relaxing the constraint that Lightweight Python Components be hermetic. Unlike for Lightweight Python Component functions, Containerized Python Component functions can depend on symbols defined outside of the function's definition, imports from outside of the function definition, code in adjacent Python modules, etc. To achieve this, the KFP SDK provides a convenient way to package your Python code into a container.

To start using Containerized Python Components, you can modify the `add` component from [Lightweight Python Components](lightweight-python-components) example:

```python
from kfp import dsl

@dsl.component
def add(a: int, b: int) -> int:
    return a + b
```

## 1. Source code setup
Start by creating an empty `src/` directory to contain your component source code:

```txt
src/
```

Next, add the following simple module `src/math.py` with one helper function:

```python
# src/math.py
def add_numbers(num1, num2):
    return num1 + num2
```

Lastly, move our component to `src/my_component.py` and modify it to use the helper function:

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

In this step, you'll set the tag of the component's container image by setting the `target_image` argument on your component in `src/my_component.py`.

Note that `target_image` is distinct from `base_image`. In Lightweight Python Components, `base_image` is the image that will be used when executing your Python function. In a Containerized Python Component, `base_image` specifies the base image that KFP will use when building your new container (i.e., [the Docker `FROM` instruction](https://docs.docker.com/engine/reference/builder/#from)).

By comparison, `target_image` is what KFP will use as the tag for the image you build in the next step. `target_image` should not be used for Lightweight Python Components.

The following example includes `base_image` for clarity, but this is not necessary as `base_image` will to default to `python:3.7` if omitted.

```python
@dsl.component(
    base_image='python:3.7',
    target_image='gcr.io/my-project/my-component:v1',
)
```

## 3. Build the component
Now that your code is in a standalone directory and you've specified a target image, you can conveniently build an image using the [`kfp component build`][kfp-component-build] CLI command:

```sh
kfp component build src/ --component-filepattern my_component.py --no-push-image
```

If you have a [configured Docker to use a private image registry](https://docs.docker.com/engine/reference/commandline/login/), you can replace the `--no-push-image` flag with `--push-image` to automatically push the image after building.

4. Use the component in a pipeline

Finally, you can use the component in an image. Since the above example `target_image` uses [Google Cloud Artifact Registry](https://cloud.google.com/artifact-registry/docs/docker/authentication) (`gcr.io`), following assumes you have pushed your image to Google Cloud Artifact Registry, you are running your pipeline on [Google Cloud Vertex AI Pipelines](https://cloud.google.com/vertex-ai/docs/pipelines/introduction), and you have configured [IAM permissions](https://cloud.google.com/iam) so that Vertex AI Pipelines can pull images from Artifact Registry.

```python
from kfp import dsl

@dsl.pipeline
def math_pipeline(x: int, y: int) -> int:
    task1 = add(a=x, b=y)
    task2 = add(a=task1.output, b=x)
    return task2.output

compiler.Compiler().compile(addition_pipeline, 'pipeline.yaml')
```

[kfp-component-build]: https://kubeflow-pipelines.readthedocs.io/en/master/source/cli.html#kfp-component-build