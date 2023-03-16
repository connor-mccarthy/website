+++
title = "Components"
description = "Author KFP components"
weight = 2
+++

The component is the atomic building block of KFP pipelines. A component is a remote function definition; it specifies inputs, has user-defined logic, and can create outputs. When the component template is provided with input parameters, we call it a task.

The types of components you can author in KFP are the following:
1. Python Component  
   * Lightweight Python Component  
   * Containerized Python Component
2. Container Component

Python Components are a convenient way to author components implemented in pure Python. There are two types: lightweight and containerized.

Container Components are a most flexible, advanced approach for authoring components and typically only necessary for components that do not use pure Python.
