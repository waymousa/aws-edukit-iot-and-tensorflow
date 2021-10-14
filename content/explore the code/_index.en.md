+++
title = "Explore the code"
weight = 4
chapter = true
pre = "<b>4. </b>"
+++

In addition to the core2aws environment which is the base part of the project, we have added some new components and included some code to support TensorFlow lite micro on the Edukit.  

## Tensorflow Micro Component

In the project expoer, open the components directory.  You will see the tflite folder.  This comntains the code for Tensorflow Lite for the ESP32 platform.

## Keywork Detection Model and code

I the project explorer, open the includes directory.  In here ou will see the supporting code fromt eh Tensorflow project that allows the Edukit enable the microphone, write microphone data to a ringbuffer, read the buffer and convert the audio to a spectogram, run inferences using the Tensroflow model, confirm if a command was detected and respond to that command.

## Tying it all together

Now that you have seen the code layout in the project, you can tie it all together by running the tflite task in the main.c for the application.