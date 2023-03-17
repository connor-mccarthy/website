+++
title = "Components"
description = "Author KFP components"
weight = 4
+++

Components are the building blocks of KFP pipelines. A component is a remote function definition; it specifies inputs, has user-defined logic in its body, and can create outputs. When the component template is provided with input parameters, we call it a task.

The types of components you can author in KFP are the following:
1. Python Component  
   * Lightweight Python Component  
   * Containerized Python Component
2. Container Component

Python Components are a convenient way to author components implemented in pure Python. There are two types: Lightweight and Containerized.

Container Components are a more flexible, advanced approach, which allow you to define a component using an arbitrary container definition. This is the recommended approach for components that are not implemented in pure Python.
