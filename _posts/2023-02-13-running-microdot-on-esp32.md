---
layout: post
title: "Running Microdot on ESP32"
date: 2023-02-12 00:00:01 +0600
categories: [ESP32, MicroPython, Microdot]
post_image: "/assets/images/blog/??"
author: thecapn32
---

## Introduction

The ESP32 is a powerful microcontroller that has built-in Wi-Fi and Bluetooth connectivity, making it an ideal choice for a wide range of Internet of Things (IoT) applications. MicroPython is a version of the Python programming language that is optimized for use on microcontrollers such as the ESP32, making it possible to program the microcontroller using Python syntax. Using MicroPython with the ESP32 can speed up the development process for IoT products by providing a high-level, easy-to-use programming language and development environment. This enables developers to write code quickly and easily, without worrying about the low-level details of programming in languages like C.

MicroPython is especially useful for prototyping and rapid development, where the goal is to get a working proof-of-concept or minimum viable product up and running as quickly as possible. With MicroPython, developers can focus on the core logic and functionality of their IoT products, without spending a lot of time on the details of working with hardware and managing memory and other low-level resources.

In addition, MicroPython provides a range of libraries and modules for working with common IoT protocols and devices, such as Wi-Fi, Bluetooth, and sensors. This further speeds up the development process, while MicroPython's built-in REPL (read-eval-print loop) enables rapid testing and debugging, saving a lot of time during development.

To get started with MicroPython on the ESP32, you'll first need to install the MicroPython firmware using a tool called esptool.py. This allows you to flash the firmware onto the ESP32's flash memory. Once the firmware is installed, you can connect to the ESP32 using a serial terminal or WebREPL, and begin writing and executing Python code on the microcontroller.

Popular projects that can be built using the ESP32 and MicroPython include home automation systems, environmental sensors, and robots. With its built-in Wi-Fi and Bluetooth connectivity, the ESP32 is particularly well-suited for projects that require remote access or communication with other devices.

MicroDot is a MicroPython library that enables the creation of web servers on microcontrollers. It provides a simple and lightweight way to serve static and dynamic content over HTTP, making it useful for creating web-based interfaces for IoT devices and other embedded systems. With MicroDot, you can build web applications using familiar technologies like HTML, CSS, and JavaScript, and the library is designed to work with popular web frameworks like Flask and Django. MicroDot is built on top of MicroPython's built-in asyncio library, which enables asynchronous I/O operations on a single-threaded event loop, making it possible to serve HTTP requests on a microcontroller without blocking other operations or consuming excessive system resources.

By using MicroDot to create a web server on ESP32, you can create a web-based user interface for your IoT project. This can be especially useful for projects that require remote access or control, as it allows you to interact with the device from anywhere with an internet connection.

### Environment Setup

For running microdot on esp32 first we must have python3 installed and then we need to install these python packages
```sh
python3 -m pip install esptool
python3 -m pip install adafruit-ampy
```
esptool will be used to upload firmware to esp32 flash and ampy is used to add microdot files to esp32. 

now go to this [Micropython](https://micropython.org/download/?) and download micropython firmware (.bin file) for your esp32 or esp32s3 board. After this first we need to erase our boards flash then we need to upload the .bin file to esp32 using this command in terminal:
```sh
esptool.py --port /dev/ttyUSB0 erase_flash
esptool.py --chip esp32 --port /dev/ttyUSB0 write_flash -z 0x1000 file_name.bin
```

after these steps now we need to add microdot modules to our esp32 file system with the help of ampy. First we need to download microdot library from it's github page [Microdot](https://github.com). then we need to add microdot.py and microdot_asyncio.py to esp32, we can achive that by using ampy tool. 

```sh
ampy --port DEVICE_PORT put {address}/microdot.py /microdot.py
ampy --port DEVICE_PORT put {address}/microdot_asyncio.py /microdot_asyncio.py
```
after this we need to connect to esp32 serial port, if you are based on linux or mac you can use "screen" to connect to esp32, if you are using windows you can use Putty to connect to esp32.

```sh
screen /dev/SERIAL_PORT 115200
```

for windows connect through serial port with PuTTY and enter serial port accordingly
![Putty Basic Setting](/assets/images/blog/esp32-microdot/PuTTY_BasicSettings.png "Putty Basic Setting")
now we need to make a wifi connection with our esp32, use this code snippet to connect to wifi:
```python
import network
sta_if = network.WLAN(network.STA_IF)
sta_if.active(True)
sta_if.connect('your-ssid', 'your-password')
while(!sta_if.isconnected()):
    pass
```

then wait till it connects to wifi, now get the ip address of your esp32 board by entering:

```python
sta_if.ifconfig()
```

now it is time to run microdot on esp32 and start serving webpages, add the following code to esp32 REPL:


```python
import uasyncio
from microdot_asyncio import Microdot

app = Microdot()

@app.route('/')
def index(request):
    return 'Hello, world!'

app.run()
```
now you can make requests to the webserver with curl or browser by entering the ip address of your esp32 device

```sh
curl http://{your-device-ip}
```
![Curl Request Result](/assets/images/blog/esp32-microdot/result-curl.png "Curl Request Result")
after reading a request you must see "Hello, world!" message on the webpage or printed in the terminal.
![Browser Request Result](/assets/images/blog/esp32-microdot/result-browser.png "Browser Request Result")
