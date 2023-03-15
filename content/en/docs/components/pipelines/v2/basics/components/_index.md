+++
title = "Components"
description = "Author KFP components"
weight = 2
+++

The component is the atomic building block of KFP pipelines. A component is a remote function definition; it specifies inputs, has user-defined logic, and can create outputs. In this sense, a component is a template. When the component template is provided with input parameters, we call it a task.

There are two types of components in KFP:
1. Python Components
2. Container Component

There are also two types of Python Components: Lighweight Python Components and Containerized Python Components. We'll take a look at lightweight Python components first.