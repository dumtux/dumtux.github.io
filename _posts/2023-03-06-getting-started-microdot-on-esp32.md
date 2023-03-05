---
layout: post
title: "Getting Started with Microdot on ESP32"
date: 2023-03-06 00:00:01 +0600
categories: [ESP32, MicroPython, Microdot]
post_image: "/assets/images/blog/esp32-microdot.png"
author: thecapn32
---


## Introduction

The ESP32 microcontroller is a popular choice for IoT applications due to its built-in Wi-Fi and Bluetooth connectivity, which make it easier to connect to the internet and other devices. One of the programming languages optimized for microcontrollers such as ESP32 is [MicroPython](https://micropython.org/), which is a lean and efficient implementation of the Python 3.

Flask and Django are widely-used Web frameworks in Python.
For MicroPython, there is the [Microdot](https://github.com/miguelgrinberg/microdot), a minimalistic Python web framework inspired by Flask.

## Environment Setup

In order to use Microdot with ESP32, we need to follow certain steps.

* Install [esptool](https://github.com/espressif/esptool). Esptool is used to upload firmware to the ESP32 flash memory.
* Install [adafruit-ampy](https://github.com/scientifichackers/ampy).  Ampy is used to add Microdot files to ESP32.

```sh
python3 -m pip install esptool
python3 -m pip install adafruit-ampy
```

## Uploading Micropython Firmware

Next step is downloading MicroPython firmware for your board from the [Micropython Website](https://micropython.org/download/esp32/), Once downloaded, you can erase the flash on your board using the following command in the terminal:

```sh
esptool.py --port /dev/ttyUSB0 erase_flash
```

Replace /dev/ttyUSB0 with the appropriate serial port for your board (COMx for windows).
Then, you can upload the .bin file to your ESP32 using the following command:

```sh
esptool.py --chip esp32 --port /dev/ttyUSB0 write_flash -z 0x1000 file_name.bin
```

After completing the firmware upload process, the next step is to add the Microdot library to the ESP32 file system. You can download the Microdot library from its Github page [here](https://github.com/miguelgrinberg/microdot). To achieve this, open your terminal and run the following commands:

```sh
ampy --port DEVICE_PORT put FILE_ADDRESS/microdot.py /microdot.py
ampy --port DEVICE_PORT put FILE_ADDRESS/microdot_asyncio.py /microdot_asyncio.py
```

## Connecting to esp32 Serial Port

Now the board is ready to be programmed, To start programming the ESP32 board, we need to connect to its serial port and begin writing Micropython code. If you are a Linux or Mac user, you can connect to the ESP32 board using the screen CLI by entering the following command in your terminal: 

```sh
screen /dev/SERIAL_PORT 115200
```

For Windows users, you can connect to the board through a serial port using [PuTTY](https://www.putty.org/). You can configure PuTTY by entering the serial port accordingly. Here is an example of basic PuTTY settings for ESP32:

![Putty Basic Setting](/assets/images/blog/esp32-microdot/PuTTY_BasicSettings.png "Putty Basic Setting")

## Programming the board

Once you have connected to the ESP32 board, you will need to establish a Wi-Fi connection. Here is an example code snippet for establishing Wi-Fi connection:

```python
import network

ssid = "your_SSID"
password = "your_WIFI_password"

station = network.WLAN(network.STA_IF)
station.active(True)
station.connect(ssid, password)

while not station.isconnected():
    pass

print("Connection successful")
print(station.ifconfig())
```

This code will first import the network module, which is used to connect the ESP32 to a Wi-Fi network. You will need to replace the your_SSID and your_WIFI_password placeholders with the name and password of your Wi-Fi network.

Next, the code creates a station object which represents the Wi-Fi connection. It then activates the station object, and calls the connect method with the SSID and password. The code then enters a loop that waits until the connection is established, and finally prints a message confirming the successful connection and the IP address of the board.
now it is time to run microdot on esp32 and start serving webpages, add the following code to esp32 REPL:

Now we can import Microdot module and start writing the webserver code. First, import the required libraries and create an instance of the Microdot class:

```python
from microdot_asyncio import Microdot

app = Microdot()
```

Next, create a route for the server to handle, in this case a simple hello world response for the root path /
```python
@app.route('/')
def index(request):
    return 'Hello, world!'

app.run(port=80)
```

## Testing Webserver

Now you can make requests to the webserver with curl or browser by entering the ip address of your esp32 device:

```sh
curl http://{your-device-ip}
```

After reading a request you must see "Hello, world!" message on the webpage or printed in the terminal.

![Curl Request Result](/assets/images/blog/esp32-microdot/result-curl.png "Curl Request Result")

![Browser Request Result](/assets/images/blog/esp32-microdot/result-browser.png "Browser Request Result")

## Toggling LED from Webapp

Next thing we want to in the article is making an led blink . We need to reset esp32 board and starting writing code to it.

Define the LED GPIO as an output, selecting the GPIO number according to your board:

```python
import machine

gpioNum = 5
led = machine.Pin(gpioNum, machine.Pin.OUT)
```

You can test the LED working with the following code:

```python
led.value(0)
led.value(1)
```

Write the code to connect to the WiFi station:

```python
import network

ssid = "your_SSID"
password = "your_WIFI_password"

station = network.WLAN(network.STA_IF)
station.active(True)
station.connect(ssid, password)

while not station.isconnected():
    pass

print("Connection successful")
print(station.ifconfig())
```

now we need to write microdot code to server our webserver

```python
from microdot_asyncio import Microdot

app = Microdot()

htmlDoc = '''
<!DOCTYPE html>
<html>

<body>

    <h1>PCBCrew Team</h1>

    <button type="button" onclick="toggleLed()">Toggle Led</button>

</body>

</html>

<script>
    async function toggleLed() {
        const resp = await fetch(
            "http://192.168.43.238/toggle"
        );
        resp = await resp.json();
    }
</script>'''

@app.route('/')
def index(request):
   return htmlDoc, 200, {'Content-Type': 'text/html'}

@app.route('/toggle')
def toggleLed(request):
    led.value(not led.value())
```

It is time to start webserver with:

```python
app.run(port=80)
```

After getting webserver running reach out to the browser and enter your board IP accordingly, A button will appear in the webpage, clicking this button will toggle led connected to the board:

![LED Toggling with Microdot on ESP32](/assets/images/blog/esp32-microdot/led-toggle.png)

## Conclusion (Further Reading)

Microdot also supports `async` keywords for routing just like [FastAPI](https://fastapi.tiangolo.com/) does.
We found these libraries and articles are worth to read:

* [uasyncio](https://docs.micropython.org/en/latest/library/uasyncio.html), asynchronous I/O scheduler
* [Make Your Microcontroller Multi-task With Asynchronous Programming](https://yiweimao.github.io/blog/async_microcontroller/)
