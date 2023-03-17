+++
title = "Lightweight Python Components"
description = "Create a component from a self-contained Python function"
weight = 1
+++

The easiest way to get started authoring components is by creating a Lightweight Python Component. We saw an example of the `say_hello` Lightweight Python Components in the [Hello World][hello-world-pipeline] pipeline. Let's take a look at another Lightweight Python Component that adds two numbers together:

```python
from kfp import dsl

@dsl.component
def add(a: int, b: int) -> int:
    return a + b
```

The `@dsl.component` decorator turns Python functions into components that can be used in a pipeline definition or executed independently as a single step pipeline.

### Python function requirements
There are a few requirements for functions that can be decorated with the `@dsl.component` decorator:


1. **Type annotations:** The function inputs and outputs must have valid KFP type annotations.

    There are two forms of inputs/outputs in KFP: parameters and artifacts. The type affects UI rendering, ML metadata tracking, and how you interact with the input/output inside the component body.

    The type of the input/output is determined by its annotation. In the example above `a` and `b` are both parameters of type `int`. Valid parameter annotations include Python's built-in `int`, `float`, `str`, `bool`, `typing.Dict`, and `typing.List`.
    
    Artifacts are discussed in detail in [Data Types][data-types].


2. **Hermetic:** The Python function may not reference any symbols defined outside of its body.

    For example, if you wish to use a constant, the constant must be defined inside the function:

    ```python
    @dsl.component
    def double(a: int) -> int:
        """Succeeds at runtime."""
        VALID_CONSTANT = 2
        return VALID_CONSTANT * a
    ```

    By comparison, the following is invalid and will fail at runtime:
    ```python
    # non-example!
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


    This is, of course, a fairly constraining requirement, so we'll drop this constraint later when we look at the more flexible [Containerized Python Components][containerized-python-components].

### dsl.component Decorator Arguments
In the above examples we used the [`@dsl.component`][https://kubeflow-pipelines.readthedocs.io/en/master/source/dsl.html#kfp.dsl.component] with only one argument: the Python function.

The decorator accepts some additional arguments, which are described by the [KFP SDK DSL reference docs][dsl-reference-documentation].

#### packages_to_install

Most realistic Lightweight Python Components will depend on other Python libraries. You can pass a list of requirements to `packages_to_install` and the component will install these packages at runtime:

```python
@dsl.component(packages_to_install=['numpy==1.21.6'])
def sin(val: float = 3.14) -> float:
    return np.sin(val).item()
```
#### pip_index_urls

`pip_index_urls` exposes the ability to pip install `packages_to_install` from the non-default package index [PyPI.org](https://pypi.org/) (index URL: https://pypi.org/simple). Specifically, `pip_index_urls` allows you to modify [`pip install`](https://pip.pypa.io/)'s [`--index-url`](https://pip.pypa.io/en/stable/cli/pip_install/#cmdoption-0) and [`--extra-index-url`](https://pip.pypa.io/en/stable/cli/pip_install/#cmdoption-extra-index-url) options.

Take the following component:

```python
@dsl.component(packages_to_install=['custom-ml-package==0.0.1', 'numpy==1.21.6'],
               pip_index_urls=['http://myprivaterepo.com/simple', 'http://pypi.org/simple'],
)
def comp():
    from custom_ml_package import model_trainer
    ...
```

These arguments translate to the following command:

```sh
pip install custom-ml-package==0.0.1 numpy==1.21.6 kfp==2 --index-url http://myprivaterepo.com/simple --trusted-host http://myprivaterepo.com/simple --extra-index-url http://pypi.org/simple --trusted-host http://pypi.org/simple
```

Note that `'http://pypi.org/simple'` is not included in `pip_index_urls` by default. The example above includes this index URL explicitly so that `kfp` and `numpy` can be installed, even though they are not hosted at `http://myprivaterepo.com/simple`.

#### base_image

When you create a Lightweight Python Component, your Python function code is extracted by the KFP SDK and executed inside of a container. By default, the container image used is [`python:3.7`](https://hub.docker.com/_/python). You can override this image with `base_image`. This can be useful if your code requires a specific Python version or dependencies included in the non-default image.

```python
@dsl.component(base_image='python:3.8')
def print_py_version():
    import sys
    print(sys.version)
```

#### install_kfp_package and kfp_package_path
<!-- Describing as the KFP runtime package, because eventually we may convert this to a true runtime package. It's best that users think of it this way, even though it currently installs the full KFP package. -->

`install_kfp_package` and `kfp_package_path` can be used together to provide granular control over whether and from where the `kfp` runtime package is installed at component runtime. **Note:** This is generally not necessary and is discouraged for the majority of use-cases.

By default, the `kfp` runtime package will be installed at component runtime. This is how KFP executes your Python function on your behalf. `install_kfp_package` can be set to `False` to disable this behavior. `kfp_package_path` or `pip_index_urls` can be used to install `kfp` from another location, such as a GitHub branch or a private package repository, respectively.




[data-passing]: /docs/components/pipelines/v2/author-a-pipeline/component-io
[dsl-reference-documentation]: https://kubeflow-pipelines.readthedocs.io/en/master/source/dsl.html
[python-docker-image]: https://hub.docker.com/_/python
[hello-world-pipeline]: TODO
[type-annotations]: TODO
[data-types]: TODO
[containerized-python-components]: TODO