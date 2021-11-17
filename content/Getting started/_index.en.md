+++
title = "Getting started"
chapter = true
weight = 1
pre = "<b>1. </b>"
+++

## Install & Configure your local toolchain

To complete this workshop you will need to use a Windows, Linux or Mac workstation which is configured with the toolchain you will need to deploy code to your Edukit device.  

To set up your workstation visit [AWS IoT EduKit Workshop](https://edukit.workshop.aws/en/getting-started/prerequisites.html) and follow the instructions for your OS.

Specific steps required for this lab are to install:
* Git
* Visual Studio
* PlatformIO
* Silicon Labs USB to UART bridge

## Set up prerequisites

For this workshop we will be focussing on how to run a simple Tensorflow application on our micrcontroller unit its not necessary to integrate the controller with any AWS services for the lab.  The key benefit of TinyML is the ability to run inference at the edge, make decisions and take actions there, interacting with the cloud when needed as part of a wider solution.

When you are considering how to use TinyML as part of a wider application then you can utilise services such as:

* OTA for flashing the microcontroller and updating the model *see the OTA track in the repeat for this builders session* 
* MQTT for sending summary statiasctics and trends to the cloud for further processing
* S3 for uploading data to improve your model, for example when trainign a keyword detection model to understand new accents
* SageMaker Edge Manager for improving your models over time
