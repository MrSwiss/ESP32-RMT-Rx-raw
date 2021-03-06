# ESP32-RMT-Rx-raw
ESP32 RMT application, displaying raw RMT durations from multiple receive channels.
Duration data from all enabled channels are simply displayed on the monitor.
This requires that the ESP32 be connected to the host computer with a USB cable.
No IR protocol decode is implemented.

Why multiple channels? Most of my remotes use the standard 38KHz IR carrier frequency. One uses 56 KHz (*%#^!). I connected a 38KHz IR receiver to one GPIO and a 56KHz receiver to another GPIO. My IR sniffer now can receive and display raw codes from either frequency.

The RMT receive input is from an IR receiver sensor. Make sure that the voltage range of your sensor is compatible with the ESP32. My sensors can use VCC from 2.7 to 5.5 volts.
* The output of the IR sensor that drives the RMT receiver input is high (logic level 1) when the IR signal is idle (no IR pulses are being transmitted).
* The output of the IR sensor is driven low when the sensor detects IR pulses.  The output of the IR sensor stays low as long as the IR sensor continues to detect IR pulses.

This application formats the receive data so it is compatible with the ESP32-RMT-server transmit application, documented at https://github.com/kimeckert/ESP32-RMT-server.
* Active IR pulses beamed to the IR sensor cause the output of the sensor to go low. The RMT receives this as a logic zero. This application reports the duration of the low level as a positive integer. The ESP32-RMT-server transmit application decodes the positive integer and creates IR output pulses of the same duration.
* Durations without active IR pulses cause the output of the sensor to go high. The RMT receives this as a logic one. This application reports the duration of the high level as a negative integer. The ESP32-RMT-server transmit application decodes the negative integer and creates an idle (no IR pulses) output of the same duration.
* When the duration of either active IR pulses or no active IR pulses exceeds the RMT value RMT_IDLE_THRES_CHn channel clock ticks, the RMT will identify this as the end of an IR transmission sequence. The RMT will then append a zero duration to the data and return to the idle state.  This application reports the zero value as the final value in the durations.  It does not represent a duration of zero, it represents the end of the IR transmission sequence.

This application has been tested on an Adafruit ESP32 Feather board.
The application flashes the on-board visible LED and reports mark and space durations when responding to received RMT data.

## Examples of received data.
A single RMT item consists of two durations.<br />

  Received 34 items  
  
9012,-4478,590,-540,591,-535,588,-1659,594,-536,590,-537,592,-535,591,-535,591,-536,590,-1658,590,-1659,594,-536,588,-1659,595,-1654,591,-1659,595,-1654,595,-1656,593,-1657,593,-538,590,-536,590,-1657,594,-536,590,-538,591,-535,590,-537,593,-535,591,-1656,593,-1657,592,-537,592,-1655,594,-1655,596,-1654,593,-1654,597,0

  Received 34 items  
  
9009,-4479,595,-536,589,-538,589,-1658,593,-537,592,-535,590,-537,587,-540,591,-536,591,-1655,596,-1654,591,-540,590,-1656,594,-1656,592,-1656,591,-1660,592,-1657,592,-1657,593,-535,591,-537,591,-1655,592,-538,588,-539,591,-536,589,-538,592,-535,589,-1657,594,-1655,593,-537,593,-1654,592,-1658,594,-1656,592,-1656,595,0

  Received 2 items  
  
9006,-2227,595,0

  Received 2 items  
  
9010,-2226,593,0

  Received 2 items  
  
9011,-2223,595,0
