---
comments: true
description: Azure Machine Learning YOLOv8 quickstart
keywords: Ultralytics, YOLO, Deep Learning, Object detection, quickstart, AzureML
---

/!\ Note that this guide is only for quick trials from a compute terminal or on a compute Notebook.
[This complete guide explains](azureml-training-python-sdk.md) how you can train a model with an AzureML job.

## Step 1: Create a compute instance

From your AzureML workspace, select Compute > Compute instances > New, select the compute with the resources you need.

<img width="1741" alt="create-compute-arrow" src="https://github.com/ouphi/ultralytics/assets/17216799/3e92fcc0-a08e-41a4-af81-d289cfe3b8f2">

## Quickstart from terminal

Start your compute and open a Terminal:

![open-terminal](https://github.com/ouphi/ultralytics/assets/17216799/635152f1-f4a3-4261-b111-d416cb5ef357)


Create your conda virtualenv and install pip in it:

```bash
conda create --name yolov8env -y
conda activate yolov8env
conda install pip -y
```

Install the required dependencies:

```bash
cd ultralytics
pip install -r requirements.txt
pip install ultralytics
pip install onnx>=1.12.0
```

Predict:
```bash
yolo predict model=yolov8n.pt source='https://ultralytics.com/images/bus.jpg'
```

Train a detection model for 10 epochs with an initial learning_rate of 0.01:
```bash
yolo train data=coco128.yaml model=yolov8n.pt epochs=10 lr0=0.01
```

You can find more [instructions to use the Ultralytics cli here](https://docs.ultralytics.com/quickstart/#use-ultralytics-with-cli).

## Quickstart from Notebook
### Create a new IPython kernel

Open the compute Terminal.

![open-terminal](https://github.com/ouphi/ultralytics/assets/17216799/635152f1-f4a3-4261-b111-d416cb5ef357)

From your compute terminal you need to add a new ipykernel that will be used by your notebook to manage your dependencies:

```bash
conda create --name yolov8env -y
conda activate yolov8env
conda install pip -y
conda install ipykernel -y
python -m ipykernel install --user --name yolov8env --display-name "yolov8env"
```

Close your terminal and create a new notebook. From your Notebook, you can select the new kernel.

Then you can open a Notebook cell and install the required dependencies:

```bash
%%bash
source activate yolov8env
cd ultralytics
pip install -r requirements.txt
pip install ultralytics
pip install onnx>=1.12.0
```

Note that we need to use the `source activate yolov8env` for all the %%bash cells, to make sure that the %%bash cell uses environment we want.

Run some predictions using the [Ultralytics CLI](https://docs.ultralytics.com/quickstart/#use-ultralytics-with-cli):

```bash
%%bash
source activate yolov8env
yolo predict model=yolov8n.pt source='https://ultralytics.com/images/bus.jpg'
```

Or with the [Ultralytic Python interface](https://docs.ultralytics.com/quickstart/#use-ultralytics-with-python), for example to train the model:

```python
from ultralytics import YOLO

# Load a model
model = YOLO("yolov8n.yaml")  # build a new model from scratch
model = YOLO("yolov8n.pt")  # load a pretrained model (recommended for training)

# Use the model
model.train(data="coco128.yaml", epochs=3)  # train the model
metrics = model.val()  # evaluate model performance on the validation set
results = model("https://ultralytics.com/images/bus.jpg")  # predict on an image
path = model.export(format="onnx")  # export the model to ONNX format
```
