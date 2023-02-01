---
layout: post
title: "Creating a Svelte Web Application For ESP32"
date: 2023-02-01 00:00:01 +0600
categories: [ESP32, Svelte]
post_image: "/assets/images/blog/esp32-svelte.png"
author: Mohammad
---

## Introduction

The ESP32 is a versatile microcontroller, engineered for the Internet of Things (IoT) and connected devices. Developed by Espressif Systems, it is based on the ESP-IDF framework and offers dual connectivity options of Wi-Fi and Bluetooth for seamless integration with both local and cloud-based networks. This versatility extends to its capability of hosting dynamic and static web pages, as well as responding to HTTP requests from various clients. The ESP32's broad range of capabilities, including home automation and wearable device applications, make it a good solution for IoT development.

In this article, we will show you how to create a simple webpage using the Svelte framework and run it on an ESP32. Svelte is a JavaScript framework known for its compiler that produces optimized code for better browser performance, resulting in faster speeds, smaller code size, and a simplified development process. These features make Svelte a perfect match for building IoT applications on the ESP32. The combination of Svelte's optimization, fast performance, and support for accessibility and network connectivity, provides developers with a robust and efficient solution for building next-generation, accessible, and innovative IoT applications.

## Setting up environment

For setting up the development environment Windows users will require WSL (Windows Subsystem for Linux) while macOS and Linux users already have a terminal available. Along with that, make sure to have Node.js installed on your system. You can follow the instructions from [nodejs.org](https://nodejs.org/en/) for installation. Additionally, you'll need to install the ESP-IDF framework to fully utilize the ESP32 for IoT applications. You can find detailed instructions for installing ESP-IDF on your system on Espressif's website for [Windows](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/windows-setup.html) users and here for [Linux and macOS](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html) users.

## Create Svelte Skeleton project

first create a skeleton Svelte project. This can be done by running the following command in your terminal:

```sh
npm create svelte@latest web-app
```

select skeleton svelete project, During the installation, you will be prompted with a series of questions. You can select "No" for all of them for this project.
Next, navigate into the newly created project directory and install the required dependencies by running:

```sh
cd web-app
npm install
```

Now that the project has been created, change the content of `src/routes/+page.svelte` to the following:

```html
<script>
  function buttonClick() {
    alert("Button clicked");
  }
</script>

<h1>Hello World</h1>
<button class="button" on:click="{buttonClick}">Click!</button>

<style>
  .button {
    background-color: #4caf50; /* Green */
    border: none;
    color: white;
    padding: 15px 32px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
  }
</style>
```

You can run the following command in your terminal to launch the web page:

```sh
npm run dev -- --open
```

This will open the webpage in your browser, where you will be able to see the "Hello World" message and a button. When you click the button, you will see an alert message displaying "Button clicked."

![Svlete Skeleton Project](/assets/images/blog/esp32-svelte/sevlte-sample-page.png "svelte skeleton project")

---

To upload the webpage to the ESP32 and serve it as a web server, we need to build the Svelte project first. To do this, we need to install a package by running:

```sh
npm i -D @sveltejs/adapter-static
```

Then, modify the svelte.config.js file by changing import adapter from `@sveltejs/adapter-auto` to import adapter from `@sveltejs/adapter-static`.

```js
- 1     import adapter from '@sveltejs/adapter-auto';
+ 1     import adapter from '@sveltejs/adapter-static';
```

Additionally, create a file `src/routes/+layout.js` and add the following line to it:

```js
export const prerender = true;
```

To build the Svelte project, run:

```sh
npm run build
```

After the build process is complete, a build folder will be generated. To make it easier to manage the files, rename the build folder to "web". This renamed folder will later be copied to another directory for use on the ESP32 project.

## Create ESP32 WiFi station project

To make an esp-idf project, we will use the wifi station example from esp-idf directory esp-idf/example/wifi/get_started/sofAP. Copy the folder to your project directory and navigate to it in your terminal or esp-idf WSL (if using Windows). Then, run the command:

```sh
idf.py menuconfig
```

Under the Serial Flasher Config tab, select the Flash size option and set it to the appropriate size for your module.

![flash-config](/assets/images/blog/esp32-svelte/flash-size-config.png "flash-config")

We need to change Flash Partition Table because we need to upload web folder on flash, under Partition Table tab, select Partition Table,

![flash partition](/assets/images/blog/esp32-svelte/custom-partition-config.png "flash partition")

and also Under Component Config, SPIFFS Configuration tab change SPIFFS File System Maximum Size to the required size according to your needs. This will ensure that the files generated by Svelte will be properly served through the ESP32.

![max-name](/assets/images/blog/esp32-svelte/spiffs-max-name-config.png "max-name")

at last goto Example Configuration and set SSID and Password of your esp32 wifi station

![Svlete Skeleton Project](/assets/images/blog/esp32-svelte/ssid-pass-config.png "svelte skeleton project")

The project configuration is complete. The next step is to add files and make necessary modifications. Start by creating the "softAP/partitions.csv" file to partition the flash storage.
The content of softAP/partitions.csv should be as follows:

```
# Name,   Type, SubType, Offset,  Size, Flags
# Note: if you have increased the bootloader size, make sure to update the offsets to avoid overlap
nvs,      data, nvs,     ,        0x6000,
phy_init, data, phy,     ,        0x1000,
factory,  app,  factory, ,        1M,
spiffs,   data, spiffs,  ,        1M,
```

After this step, copy the "web" directory from the `web-app` Svelte project to the ESP-IDF `softAP` project directory. Then, add the following line to softAP/main/CMakeLists.txt:

```
spiffs_create_partition_image(spiffs ../web FLASH_IN_PROJECT)
```

now we need to modify the source code in file `softAP/src/main/softap_example_main.c`

```c
#include "esp_http_server.h"
#include "esp_spiffs.h"
```

and defining a global variable:

```c
static httpd_handle_t server = NULL;
```

add the following code at the end of the main function to configure and start the HTTP server:

```c
httpd_config_t config = HTTPD_DEFAULT_CONFIG();
config.uri_match_fn = httpd_uri_match_wildcard;
ESP_ERROR_CHECK(httpd_start(&server, &config));

httpd_uri_t default_url = {
    .uri = "/*",
    .method = HTTP_GET,
    .handler = on_default_url};

httpd_register_uri_handler(server, &default_url);
```

Finaly, Also add the following function to get files from spiffs and send it via http:

```c
static esp_err_t on_default_url(httpd_req_t *req)
{
    ESP_LOGI(TAG, "URL: %s", req->uri);

    esp_vfs_spiffs_conf_t esp_vfs_spiffs_conf = {
        .base_path = "/spiffs",
        .partition_label = NULL,
        .max_files = 5,
        .format_if_mount_failed = true};
    esp_vfs_spiffs_register(&esp_vfs_spiffs_conf);

    char path[600];
    if (strcmp(req->uri, "/save") == 0)
        httpd_resp_send_404(req);
    if (strcmp(req->uri, "/") == 0)
    {
        strcpy(path, "/spiffs/index.html");
    }
    else
    {
        sprintf(path, "/spiffs%s", req->uri);
    }
    char *ext = strrchr(path, '.');
    if (strcmp(ext, ".css") == 0)
    {
        httpd_resp_set_type(req, "text/css");
    }
    if (strcmp(ext, ".js") == 0)
    {
        httpd_resp_set_type(req, "text/javascript");
    }
    if (strcmp(ext, ".png") == 0)
    {
        httpd_resp_set_type(req, "image/png");
    }

    FILE *file = fopen(path, "r");
    if (file == NULL)
    {
        httpd_resp_send_404(req);
        esp_vfs_spiffs_unregister(NULL);
        return ESP_OK;
    }
    char lineRead[256];
    while (fgets(lineRead, sizeof(lineRead), file))
    {
        httpd_resp_sendstr_chunk(req, lineRead);
    }
    httpd_resp_sendstr_chunk(req, NULL);

    esp_vfs_spiffs_unregister(NULL);
    return ESP_OK;
}
```

Now we can flash the board:

```sh
idf.py -p PORT flash monitor
```
