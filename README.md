This project solves a simple problem: you want to know of a door (or window, or gate) is open or not. This project toggles a MQTT topic from false to true in case the door opens, and back.
The superpower if it is the power consumption. The ESP8266 is not powered up all the time and monitoring a switch, but the switch is the power itself. Meaning: when the ESP is powered, the door is deemed as open, and when the power is shut off, the door is deemed as closed.

*Why the heck this way?*

Because now you can use a simple NC ("normally closed") reed contact to power the ESP up when the door meets it's final position (if there is any; my garage gate has one). Once the gate is closed again, the power is shut off. This way the ESP consumes power only while the gate is open.

Of course you can use a NO ("normally open") reed contact and mount the ESP on the closed door, so power is again enabled when opening the door. Or in case your door is normally open and only closed in rare situations, you might build the code the other way round.

*How the heck does that work?*

I'm using a simple feature: MQTT last will. With that you can define a message that will be sent to the MQTT broker once the connection got lost (in fact, the broker stores it from the beginning and sends it himself). So here we actively send the "TRUE" (door open) and define the "FALSE" (door closed) as last will. Once the ESP looses the connection, the last will will be executed. Smart, isn't it?

Be aware that in case the ESP looses the connection for any other reasons than powering down, the flag will be "FALSE" even in case of an open door. So if you depend on being on the safe side, you better change to code so that the ESP declares "FALSE" (close) activly, and anything other is declared as "open".



-----------------------------------------
HOW TO GET IT UP AND RUNNING
-----------------------------------------

__hardware you will need__

- 1x ESP8266 (ESP32 will also work, but then you have to figure out the WiFi on your own because it's slightly different)
- 1x reed contact (NC = normally closed preferred, see above)

__MQTT__

To make MQTT work you have to add your MQTT broker's URL ("mqttServer"), the username ("mqttUsername") and password ("mqttPassword"). You may also want to adjust the topics - by default it's using my structure ("monitor/garagentor").  

__WiFi__

Open the garagedoor.c code in your favorite IDE (e.g. Arduino IDE), connect the ESP to your PC, and flash the firmware.

Once it has rebooted, it will look for known WiFi networks (there are none), than it will switch to AP mode. You will see a new WiFi named "garagentor_ABCDEF" (with ABCDEF the serial number of your ESP). Connect to it, open http://192.168.4.1 with a browser, there choose the WiFi the ESP shall connect to in production, provide the password and confirm. The ESP will reboot and connect to that given network. (This process is provided by the WiFiManager.h library, not my own work.)

You should now see the MQTT topic toggling to "TRUE" until you switch off power of the ESP. The last will takes some seconds to be executed (the broker first tries to reestablish the connection), but after the connection timing out, the topic will be set to "FALSE".

Looking forward to your feedback.
