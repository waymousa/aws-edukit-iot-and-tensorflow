+++
title = "Customize the code"
chapter = true
weight = 6
pre = "<b>6. </b>"
+++

In this section we will chaneg the command responder code so that it causes the Edukit LED's to light up.  Using the microcontroller in this way is called actuation.  You can use the same approach to actuate any response the EduKit can handle, such as posting an MQTT message, activating a motor or changing the operating state of machinery.

Files we will be changing are:
* main\includes\respond.h - header file for the new respond code to manage the leds
* main\respond.c - the code for managing the led state
* main\tflite\command_responder.c - code where we actuate the mirocontroller leds and LCD screen

1. Create main\includes\respond.h

This is the header file for the respond.c file that we will use to control the LED.  In the main folder, create a new file called respond.h and put the following code in it:

```
/*
 * AWS IoT EduKit - Core2 for AWS IoT EduKit
 * Cloud Connected Blinky v1.3.0
 * blink.h
 * 
 * Copyright (C) 2020 Amazon.com, Inc. or its affiliates.  All Rights Reserved.
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy of
 * this software and associated documentation files (the "Software"), to deal in
 * the Software without restriction, including without limitation the rights to
 * use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
 * the Software, and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in all
 * copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
 * FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
 * COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
 * IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
 * CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
 */

void respond(char *response);
```
2. Create main\respond.c

This is the header file for the respond.c file that we will use to control the LED.  In the main folder, create a new file called respond.c and put the following code in it:

```
/*
 * AWS IoT EduKit - Core2 for AWS IoT EduKit
 * Cloud Connected Blinky v1.3.0
 * blink.c
 * 
 * Copyright (C) 2020 Amazon.com, Inc. or its affiliates.  All Rights Reserved.
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy of
 * this software and associated documentation files (the "Software"), to deal in
 * the Software without restriction, including without limitation the rights to
 * use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
 * the Software, and to permit persons to whom the Software is furnished to do so,
 * subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in all
 * copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
 * FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
 * COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
 * IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
 * CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
 */

#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"
#include "core2forAWS.h"
#include "respond.h"

static const char *TAG = "Respond";

void respond(char *response){

    const char* yes = "y";
    const char* no = "n";
    const char* unknown = "u";

    Core2ForAWS_Sk6812_Clear();
    Core2ForAWS_Sk6812_Show();
    if (response == yes){
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_LEFT, 0x00ff00);
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_RIGHT, 0x000000);
        Core2ForAWS_Sk6812_Show();
    } else if (response == no){
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_LEFT, 0x000000);
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_RIGHT, 0xff0000);
        Core2ForAWS_Sk6812_Show();
    } else if (response == unknown){
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_LEFT, 0xffffff);
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_RIGHT, 0xffffff);
        Core2ForAWS_Sk6812_Show();
    } else {        
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_LEFT, 0x000000);
        Core2ForAWS_Sk6812_SetSideColor(SK6812_SIDE_RIGHT, 0x000000);
        Core2ForAWS_Sk6812_Show();
    }
}
```
In summary, this code looks at the incoming response and sets the LED status based on the value of that response.  On the EduKit we have LED on the left and right hand side of the device that can be set to different colours.  In our case we will make the following actuations:
* if the keyword is unknown, light left and right leds white
* if the keywork is yes, light the left led in green and the right led as off
* if the keyword is no, light the right led as red and the left led as off
* for all other keywords, set both leds to off

3. Update main\tflite\command_responder.c

Last, we will update the command_responder.c file so that it actuates the microcontroller features when it detects a keyword it is interested in.

```
/* Copyright 2019 The TensorFlow Authors. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
==============================================================================*/

//#include <string.h>

#include "command_responder.h"

extern "C" {
  #include "ui.h"
}

extern "C" {
    #include "respond.h"
}

// The default implementation writes out the name of the recognized command
// to the error console. Real applications will want to take some custom
// action instead, and should implement their own versions of this function.
void RespondToCommand(tflite::ErrorReporter* error_reporter,
                      int32_t current_time, const char* found_command,
                      uint8_t score, bool is_new_command) {  

  const char* yes = "yes";
  const char* no = "no";
  const char* unknown = "unknown";

  if (is_new_command) {
    TF_LITE_REPORT_ERROR(error_reporter, "Heard %s (%d) @%dms", found_command,
                         score, current_time);
    if (found_command == yes){
      ui_textarea_add("Heard yes.\n", NULL, 0);
      respond("y");
    } else if (found_command == no){
      ui_textarea_add("Heard no.\n", NULL, 0);
      respond("n");
    } else if (found_command == unknown){
      ui_textarea_add("Heard unknown.\n", NULL, 0);
      respond("u");
    } else {
      ui_textarea_add("Heard silence.\n", NULL, 0);
    }    
  }
}
```
Diving deeper on this code, first we include the respond.h and ui.h as external C applications.  The command_responder.cc id written in C++ so we must use the extern keyword to let the compiler know that the methods calls are to a C program.

Next, we create some new variables to represent the states we are interested in, specifically "yes", "no" and "unknown".

Last, we process the command we recieved and based ont eh found_command variable value we call the ui.c and respond.c methods to update the LCD screen and led state respectively.

4. Click the PlatformIO logo on the VS Code activity bar (left most menu).
5. From the Quick Access menu, under Miscellaneous, select New Terminal. The terminal viewport should load with a new terminal labeled PlatformIO CLI.
6. Run the following command to build the firmware for your device.
```
pio run --environment core2foraws
```
7. With the build sucessful, run the following command to upload the copmpiled code to your device over USB.
```
pio run --environment core2foraws --target upload
```
After the device reboots you should see the display light up with a message saying Tensorflow for a few minutes.
You should now see the LCD screen output showing a continous stream of what it is hearing.  When the model detects a yes or no the leds should light up!  Congratulations!