
---
title: How to make a DSTIKE deauther for less than 2€
tags: 
categories: Wifi
---

In this entry I will show how to create the cheapest DSTIKE deauther to constantly deauthenticate devices in a wifi netowrk.

The DSTIKE Deauther is a compact, programmable device leveraging the ESP8266 or ESP32 microcontroller platforms. Its primary function is to conduct Wi-Fi deauthentication attacks by sending deauthentication packets to targeted networks.

## Prerequisites

ESP01S. [Link](https://es.aliexpress.com/item/1005006323836809.html?spm=a2g0o.productlist.main.1.28415b914FtJbl&algo_pvid=ca944ce8-7dae-48fb-9d55-2356d438dcf7&algo_exp_id=ca944ce8-7dae-48fb-9d55-2356d438dcf7-0&pdp_npi=4%40dis%21EUR%214.21%211.38%21%21%2132.06%2110.51%21%40211b61a417088881547937946ebbb6%2112000036764337055%21sea%21ES%213170010347%21&curPageLogUid=6AcsFvLv3Uhi&utparam-url=scene%3Asearch%7Cquery_from%3A)

USB ESP01S Adaptor. [Link](https://es.aliexpress.com/item/1005003772310662.html?spm=a2g0o.productlist.main.21.2d2d316fsmZ9iq&algo_pvid=b1e91228-dbab-4a9d-b1be-8f97de1fc506&algo_exp_id=b1e91228-dbab-4a9d-b1be-8f97de1fc506-10&pdp_npi=4%40dis%21EUR%212.12%211.44%21%21%212.24%211.52%21%40211b613117088881362586120e8ee5%2112000027112227203%21sea%21ES%213170010347%21&curPageLogUid=7Dhw4WLwdVY0&utparam-url=scene%3Asearch%7Cquery_from%3A)

[Buy both together link](https://es.aliexpress.com/item/1005002975811689.html?spm=a2g0o.productlist.main.5.6be9767fz26r9g&algo_pvid=639524e7-0b00-42fb-ad74-19ae97538200&algo_exp_id=639524e7-0b00-42fb-ad74-19ae97538200-2&pdp_ext_f=%7B%22sku_id%22%3A%2212000023045787079%22%7D&pdp_npi=3%40dis%21EUR%214.09%212.91%21%21%21%21%21%40211beca116792522848537831d0703%2112000023045787079%21sea%21ES%213767851196&curPageLogUid=JBsjD0LjXKUa)

Once you have the two pieces, connect the ESP01S to the USB adapter (LIKE THIS!):

![Screenshot1.jpg](/assets/img/screenshots/deauther/Screenshot1.jpg)

⚠️Be careful with connecting the esp01s wrong, it has to be like in the picture above, not the other way, you can get a burn if you turn it upside down.

There are 8 pins but each pin is not the same as the other, each one has its function, if you put them upside down, the power remains in the adaptor heating the chips and does not pass to the ESP01S ⚠️

Install the drivers for the CH341 chip, utilized with ESP01 USB adaptor from [here](https://www.wch-ic.com/downloads/CH341SER_ZIP.html)

Then, it's time to flash the DSTIKE deauther firmware, so first download it from the [original website.](https://deauther.com/docs/download/)

.BIN --> DSTIKE --> NODECMU

Now let's flash the deauther, for that I will use [this tool.](https://github.com/nodemcu/nodemcu-flasher)

Once you have all the above, open the flashing tool and go to "Advanced" tab. You need to set the BAUD to 115200, Flash size 1MB, Flash speed 40MHz and SPI Mode DOUT, in the case of the NodeMCU Flasher:

![Screenshot_6.png](/assets/img/screenshots/deauther/Screenshot_6.png)

Move to "Config" tab and set the file to flash:

![Screenshot_7.png](/assets/img/screenshots/deauther/Screenshot_7.png)

Finally move to "Operation" tab and set your COM port:

![Screenshot_5.png](/assets/img/screenshots/deauther/Screenshot_5.png)

It will probably be COM4 but in case yours is different go to "Device Manager" --> Ports (COM and LTP):

![Screenshot_8.png](/assets/img/screenshots/deauther/Screenshot_8.png)

After some minutes the flashing process will be finished and the "pwned" wifi network will be shown.

If it does not appear unplug the USB and plug it again.

![Screenshot2.jpg](/assets/img/screenshots/deauther/Screenshot2.jpg)

By default, the password is "deauther" but you will be able to change it later in settings. Go to 192.168.4.1 and configure it :) 

![Screenshot_9.png](/assets/img/screenshots/deauther/Screenshot_9.png)