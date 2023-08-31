# YOLOv5 🚀 on AzureML Notebook

This guide provides a quickstart to use YOLOv5 from an AzureML Notebook.

## Prerequisites 
Prerequisites: An [AzureML workspace](https://learn.microsoft.com/azure/machine-learning/concept-workspace?view=azureml-api-2).

## Create a compute instance

## Run YOLOv5 on a Notebook

Add the following cells to the Notebook and run them:

Clone YOLOv5 repository with its submodules:

```
%%bash
git clone --recurse-submodules https://github.com/ultralytics/yolov5
```

Install required dependencies:

```
%%bash
pip install -r yolov5/requirements.txt
pip install onnx>=1.10.0
```

Train the YOLOv5 model:

```
%%bash
cd yolov5
python train.py
```

Validate the model for Precision, Recall, and mAP

```
%%bash
cd yolov5
python val.py --weights yolov5s.pt
```

Run inference on images and videos:

```
%%bash
cd yolov5
python detect.py --weights yolov5s.pt --source path/to/images
```

Export models to other formats:

```
%%bash
cd yolov5
python detect.py --weights yolov5s.pt --source path/to/images
```