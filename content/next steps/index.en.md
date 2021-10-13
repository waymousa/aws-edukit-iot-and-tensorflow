---
title : "Next Steps"
weight : 99
---

# Next Steps
Now that you have had some hands on experience with deploying and running models on the edge you can refine the project to do additional things.

## Improve model performance
The model has been created with a very basic data set so it can't know for certain what it heard all the time because it doesn't know your voice.  To improve the performance of the model you need to train your model with new data so it recognises you.

## Update model over the air

As your model gets better over time you can create a pipeline to update the model and deploy the code using the OTA feature.

## Deploy a cascade architecture

Now that you have a model that is updating at the edge, you can start summarising inference data and sending it to the cloud for further processing.  MCU devices can do some inference, but for more advanced utterances the data needs to be sent to cloud for more powerful processing.  You can use MQTT to help you do this.

## Resources and further reading
Here are links to additional resources you can use.