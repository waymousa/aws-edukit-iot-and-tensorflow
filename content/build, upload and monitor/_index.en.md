+++
title = "Build, upload and monitor"
chapter = true
weight = 5
pre = "<b>5. </b>"
+++

In this section we build and deploy the code to the Edukit, then monitor it to see if the model is detecting keywords..  

1. Click the PlatformIO logo on the VS Code activity bar (left most menu).
2. From the Quick Access menu, under Miscellaneous, select New Terminal. The terminal viewport should load with a new terminal labeled PlatformIO CLI.
3. Run the following command to build the firmware for your device.
```
pio run --environment core2foraws
```
4. With the build sucessful, run the following command to upload the copmpiled code to your device over USB.
```
pio run --environment core2foraws --target upload
```
5. Lastly, monitor the serial output from the device on your host machine.
```
pio run --environment core2foraws --target monitor
```
You should now see the serial output showing a continous stream of recordings being exampined for keywords.  Try it out by speakling clearly into the microphone on the Edukit and saying "yes" or "no".


![AWS logo](/images/serial_output.png)