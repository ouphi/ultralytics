---
comments: true
description: Tutorial to train YOLOv8 with Azure Machine Learning
keywords: Ultralytics, YOLO, Deep Learning, Object detection, Tutorial, AzureML
---

This guide will help new users train YOLOv8 on AzureML.

Prerequisites: An [AzureML workspace](https://learn.microsoft.com/en-us/azure/machine-learning/concept-workspace?view=azureml-api-2)

## 1. Create a CPU compute instance

From you AzureML workspace, on the left tab select Compute > New

Create a new CPU compute instance. You just need a compute instance with light resources as its purpose is solely to
create AzureML resources and initiate training on a larger instance.


