---
title: 'Architecture & key concepts'
titleSuffix: Azure Machine Learning
description: Learn about the architecture, terms, and concepts that make up Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 08/20/2020
ms.custom: seoapril2019, seodec18
# As a data scientist, I want to understand the big picture about how Azure Machine Learning works.
---

# How Azure Machine Learning works: Architecture and concepts

Learn about the architecture and concepts for [Azure Machine Learning](overview-what-is-azure-ml.md).  This article gives you a high-level understanding of the components and how they work together to assist in the process of building, deploying, and maintaining machine learning models.

## <a name="workspace"></a> Workspace

A [machine learning workspace](concept-workspace.md) is the top-level resource for Azure Machine Learning.

:::image type="content" source="media/concept-azure-machine-learning-architecture/architecture.svg" alt-text="Diagram: Azure Machine Learning architecture of a workspace and its components":::

The workspace is the centralized place to:

* Manage resources you use for training and deployment of models, such as [computes](#compute-instance)
* Store assets you create when you use Azure Machine Learning, including:
  * [Environments](#environments)
  * [Experiments](#experiments)
  * [Pipelines](#ml-pipelines)
  * [Datasets](#datasets-and-datastores)
  * [Models](#models)
  * [Endpoints](#endpoints)

A workspace includes other Azure resources that are used by the workspace:

+ [Azure Container Registry (ACR)](https://azure.microsoft.com/services/container-registry/): Registers docker containers that you use during training and when you deploy a model. To minimize costs, ACR is only created when deployment images are created.
+ [Azure Storage account](https://azure.microsoft.com/services/storage/): Is used as the default datastore for the workspace.  Jupyter notebooks that are used with your Azure Machine Learning compute instances are stored here as well.
+ [Azure Application Insights](https://azure.microsoft.com/services/application-insights/): Stores monitoring information about your models.
+ [Azure Key Vault](https://azure.microsoft.com/services/key-vault/): Stores secrets that are used by compute targets and other sensitive information that's needed by the workspace.

You can share a workspace with others.

## Computes

<a name="compute-targets"></a>
A [compute target](concept-compute-target.md) is any machine or set of machines you use to run your training script or host your service deployment. You can use your local machine or a remote compute resource as a compute target.  With compute targets, you can start training on your local machine and then scale out to the cloud without changing your training script.

Azure Machine Learning introduces two fully managed cloud-based virtual machines (VM) that are configured for machine learning tasks:

* <a name="compute-instance"></a> **Compute instance**: A compute instance is a VM that includes multiple tools and environments installed for machine learning. The primary use of a compute instance is for your development workstation.  You can start running sample notebooks with no setup required. A compute instance can also be used as a compute target for training and inferencing jobs.

* **Compute clusters**: Compute clusters are a cluster of VMs with multi-node scaling capabilities. Compute clusters are better suited for compute targets for large jobs and production.  The cluster scales up automatically when a job is submitted.  Use as a training compute target or for dev/test deployment.

For more information about training compute targets, see [Training compute targets](concept-compute-target.md#train).  For more information about deployment compute targets, see [Deployment targets](concept-compute-target.md#deploy).

## Datasets and datastores

[**Azure Machine Learning Datasets**](concept-data.md#datasets)  make it easier to access and work with your data. By creating a dataset, you create a reference to the data source location along with a copy of its metadata. Because the data remains in its existing location, you incur no extra storage cost, and don't risk the integrity of your data sources.

For more information, see [Create and register Azure Machine Learning Datasets](how-to-create-register-datasets.md).  For more examples using Datasets, see the [sample notebooks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/datasets-tutorial).

Datasets use [datastores](concept-data.md#datastores) to securely connect to your Azure storage services. Datastores store connection information without putting your authentication credentials and the integrity of your original data source at risk. They store connection information, like your subscription ID and token authorization in your Key Vault associated with the workspace, so you can securely access your storage without having to hard code them in your script.

## Environments

[Workspace](#workspace) > **Environments**

An [environment](concept-environments.md) is the encapsulation of the environment where training or scoring of your machine learning model happens. The environment specifies the Python packages, environment variables, and software settings around your training and scoring scripts.  

For code samples, see the "Manage environments" section of [How to use environments](how-to-use-environments.md#manage-environments).

## Experiments

[Workspace](#workspace) > **Experiments**

An experiment is a grouping of many runs from a specified script. It always belongs to a workspace. When you submit a run, you provide an experiment name. Information for the run is stored under that experiment. If the name doesn't exist when you submit an experiment, a new experiment is automatically created.
  
For an example of using an experiment, see [Tutorial: Train your first model](tutorial-1st-experiment-sdk-train.md).

### Runs

[Workspace](#workspace) > [Experiments](#experiments) > **Run**

A run is a single execution of a training script. An experiment will typically contain multiple runs.

Azure Machine Learning records all runs and stores the following information in the experiment:

* Metadata about the run (timestamp, duration, and so on)
* Metrics that are logged by your script
* Output files that are autocollected by the experiment or explicitly uploaded by you
* A snapshot of the directory that contains your scripts, prior to the run

You produce a run when you submit a script to train a model. A run can have zero or more child runs. For example, the top-level run might have two child runs, each of which might have its own child run.

### Run configurations

[Workspace](#workspace) > [Experiments](#experiments) > [Run](#runs) > **Run configuration**

A run configuration is a set of instructions that defines how a script should be run in a specified compute target. The configuration includes a wide set of behavior definitions, such as whether to use an existing Python environment or to use a Conda environment that's built from a specification.

A run configuration can be persisted into a file inside the directory that contains your training script.   Or it can be constructed as an in-memory object and used to submit a run.

For example run configurations, see [Select and use a compute target to train your model](how-to-set-up-training-targets.md).

### Estimators

To facilitate model training with popular frameworks, the estimator class allows you to easily construct run configurations. You can create and use a generic [Estimator](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator?view=azure-ml-py) to submit training scripts that use any learning framework you choose (such as scikit-learn).

For more information about estimators, see [Train ML models with estimators](how-to-train-ml-models.md).

### Snapshots

[Workspace](#workspace) > [Experiments](#experiments) > [Run](#runs) > **Snapshot**

When you submit a run, Azure Machine Learning compresses the directory that contains the script as a zip file and sends it to the compute target. The zip file is then extracted, and the script is run there. Azure Machine Learning also stores the zip file as a snapshot as part of the run record. Anyone with access to the workspace can browse a run record and download the snapshot.


### Logging

When you develop your solution, use the Azure Machine Learning Python SDK in your Python script to log arbitrary metrics. After the run, query the metrics to determine whether the run has produced the model you want to deploy.


> [!NOTE]
> [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]

### Git tracking and integration

When you start a training run where the source directory is a local Git repository, information about the repository is stored in the run history. This works with runs submitted using an estimator, ML pipeline, or script run. It also works for runs submitted from the SDK or Machine Learning CLI.

For more information, see [Git integration for Azure Machine Learning](concept-train-model-git-integration.md).

## Models

At its simplest, a model is a piece of code that takes an input and produces output. Creating a machine learning model involves selecting an algorithm, providing it with data, and [tuning hyperparameters](how-to-tune-hyperparameters.md). Training is an iterative process that produces a trained model, which encapsulates what the model learned during the training process.

You can bring a model that was trained outside of Azure Machine Learning. Or you can train a model by submitting a [run](#runs) of an [experiment](#experiments) to a [compute target](#compute-targets) in Azure Machine Learning. Once you have a model, you [register the model](#register-model) in the workspace.

Azure Machine Learning is framework agnostic. When you create a model, you can use any popular machine learning framework, such as Scikit-learn, XGBoost, PyTorch, TensorFlow, and Chainer.

For an example of training a model using Scikit-learn, see [Tutorial: Train an image classification model with Azure Machine Learning](tutorial-train-models-with-aml.md).


### <a name="register-model"></a> Model registry

[Workspace](#workspace) > **Models**

The **model registry** lets you keep track of all the models in your Azure Machine Learning workspace.

Models are identified by name and version. Each time you register a model with the same name as an existing one, the registry assumes that it's a new version. The version is incremented, and the new model is registered under the same name.

When you register the model, you can provide additional metadata tags and then use the tags when you search for models.

> [!TIP]
> A registered model is a logical container for one or more files that make up your model. For example, if you have a model that is stored in multiple files, you can register them as a single model in your Azure Machine Learning workspace. After registration, you can then download or deploy the registered model and receive all the files that were registered.

You can't delete a registered model that is being used by an active deployment.

For an example of registering a model, see [Train an image classification model with Azure Machine Learning](tutorial-train-models-with-aml.md).

## Deployment

You deploy a [registered model](#register-model) as a service endpoint. You need the following components:

* **Environment**. This environment encapsulates the dependencies required to run your model for inference.
* **Scoring code**. This script accepts requests, scores the requests by using the model, and returns the results.
* **Inference configuration**. The inference configuration specifies the environment, entry script, and other components needed to run the model as a service.

For more information about these components, see [Deploy models with Azure Machine Learning](how-to-deploy-and-where.md).

### Endpoints

[Workspace](#workspace) > **Endpoints**

An endpoint is an instantiation of your model into either a web service that can be hosted in the cloud or an IoT module for integrated device deployments.

#### Web service endpoint

When deploying a model as a web service, the endpoint can be deployed on Azure Container Instances, Azure Kubernetes Service, or FPGAs. You create the service from your model, script, and associated files. These are placed into a base container image, which contains the execution environment for the model. The image has a load-balanced, HTTP endpoint that receives scoring requests that are sent to the web service.

You can enable Application Insights telemetry or model telemetry to monitor your web service. The telemetry data is accessible only to you.  It's stored in your Application Insights and storage account instances.

If you've enabled automatic scaling, Azure automatically scales your deployment.

For an example of deploying a model as a web service, see [Deploy an image classification model in Azure Container Instances](tutorial-deploy-models-with-aml.md).

#### IoT module endpoints

A deployed IoT module endpoint is a Docker container that includes your model and associated script or application and any additional dependencies. You deploy these modules by using Azure IoT Edge on edge devices.

If you've enabled monitoring, Azure collects telemetry data from the model inside the Azure IoT Edge module. The telemetry data is accessible only to you, and it's stored in your storage account instance.

Azure IoT Edge ensures that your module is running, and it monitors the device that's hosting it. 
## Automation

### Azure Machine Learning CLI 

The [Azure Machine Learning CLI](reference-azure-machine-learning-cli.md) is an extension to the Azure CLI, a cross-platform command-line interface for the Azure platform. This extension provides commands to automate your machine learning activities.

### ML Pipelines

You use [machine learning pipelines](concept-ml-pipelines.md) to create and manage workflows that stitch together machine learning phases. For example, a pipeline might include data preparation, model training, model deployment, and inference/scoring phases. Each phase can encompass multiple steps, each of which can run unattended in various compute targets. 

Pipeline steps are reusable, and can be run without rerunning the previous steps if the output of those steps hasn't changed. For example, you can retrain a model without rerunning costly data preparation steps if the data hasn't changed. Pipelines also allow data scientists to collaborate while working on separate areas of a machine learning workflow.

## Interacting with your workspace

### Studio

[Azure Machine Learning studio](https://ml.azure.com) provides a web view of all the artifacts in your workspace.  You can view results and details of your datasets, experiments, pipelines, models, and endpoints.  You can also manage compute resources and datastores in the studio.

Studio is also where you access the interactive tools that are part of Azure Machine Learning:

+ [Azure Machine Learning designer (preview)](concept-designer.md) to perform workflow steps without writing code
+ Web experience for [automated machine learning](concept-automated-ml.md)
+ [Data labeling projects](how-to-create-labeling-projects.md) to create, manage, and monitor projects to label your data

### Programming tools

> [!IMPORTANT]
> Tools marked (preview) below are currently in public preview.
> The preview version is provided without a service level agreement, and it's not recommended for production workloads. Certain features might not be supported or might have constrained capabilities. 
> For more information, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

+  Interact with the service in any Python environment with the [Azure Machine Learning SDK for Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py).
+ Interact with the service in any R environment with the [Azure Machine Learning SDK for R](https://azure.github.io/azureml-sdk-for-r/reference/index.html) (preview).
+ Use [Azure Machine Learning CLI](https://docs.microsoft.com/azure/machine-learning/reference-azure-machine-learning-cli) for automation.
+ The [Many Models Solution Accelerator](https://aka.ms/many-models) (preview) builds on Azure Machine Learning and enables you to train, operate, and manage hundreds or even thousands of machine learning models.

## Next steps

To get started with Azure Machine Learning, see:

* [What is Azure Machine Learning?](overview-what-is-azure-ml.md)
* [Create an Azure Machine Learning workspace](how-to-manage-workspace.md)
* [Tutorial (part 1): Train a model](tutorial-train-models-with-aml.md)
