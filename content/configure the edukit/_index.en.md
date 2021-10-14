+++
title = "Configure the Edukit"
chapter = true
weight = 3
pre = "<b>3. </b>"
+++

In this section we need to set up the WiFi so that your device can connect to the internet.  

1. Click the PlatformIO logo on the VS Code activity bar (left most menu).
2. From the Quick Access menu, under Miscellaneous, select New Terminal. The terminal viewport should load with a new terminal labeled PlatformIO CLI.
3. Run the following command to open the configuration menu for your device.
:::code{showCopyAction=true}
pio run --environment core2foraws --target menuconfig
:::
![AWS logo](/static/idf_menuconfig-aws_endpoint.en.webp)
4. Use the direction keys on your keyboard to go to Component config –> Amazon Web Services IoT Platform and open AWS IoT Endpoint Hostname to set the string. You can paste the address you copied moments ago into the box and hit enter to set that symbol. Next, go back to the configuration home screen by pressing the ESC key twice. Then select AWS IoT EduKit Configuration from the menu. Set your WiFi SSID and WiFi Password with your Wi-FI credentials. Once you are finished, press the s button on your keyboard to save, confirm the location of the file by pressing enter, followed by q to quit.
