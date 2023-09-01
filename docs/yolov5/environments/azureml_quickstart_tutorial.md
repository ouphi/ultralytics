# YOLOv5 🚀 on AzureML compute instance

This guide provides a quickstart to use YOLOv5 from an AzureML Notebook.

## Prerequisites 
Prerequisites: An [AzureML workspace](https://learn.microsoft.com/azure/machine-learning/concept-workspace?view=azureml-api-2).

## Step 1: Create a compute instance

From your AzureML workspace, select Compute > Compute instances > New, select the compute with the resources you need.

<img width="1741" alt="create-compute-arrow" src="https://github.com/ouphi/ultralytics/assets/17216799/3e92fcc0-a08e-41a4-af81-d289cfe3b8f2">

## Step 2: Open a Terminal

Now from the Notebooks view, open a Terminal and select your compute.

![open-terminal-arrow](https://github.com/ouphi/ultralytics/assets/17216799/c4697143-7234-4a04-89ea-9084ed9c6312)

## Setup and run YOLOv5

Now you can, create a virtual environment:

```bash
conda create --name yolov5env -y
conda activate yolov5env
conda install pip -y
```

Clone YOLOv5 repository with its submodules:

```bash
git clone https://github.com/ultralytics/yolov5
cd yolov5
git submodule update --init --recursive # Note that you might have a message asking you to add your folder as a safe.directory just copy the recommended command   
```

Install the required dependencies:

```bash
pip install -r yolov5/requirements.txt
pip install onnx>=1.10.0
```

Train the YOLOv5 model:

```bash
python train.py
```

Validate the model for Precision, Recall, and mAP

```bash
python val.py --weights yolov5s.pt
```

Run inference on images and videos:

```bash
python detect.py --weights yolov5s.pt --source path/to/images
```

Export models to other formats:

```bash
python detect.py --weights yolov5s.pt --source path/to/images
```

## Use a Notebook

Note that if you want to run these commands from a Notebook, you need to [create a new Kernel](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-access-terminal?view=azureml-api-2#add-new-kernels)
and select your new Kernel on the top of your Notebook.

If you create python cells it will automatically use your custom environment, but if you add bash cells, you will need to run `source activate <your-env>` on each of these cells
to make sure if uses your custom environment.

For example:

```bash
%%bash
source activate newenv
python val.py --weights yolov5s.pt
```

