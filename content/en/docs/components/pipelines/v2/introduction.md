+++
title = "Introduction"
description = "What is Kubeflow Pipelines?"
weight = 1
                    
+++


Kubeflow Pipelines (KFP) is a platform for building and deploying portable and scalable machine learning (ML) workflows by using Docker containers.

With KFP you can [author pipelines][author-a-pipeline] using the KFP Python SDK's domain-specific language (DSL), compile the pipeline to an [intermediate representation YAML][ir-yaml], and submit the pipeline to run on a KFP-conformant backend such as the [open source KFP backend][oss-be] or [Google Cloud Vertex AI Pipelines](https://cloud.google.com/vertex-ai/docs/pipelines/introduction).

The [open source KFP backend][oss-be] is available as a core component of Kubeflow or as a standalone installation. To quickly get started with a KFP deployment and usage example, see the [Quickstart][quickstart] guide.

<!-- TODO: Include these links once the topic is available -->
<!-- [Learn more about installing Kubeflow][Installation]
[Learn more about installing Kubeflow Pipelines standalone][Installation] -->

## Why Kubeflow Pipelines?
KFP enables data scientists and machine learning engineers to:
* Author end-to-end ML workflows natively in Python
* Create fully custom ML components or leverage an ecosystem of existing components
* Easily manage, track, and visualize pipeline definitions, runs, experiments, and ML artifacts
* Efficiently use compute resources by eliminating redundant executions through [caching][caching] and parallel task execution out-of-the-box
* Maintain cross-platform pipeline portability through a platform-neutral [IR YAML pipeline definition][ir-yaml]

## What is a pipeline?

A [_pipeline_][pipelines] is the definition of a workflow with one or more steps called [_tasks_][tasks]. A task is an instantiation of a component with input parameters. At runtime, a task corresponds to a single container execution.

KFP provides a seamless way to author components that create output parameters and artifacts. Passing parameters and artifacts between tasks defines the pipeline's execution graph (directed acyclic graph).

## Next steps

* Follow the 
  [pipelines quickstart guide][Quickstart] guide to 
  deploy Kubeflow Pipelines and run your first pipeline
<!-- TODO: Uncomment these links once the topic is created -->
<!-- * Learn more about the [different ways to install KFP][installation] -->
* Learn more about [authoring pipelines][author-a-pipeline]

[quickstart]: /docs/components/pipelines/v2/quickstart
[author-a-pipeline]: /docs/components/pipelines/v2/author-a-pipeline
[components]: /docs/components/pipelines/v2/author-a-pipeline/components
[pipelines]: /docs/components/pipelines/v2/author-a-pipeline/pipelines
[tasks]: /docs/components/pipelines/v2/author-a-pipeline/tasks
[compile-a-pipeline]: /docs/components/pipelines/v2/compile-a-pipeline
[installation]: /docs/components/pipelines/v2/installation
[caching]: /docs/components/pipelines/v2/author-a-pipeline/tasks/#caching
[ir-yaml]: /docs/components/pipelines/v2/compile-a-pipeline/#ir-yaml
[oss-be]: http://localhost:1313/docs/components/pipelines/v2/installation/