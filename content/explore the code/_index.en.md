+++
title = "Explore the code"
weight = 4
chapter = true
pre = "<b>4. </b>"
+++

In addition to the core2aws environment which is the base part of the project, we have added some new components and included some code to support TensorFlow lite micro on the Edukit.  

## Tensorflow Micro Component

In the project expoer, open the components directory.  You will see the tflite folder.  This contains the code for Tensorflow Lite for the ESP32 platform.

The tflite code is integrated to the ESP32 main method by creating a C task and main method as shown here.

```
...
#include "tflite_main.h"
...
void app_main()
{
    ...
    xTaskCreatePinnedToCore(&app_main_tflite, "tflite_main_task", 1024 * 32, NULL, 8, NULL, 1);
    ...
}
```
The tflite_main.cc code is actually written in C++ so this section allows it tobe called as if it were a native C function
```
...
extern "C" void app_main_tflite(void) {
  xEventGroupWaitBits(wifi_event_group, CONNECTED_BIT, false, true, portMAX_DELAY);
  setup();
  while (true) {
    loop();
  }
}
```
And thats it, you can now run tasks that call the Tensorflow libraries.

The main task calls the main_functions.c which is located in the tflite folder.  We will dive deep on the main_functions.c in the next section.

## Keywork Detection Model and code

I the project explorer, open the includes directory.  In here ou will see the supporting code fromt eh Tensorflow project that allows the Edukit enable the microphone, write microphone data to a ringbuffer, read the buffer and convert the audio to a spectogram, run inferences using the Tensroflow model, confirm if a command was detected and respond to that command.

## Tying it all together

Now that you have seen the code layout in the project, you can tie it all together by running the tflite task in the main.c for the application.