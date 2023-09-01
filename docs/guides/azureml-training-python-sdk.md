---
comments: true
description: Tutorial to train YOLOv8 with Azure Machine Learning
keywords: Ultralytics, YOLO, Deep Learning, Object detection, Tutorial, AzureML
---

This guide explains how to train YOLOv8 with AzureML.

Prerequisites: An [AzureML workspace](https://learn.microsoft.com/azure/machine-learning/concept-workspace?view=azureml-api-2)

## Step 1: Create a CPU compute instance

From your AzureML workspace, on the left tab select Compute > New

Create a new CPU compute instance. You just need a compute instance with light resources because its purpose is solely to
create AzureML resources and initiate training on a larger instance.
Here we select a simple `Standard_DS11_v2`.

![create-azureml-compute](https://github.com/ouphi/ultralytics/assets/17216799/36bd6ffc-fb69-4b02-9d53-cbfe4ee06cab)

Start your compute instance, then on the left tab select Notebooks and create a new  File of type Notebook.

Select your Notebook, now select your corresponding compute and Kernel Python SDK 2.

### Step 2: MLClient set up

Let's configure your MLClient with your subscription-id, your resource group and your azureml workspace.

```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

# Login to configure your workspace and resource group.
credential = DefaultAzureCredential()

ml_client = MLClient(
    credential=credential,
    subscription_id="<your-subscription-id>",
    resource_group_name="<your-resource-group-name>",
    workspace_name="<your-azureml-workspace-name>",
)
```

## Step 3: Create an AzureML environment

### Create Dockerfile
Let's create the Dockerfile containing the dependencies required to run a YOLOv8 training.
Note that in this Dockerfile we install `ultralytics==8.0.166` but you can use the version of your choice.

```
%%bash
mkdir azureml-environment
echo """
FROM pytorch/pytorch:2.0.0-cuda11.7-cudnn8-runtime

# Downloads to user config dir
ADD https://ultralytics.com/assets/Arial.ttf https://ultralytics.com/assets/Arial.Unicode.ttf /root/.config/Ultralytics/

# Install linux packages
ENV DEBIAN_FRONTEND noninteractive
RUN apt update
RUN TZ=Etc/UTC apt install -y tzdata
RUN apt install --no-install-recommends -y gcc git zip curl htop libgl1-mesa-glx libglib2.0-0 libpython3-dev gnupg g++

# Security updates
# https://security.snyk.io/vuln/SNYK-UBUNTU1804-OPENSSL-3314796
RUN apt upgrade --no-install-recommends -y openssl tar

RUN pip install ultralytics==8.0.166

# AzureML leverage mlflow
RUN pip install azureml-mlflow==1.52.0
RUN pip install mlflow==2.4.2
""" > azureml-environment/Dockerfile
```

### Create AzureML environment
Now let's create your AzureML environment:

```python
from azure.ai.ml.entities import Environment, BuildContext

env_docker_context = Environment(
    build=BuildContext(path="azureml-environment"),
    name="yolov8-environment",
    description="Environment to run yolov8 training.",
)
ml_client.environments.create_or_update(env_docker_context)
```

Now, in Environments > Custom environments you should see your environment:

![yolov8-environment-arrow](https://github.com/ouphi/ultralytics/assets/17216799/cc41dbe4-7a43-45e9-aa0d-b53d11d4a92d)


## Step 4: Create an AzureML dataset

You can create an AzureML dataset from the AzureML UI, with the python SDK or with the az cli.
For this example we will use the coco128 dataset.

Download the dataset in the compute instance:

```
%%bash
wget https://ultralytics.com/assets/coco128.zip
unzip coco128.zip
```

Now let's create an AzureML dataset. It will contain the images in the coco128 folder: 

```
from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

coco_data = Data(
    path="coco128",
    type=AssetTypes.URI_FOLDER,
    description="Coco 128 dataset",
    name="coco128"
)

ml_client.data.create_or_update(coco_data)
```

Now in Data > Data assets you should see your dataset.

## Step 5: Create a compute cluster instance

You need to create a compute cluster to run your training. 
Here we are using a Standard_NC6s_v3 with one V100 GPU but you need to adapt the spec of the compute cluster depending 
of your quota and the resources required by your training.
Note that we are creating the cluster with the Python SDK, but you can also create your cluster from the AzureML UI in 
Compute > Compute clusters.

```python
from azure.ai.ml.entities import AmlCompute

cluster = AmlCompute(
    name="cluster-with-1-v100-gpu",
    type="amlcompute",
    size="Standard_NC6s_v3",
    location="westeurope",
    min_instances=0,
    max_instances=2,
    idle_time_before_scale_down=120,
)
ml_client.begin_create_or_update(cluster).result()
```

## Step 6: Prepare the training folder

Now let's create a folder called yolov8-training. It will contain all the files required to run the training:

```
%%bash
mkdir yolov8-training
```

We need a dataset yaml definition. Let's download coco128.yaml for this example and rename it custom-coco128.yaml

```
%%bash
wget https://raw.githubusercontent.com/ultralytics/ultralytics/main/ultralytics/cfg/datasets/coco128.yaml -O yolov8-training/custom-coco128.yaml
```

We want to make sure that the AzureML job uses our custom AzureML dataset, rather than downloading it before the training, so let’s remove the last line of the custom-coco128.yaml

```bash
%%bash
sed -i "s|download:.*$||" yolov8-training/custom-coco128.yaml
```

## Step 7: Run the AzureML job

Let's define the AzureML job.

```bash
from azure.ai.ml import command
from azure.ai.ml import Input

training_job = command(
    inputs=dict(
        training_data=Input(
            type="uri_folder",
            path="azureml:coco128:1",
        )
    ),
    code="yolov8-training",
    command="""
        sed -i "s|path:.*$|path: ${{ inputs.training_data }}|" custom-coco128.yaml &&
        yolo train data=custom-coco128.yaml model=yolov8n.pt epochs=3
    """,
    environment="azureml:yolov8-environment:1",
    compute="cluster-with-1-v100-gpu",
    display_name="yolov8-experiment",
    experiment_name="yolov8-experiment"
)
```

Note that the sed command replaces the path by the dynamic path of the input dataset.

Now let's run the job:

```
%%bash
ml_client.create_or_update(training_job)
```

You should be able to see your AzureML job.
You should be able to see the Params and access the metrics logged with MLFlow

<img width="1504" alt="yolov8-exp" src="https://github.com/ouphi/ultralytics/assets/17216799/074d2777-e6fe-44ef-9b4a-2291b9ca27a5">

In the `Outputs + logs` tab you can find the resulting best.pt and last.pt in `yolov8-experiment/artifacts/train/weights`.
You can find other artifacts in `yolov8-experiment/artifacts/train`.
