---
layout: post
title: "Running Microdot on ESP32"
date: 2023-02-12 00:00:01 +0600
categories: [ESP32, MicroPython, Microdot]
post_image: "/assets/images/blog/??"
author: thecapn32
---

## Introduction

The ESP32 microcontroller is a popular choice for IoT applications due to its built-in Wi-Fi and Bluetooth connectivity, which make it easier to connect to the internet and other devices. One of the programming languages optimized for microcontrollers such as ESP32 is MicroPython. With MicroPython, developers can program the ESP32 using Python syntax, which is a high-level, easy-to-use language that allows them to focus on the core logic and functionality of their IoT products, rather than spending a lot of time on low-level details.

MicroPython is particularly useful for prototyping and rapid development, as it allows developers to quickly create a proof-of-concept or minimum viable product. This is because MicroPython comes with a built-in development environment and offers a range of libraries and modules for working with common IoT protocols and devices, such as Wi-Fi, Bluetooth, and sensors. Moreover, MicroPython's built-in REPL (read-eval-print loop) enables rapid testing and debugging, saving a lot of time during development.

Another useful tool for the ESP32 microcontroller is MicroDot, a MicroPython-based framework for building web applications. MicroDot simplifies the process of creating web applications for the ESP32, allowing developers to build web interfaces for their IoT products with minimal effort. With MicroDot, developers can create web applications that can be accessed from any device with a web browser, providing an easy and convenient way to interact with their IoT products.

## Getting Started

To begin working with MicroPython on the ESP32 you have to start by installing the MicroPython firmware using a tool called esptool.py, which lets you flash the firmware onto the ESP32's flash memory. Once the firmware is installed, you can add MicroPython modules like Microdot to extend the functionality of your program. Next, you can connect to the ESP32 using a serial terminal or WebREPL and start writing and executing Python code on the microcontroller.

## Environment Setup

In order to use Microdot with ESP32, we need to follow certain steps. Firstly, we need to have python3 installed on our system. Next, we need to install two python packages: esptool and adafruit-ampy. Esptool is used to upload firmware to the ESP32 flash memory and ampy is used to add Microdot files to ESP32.

```sh
python3 -m pip install esptool
python3 -m pip install adafruit-ampy
```
## Uploading Micropython Firmware

Next step is downloading MicroPython firmware for your board from the Micropython [Micropython Website](https://micropython.org/download/esp32/), Once downloaded, you can erase the flash on your board using the following command in the terminal:

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

For Windows users, you can connect to the board through a serial port using PuTTY. You can configure PuTTY by entering the serial port accordingly. Here is an example of basic PuTTY settings for ESP32:

![Putty Basic Setting](/assets/images/blog/esp32-microdot/PuTTY_BasicSettings.png "Putty Basic Setting")

## Programming the board

Once you have connected to the ESP32 board, you will need to establish a Wi-Fi connection. Here is an example code snippet for connecting to Wi-Fi:

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
import uasyncio
from microdot_asyncio import Microdot

app = Microdot()
```

Next, create a route for the server to handle, in this case a simple hello world response for the root path /
```python
@app.route('/')
def index(request):
    return 'Hello, world!'

app.run()
```

## Testing Webserver

Now you can make requests to the webserver with curl or browser by entering the ip address of your esp32 device:

```sh
curl http://{your-device-ip}
```

After reading a request you must see "Hello, world!" message on the webpage or printed in the terminal.

![Curl Request Result](/assets/images/blog/esp32-microdot/result-curl.png "Curl Request Result")

![Browser Request Result](/assets/images/blog/esp32-microdot/result-browser.png "Browser Request Result")