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

The main task calls the *main_functions.c* which is located in the tflite folder.  We will dive deep on the *main_functions.c* in the next section.

## Main functions

The main_functions.c file contains multiple sections you will need to become familiar with.  For those of you who are familiar with Arduino platform this will look very similar.

Firstly, we include the Tensorflow libraries.

```
#include "main_functions.h"

#include "audio_provider.h"
#include "command_responder.h"
#include "feature_provider.h"
#include "model.h"
#include "recognize_commands.h"
#include "tensorflow/lite/micro/micro_error_reporter.h"
#include "tensorflow/lite/micro/micro_interpreter.h"
#include "tensorflow/lite/micro/micro_mutable_op_resolver.h"
#include "tensorflow/lite/micro/system_setup.h"
#include "tensorflow/lite/schema/schema_generated.h"
```

### Intitialise

#### Declare Variables

We intitialise the code by setting the global namespace and declaring variables.  In the code example below you can see the variables for the ErrorReporter, the model and the MicroIntepreter.  These same variavbles will be used for any TensorFlow Lite Micro applications you might develop.

```
// Globals, used for compatibility with Arduino-style sketches.
namespace {
tflite::ErrorReporter* error_reporter = nullptr;
const tflite::Model* model = nullptr;
tflite::MicroInterpreter* interpreter = nullptr;
TfLiteTensor* model_input = nullptr;
FeatureProvider* feature_provider = nullptr;
RecognizeCommands* recognizer = nullptr;
int32_t previous_time = 0;
```

The next section sets up the TensorArena.  You need to experiment with this a bit to get the memory sizing right because, whilst bigger is better, on a microcontroller you maty be wastign memory that is needed for other things.  The size of your model largely determines this value.

// Create an area of memory to use for input, output, and intermediate arrays.
// The size of this will depend on the model you're using, and may need to be
// determined by experimentation.
constexpr int kTensorArenaSize = 10 * 1024;
uint8_t tensor_arena[kTensorArenaSize];
int8_t feature_buffer[kFeatureElementCount];
int8_t* model_input_buffer = nullptr;
}  // namespace
```

#### Load the model

Next, we setup the environment with the model which we pull in from the model.cc file.  More about that later in this section.

```
// The name of this function is important for Arduino compatibility.
void setup() {
  tflite::InitializeTarget();

  // Set up logging. Google style is to avoid globals or statics because of
  // lifetime uncertainty, but since this has a trivial destructor it's okay.
  // NOLINTNEXTLINE(runtime-global-variables)
  static tflite::MicroErrorReporter micro_error_reporter;
  error_reporter = &micro_error_reporter;

  // Map the model into a usable data structure. This doesn't involve any
  // copying or parsing, it's a very lightweight operation.
  model = tflite::GetModel(g_model);
  if (model->version() != TFLITE_SCHEMA_VERSION) {
    TF_LITE_REPORT_ERROR(error_reporter,
                         "Model provided is schema version %d not equal "
                         "to supported version %d.",
                         model->version(), TFLITE_SCHEMA_VERSION);
    return;
  }
```

#### Resolve Operators

To make sure our model runs effieicntly, we only need to import the Tensorflow operations that our model uses.  This reduces the memory footprint. Its important to understand the operations your model needs rather than just using the all_operations code.  See the resources section for more on how to determine this usign the python tools for Tensorflow.  We are using Reshape, DepthwiseConv2D, FullyConnectioned and Softmax operations in our model so thats what we set up here.

```
  //
  // tflite::AllOpsResolver resolver;
  // NOLINTNEXTLINE(runtime-global-variables)
  static tflite::MicroMutableOpResolver<4> micro_op_resolver(error_reporter);
  if (micro_op_resolver.AddDepthwiseConv2D() != kTfLiteOk) {
    return;
  }
  if (micro_op_resolver.AddFullyConnected() != kTfLiteOk) {
    return;
  }
  if (micro_op_resolver.AddSoftmax() != kTfLiteOk) {
    return;
  }
  if (micro_op_resolver.AddReshape() != kTfLiteOk) {
    return;
  }
```

#### Intitialize Intepreter

Next, we create an interpreter to run the model with, by passing in the tensor_arena, operations resolver, the model and the error reporter.

```
  // Build an interpreter to run the model with.
  static tflite::MicroInterpreter static_interpreter(
      model, micro_op_resolver, tensor_arena, kTensorArenaSize, error_reporter);
  interpreter = &static_interpreter;
```

#### Allocate the arena

Next we will allocate an area of memory that the tensors are going to run out of.

```
  // Allocate memory from the tensor_arena for the model's tensors.
  TfLiteStatus allocate_status = interpreter->AllocateTensors();
  if (allocate_status != kTfLiteOk) {
    TF_LITE_REPORT_ERROR(error_reporter, "AllocateTensors() failed");
    return;
  }
```

#### Define Model Inputs

And then we set up some input buffers for the Tensors so we can feed the microphone data to the model.  You mad the mode_input to the intepreter input 

```
  // Get information about the memory area to use for the model's input.
  model_input = interpreter->input(0);
  if ((model_input->dims->size != 2) || (model_input->dims->data[0] != 1) ||
      (model_input->dims->data[1] !=
       (kFeatureSliceCount * kFeatureSliceSize)) ||
      (model_input->type != kTfLiteInt8)) {
    TF_LITE_REPORT_ERROR(error_reporter,
                         "Bad input tensor parameters in model");
    return;
  }
  model_input_buffer = model_input->data.int8;
```

Now that Tensorflow is set up, we can configure th eother applications we will need.  FeatureProvider is an app that converst the audio input to Spectograms.  RecognizeCommands is the code we use uise to do something about the commands we recognise.

```
  // Prepare to access the audio spectrograms from a microphone or other source
  // that will provide the inputs to the neural network.
  // NOLINTNEXTLINE(runtime-global-variables)
  static FeatureProvider static_feature_provider(kFeatureElementCount,
                                                 feature_buffer);
  feature_provider = &static_feature_provider;

  static RecognizeCommands static_recognizer(error_reporter);
  recognizer = &static_recognizer;

  previous_time = 0;
}
```

#### Setup the main loop

```
// The name of this function is important for Arduino compatibility.
void loop() {
  // Fetch the spectrogram for the current time.
  const int32_t current_time = LatestAudioTimestamp();
  int how_many_new_slices = 0;
  TfLiteStatus feature_status = feature_provider->PopulateFeatureData(
      error_reporter, previous_time, current_time, &how_many_new_slices);
  if (feature_status != kTfLiteOk) {
    TF_LITE_REPORT_ERROR(error_reporter, "Feature generation failed");
    return;
  }
  previous_time = current_time;
  // If no new audio samples have been received since last time, don't bother
  // running the network model.
  if (how_many_new_slices == 0) {
    return;
  }

  // Copy feature buffer to input tensor
  for (int i = 0; i < kFeatureElementCount; i++) {
    model_input_buffer[i] = feature_buffer[i];
  }

  // Run the model on the spectrogram input and make sure it succeeds.
  TfLiteStatus invoke_status = interpreter->Invoke();
  if (invoke_status != kTfLiteOk) {
    TF_LITE_REPORT_ERROR(error_reporter, "Invoke failed");
    return;
  }

  // Obtain a pointer to the output tensor
  TfLiteTensor* output = interpreter->output(0);
  // Determine whether a command was recognized based on the output of inference
  const char* found_command = nullptr;
  uint8_t score = 0;
  bool is_new_command = false;
  TfLiteStatus process_status = recognizer->ProcessLatestResults(
      output, current_time, &found_command, &score, &is_new_command);
  if (process_status != kTfLiteOk) {
    TF_LITE_REPORT_ERROR(error_reporter,
                         "RecognizeCommands::ProcessLatestResults() failed");
    return;
  }
  // Do something based on the recognized command. The default implementation
  // just prints to the error console, but you should replace this with your
  // own function for a real application.
  RespondToCommand(error_reporter, current_time, found_command, score,
                   is_new_command);
}
```

## Keywork Detection Model and code

I the project explorer, open the *includes* directory.  

### model.cc

This code contains the tensorflow model that was trained in the Cloud.  The model itself is just a hexidecimal dump of the model that was trained usign Tensorflow.  This model is stored a variable *g_model[]* in the C source file.  The model itself is a TinyConv or Convolution Neural Network model that has been trained using 8bit quantised audio data.

```
#include "model.h"

alignas(8) const unsigned char g_model[] = {
  0x20, 0x00, 0x00, 0x00, 0x54, 0x46, 0x4c, 0x33, 0x00, 0x00, 0x00, 0x00,
  0x00, 0x00, 0x12, 0x00, 0x1c, 0x00, 0x04, 0x00, 0x08, 0x00, 0x0c, 0x00,
  0x10, 0x00, 0x14, 0x00, 0x00, 0x00, 0x18, 0x00, 0x12, 0x00, 0x00, 0x00,
...
  0x03, 0x00, 0x00, 0x00
};
unsigned int micro_speech_tflite_len = 18712;
```



Note the len variable at the bottom of the file.  You will need to update this when you train your own models with the size of the C variable so that the microcopntroller can allocate the right amount of memory for it.  Yes!  you need to worry about memory usage now because your dealing with a microcontroller and space is finite :-)



In here you will see the supporting code from the Tensorflow project that allows the Edukit enable the microphone, write microphone data to a ringbuffer, read the buffer and convert the audio to a spectogram, run inferences using the Tensroflow model, confirm if a command was detected and respond to that command.

## Tying it all together

Now that you have seen the code layout in the project, you can tie it all together by running the tflite task in the main.c for the application.