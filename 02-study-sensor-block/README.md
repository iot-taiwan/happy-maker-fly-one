# Sensor - Bluetooth Low Enegy

Bluetooth Low Enegy，中譯為低耗電藍芽，以下簡稱 BLE。

正式進入主題之前，先談談這個小節會帶領讀者體驗哪些實作項目。
首先我們會需要一個具有 BLE 功能的模組，目的是當成 client 端（Phone/Tablet/...）與 server 端（ARM mbed）之間溝通的橋樑，負責把 client 端的訊息傳送到 server 端。
client 端會透過一支 app 發送訊息，server 端會透過一支程式接收訊息。接著，server 端把接收到的訊息傳送到 ARM mbed websocket server，讓讀者直接透過 browser 就可以觀看訊息。要達成此一工作，我們還需要準備一個具有 Wi-Fi 功能的模組，讓 server 端具備上網的能力。將訊息傳送到 websocket server 則需要建立一個 websocket channel。這些過程串連起來之後，讀者就可以透過 app 發送訊息到 ARM mbed，並且開起 browser 看到此訊息。

## 準備工作

我們需要準備以下素材：

| Name         | Quantity |
| ------------ | -------- |     
| Arch BLE     |     1    | 
| LPC 1768     |     1    |       
| Wifi Shield  |     1    |
| 麵包板       |     1    |
| 導線         |    10    |

## 素材介紹

### Arch BLE

Arch BLE 是一塊 “mbed-Enabled” 開發版，如下圖所示，開發版上面印有 “mbed-Enabled” 的 logo，這表示我們可以利用 ARM mbed 的 Online IDE、C/C++ SDK、libraries 等等資源進行開發工作。

![圖：mbed Enabled logo ](http://imgur.com/42lNpyw.jpg)

此外，Arch BLE 內建一塊 Nordic nRF51822 chip，支援 BLE 協定，以及數個 Grove 介面，方便我們建立 BLE device。

### Wifi Shield

Wifi Shield 使用 RN171 Wi-Fi 模組，支援 802.11/b/g、TCP、UDP、FTP 等常用的通訊協定，方便我們建立各式無線網路應用。

## 實作步驟

### BLE ready

1. 將跳線帽（Jumper Cap）就定位，如下圖所示：

![圖：Arch BLE jumper cap](http://i.imgur.com/FaF5CRp.jpg)

2. LPC1768 跳線，如下圖所示：

![圖：LPC1768 jumper](http://i.imgur.com/66oiliJ.jpg)

3. Arch BLE 跳線，如下圖所示：

![圖：Arch BLE jumper](http://i.imgur.com/u6UfEhI.jpg)

4. 程式碼

｀｀｀
/* mbed Microcontroller Library
 * Copyright (c) 2006-2013 ARM Limited
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

#include "mbed.h"
#include "BLEDevice.h"

#include "UARTService.h"

#define NEED_CONSOLE_OUTPUT 0 /* Set this if you need debug messages on the console;
                               * it will have an impact on code-size and power consumption. */

#if NEED_CONSOLE_OUTPUT
#define DEBUG(...) { printf(__VA_ARGS__); }
#else
#define DEBUG(...) /* nothing */
#endif /* #if NEED_CONSOLE_OUTPUT */

BLEDevice  ble;
DigitalOut led1(LED1);

UARTService *uartServicePtr;

Serial uart(p8, p7); // p8: TXD, p7: RXD 

void disconnectionCallback(Gap::Handle_t handle, Gap::DisconnectionReason_t reason)
{
    DEBUG("Disconnected!\n\r");
    DEBUG("Restarting the advertising process\n\r");
    ble.startAdvertising();
}

void onDataWritten(const GattCharacteristicWriteCBParams *params)
{
    if ((uartServicePtr != NULL) && (params->charHandle == uartServicePtr->getTXCharacteristicHandle())) {
        uint16_t bytesRead = params->len;
        DEBUG("received %u bytes\n\r", bytesRead);
        ble.updateCharacteristicValue(uartServicePtr->getRXCharacteristicHandle(), params->data, bytesRead);
        
        for (int i = 0; i < bytesRead; i++) {
            uart.putc(params->data[i]);    
        }
    }
}

void periodicCallback(void)
{
    led1 = !led1;
}

int main(void)
{
    led1 = 1;
    Ticker ticker;
    ticker.attach(periodicCallback, 1);

    DEBUG("Initialising the nRF51822\n\r");
    ble.init();
    ble.onDisconnection(disconnectionCallback);
    ble.onDataWritten(onDataWritten);

    /* setup advertising */
    ble.accumulateAdvertisingPayload(GapAdvertisingData::BREDR_NOT_SUPPORTED);
    ble.setAdvertisingType(GapAdvertisingParams::ADV_CONNECTABLE_UNDIRECTED);
    ble.accumulateAdvertisingPayload(GapAdvertisingData::SHORTENED_LOCAL_NAME,
                                     (const uint8_t *)"BLE UART", sizeof("BLE UART") - 1);
    ble.accumulateAdvertisingPayload(GapAdvertisingData::COMPLETE_LIST_128BIT_SERVICE_IDS,
                                     (const uint8_t *)UARTServiceUUID_reversed, sizeof(UARTServiceUUID_reversed));

    ble.setAdvertisingInterval(Gap::MSEC_TO_ADVERTISEMENT_DURATION_UNITS(1000));
    ble.startAdvertising();

    UARTService uartService(ble);
    uartServicePtr = &uartService;

    while (true) {
        ble.waitForEvent();
    }
}
｀｀｀

5. 測試

安裝 nRF UART 2.0 app

下載網址：
｀｀｀
https://play.google.com/store/apps/details?id=com.nordicsemi.nrfUARTv2

｀｀｀

6. 結果

Client 端與 Arch BLE 配對成功，如下圖所示：

![圖：BLE ready](http://i.imgur.com/ccaxUx8.png)


### LPC1768 ready

1. Wifi Shield 跳線，如下圖所示：

![圖：Wifi Shield jumper](http://i.imgur.com/zKbHf57.jpg)

2. 程式碼

｀｀｀
#include "mbed.h"
#include "WiflyInterface.h"
#include "Websocket.h"


/* wifly interface:
*     - p9 and p10 are for the serial communication
*     - p19 is for the reset pin
*     - p26 is for the connection status
*     - "mbed" is the ssid of the network
*     - "password" is the password
*     - WPA is the security
*/
WiflyInterface wifly(p13, p14, p19, p26, "WWNet"/*mbed*/, "mmmmmmmm"/*password*/, WPA);

DigitalOut led1(LED1);
DigitalOut led2(LED2);
DigitalOut led3(LED3);

Serial uart(p9, p10);

int main() {
    wifly.init(); //Use DHCP
    while (!wifly.connect());
    //printf("IP Address is %s\n\r", wifly.getIPAddress());
    
    if (wifly.connect()) {
        led1 = 1;
    }

    Websocket ws("ws://sockets.mbed.org/ws/mbedschool/viewer"/*"ws://sockets.mbed.org:443/ws/demo/wo"*/);
    while (!ws.connect());
    
    ws.send("Websocket server connected.");
    
    if (ws.connect()) {
        led2 = 1;
    }

    while (1) {
        if (uart.readable()) {
            led3 = 1;
            char data[1];
            data[0] = uart.getc();
            ws.send(data);
        }
        //ws.send("WebSocket Hello World over Wifly");
        wait(1.0);
    }
}
｀｀｀

# Grove - 3Axis Digital Accelerometer

三軸加速度感測器的可以在預先不知道物體運動方向的情況下，應用各個維度的加速度感測器來檢測加速度資訊。三維加速度感測器具有體積小和輕量的特點，可以測量空間加速度，並且能夠全面準確反映物體的運動情形。本文使用的[三軸加速感測器為此](http://www.seeedstudio.com/depot/Grove-3Axis-Digital-Accelerometer15g-p-765.html)。

![三軸方向示意圖](http://i.imgur.com/ZYVs7Fe.png)

三軸方向示意圖

## 腳位接法

| HC05 pin | mbed pin   |
| -------- | ---------- |
| 1 - SCL  | P27 - SCL  |
| 2 - SDA  | P28 - SDA  |
| 3 - Vcc  | Vout - 3.3V|
| 4 - GND  | GND        |

![腳位接法範例圖](http://i.imgur.com/Z6y2NBz.png)

腳位接法範例圖

備註：ARM mbed 腳位只要是 SCL及 SDA 即可，以 LPC1768 開發版為例： P9、P10 也可使用。	

## 數據解析

感測器本身會會回傳 0~63 的結果，我們可以透過此結果依表轉換成 G 值 （1g=m/s^2） 或者角度（不建議使用），詳細對照表可參照[此文件](http://www.freescale.com.cn/files/sensors/doc/data_sheet/MMA7660FC.pdf?fpsp=1) P.26~P.27。

![數據解析對照表](http://i.imgur.com/h6il6ET.png)

數據解析對照表

## 程式碼

開始撰寫 ARM mbed 程式碼。首先，必須引入 mbed.h 及 MMA7660FC.h 標頭檔，接著將連接腳位。並且我們利用 LED 燈來顯示感測器的變化。

程式碼如下：

```
#include "mbed.h"
#include "MMA7660FC.h"
 
DigitalOut myled1(LED1);
DigitalOut myled2(LED2);
DigitalOut myled3(LED3);
DigitalOut myled4(LED4);
 
#define ADDR_MMA7660 0x98                   // I2C SLAVE ADDR MMA7660FC
 
MMA7660FC Acc(p28, p27, ADDR_MMA7660);      //sda、cl、Addr，更換接角時請務必更改此處
Serial pc(USBTX, USBRX);

// G 值對照表
float G_VALUE[64] = {0, 0.047, 0.094, 0.141, 0.188, 0.234, 0.281, 0.328, 0.375, 0.422, 0.469, 0.516, 0.563, 0.609, 0.656, 0.703, 0.750, 0.797, 0.844, 0.891, 0.938, 0.984, 1.031, 1.078, 1.125, 1.172, 1.219, 1.266, 1.313, 1.359, 1.406, 1.453, -1.500, -1.453, -1.406, -1.359, -1.313, -1.266, -1.219, -1.172, -1.125, -1.078, -1.031, -0.984, -0.938, -0.891, -0.844, -0.797, -0.750, -0.703, -0.656, -0.609, -0.563, -0.516, -0.469, -0.422, -0.375, -0.328, -0.281, -0.234, -0.188, -0.141, -0.094, -0.047};
 
 
int main() 
{
 
    Acc.init();                                                     // Initialization
    pc.printf("Value reg 0x06: %#x\n", Acc.read_reg(0x06));         // Test the correct value of the register 0x06
    pc.printf("Value reg 0x08: %#x\n", Acc.read_reg(0x08));         // Test the correct value of the register 0x08
    pc.printf("Value reg 0x07: %#x\n\r", Acc.read_reg(0x07));       // Test the correct value of the register 0x07
           
    while(1)
    {   
        myled4=1;
        float x=0, y=0, z=0;
        float ax=0, ay=0, az=0;
        
        Acc.read_Tilt(&x, &y, &z);                                  // Read the acceleration                    
        pc.printf("Tilt x: %2.2f degree \n", x);                    // Print the tilt orientation of the X axis
        pc.printf("Tilt y: %2.2f degree \n", y);                    // Print the tilt orientation of the Y axis
        pc.printf("Tilt z: %2.2f degree \n", z);                    // Print the tilt orientation of the Z axis
 
        wait_ms(100);
 
        pc.printf("x: %1.3f g \n", G_VALUE[Acc.read_x()]);          // Print the X axis acceleration
        pc.printf("y: %1.3f g \n", G_VALUE[Acc.read_y()]);          // Print the Y axis acceleration
        pc.printf("z: %1.3f g \n", G_VALUE[Acc.read_z()]);          // Print the Z axis acceleration
        pc.printf("\n");
        
        //換算成 G值
        ax = G_VALUE[Acc.read_x()];
        ay = G_VALUE[Acc.read_y()];
        az = G_VALUE[Acc.read_z()];
        
        //用 LED簡單展示數值變化
        if(ax<0){
            myled1=1;
        }else{
            myled1=0; 
        }
        if(ay<0){
            myled2=1;
        }else{
            myled2=0; 
        }
        if(az<0){
            myled3=1;
        }else{
            myled3=0; 
        }
        
        wait(1);
          
    }
}
```

## 參考資源
* https://developer.mbed.org/components/Grove-3Axis-Digital-Accelerometer/
* http://developer.mbed.org/users/edodm85/notebook/grove---3-axis-accelerometer/
* http://www.seeedstudio.com/wiki/Grove_-_3-Axis_Digital_Accelerometer%28%C2%B11.5g%29

# Grove - 3-Axis Digital Gyro

首先讓大家看一下陀螺儀的外貌，如下圖
![陀螺儀](http://i.imgur.com/eUKW5h9.jpg)


## 陀螺儀用處

陀螺儀是一種用來感測與維持方向的裝置，陀螺旋進是日常生活中常見的現象，大家小時候玩過陀螺的，知道在一定速度下，就能一直保持平衡。三軸陀螺儀最大的作用就是測量“角速度”，以判別物體的運動狀態，用於飛行器上，是要讓飛行器可以知道自己目前要去哪，以及如何保持平衡穩定的飛行方向

## 準備工作

這次的Block整合是要將陀螺儀的 Sensor Data 推送至 WebSocket。
首先第一步，我們透過 ARM mbed LPC1768 開發版將三軸陀螺儀（Grove - 3-Axis Digital Gyro）與 Wify 模組串接起，如下圖：
![Wifi + 陀螺儀](http://i.imgur.com/wZotuUa.jpg)

## 開始實作
接下來技術實作的部分就分為三個部分

* Websocket channel server 服務，本書將使用 *sockets.mbed.org*
* ARM mbed 的 Websocket client 實作
* ARM mbed 與 Grove - 3-Axis Digital Gyro實作

```
#include "mbed.h"
#include "WiflyInterface.h"
#include "Websocket.h"
#include "ITG3200.h"
 
 
/* wifly interface:
*     - p9 and p10 are for the serial communication
*     - p19 is for the reset pin
*     - p26 is for the connection status
*     - "mbed" is the ssid of the network
*     - "password" is the password
*     - WPA is the security
*/
WiflyInterface wifly(p13, p14, p19, p26, "WWNet", "mmmmmmmm", WPA);
DigitalOut led1(LED1);
ITG3200 gyro(p9, p10, 0x68);
 
 
 
int main() {
    wifly.init(); //Use DHCP
    //wifly.init("192.168.21.33","255.255.255.0","192.168.21.2");
    while (!wifly.connect());
    led1=1;
    printf("IP Address is %s\n\r", wifly.getIPAddress());
 
    Websocket ws("ws://sockets.mbed.org/ws/mbedschool/viewer");
    //Websocket ws("ws://192.168.199.159:8888");
    while (!ws.connect());
    led1=2;
 
   
    int x = 0, y = 0, z = 0, temp = 0;
    //Set highest bandwidth.
    gyro.setLpBandwidth(LPFBW_42HZ);
 
    while (1) {
        char data[256];
        wait(0.1f);
        x = gyro.getGyroX();
        y = gyro.getGyroY();
        z = gyro.getGyroZ();
        temp = gyro.getTemperature();
  
        printf("Temp: %d, X: %d, Y: %d, Z: %d\n", temp, x, y, z);
        sprintf( data , "Temp: %d, X: %d, Y: %d, Z: %d\n", temp, x, y, z );
        ws.send(data);
        
        //wait(1.0);
    }
}
```

# 3-Axio Digital Gryo 丶 3-Axio Digital Accelerometer 與 WebSocket Client 整合
各自完成各個Block後，接下來就是確認第一版「手作」土製飛行器所需要用的 sensor ，一開始大家很簡單的想， Arm Mbed Arch ELE 己經有了藍芽的功能，只要再加上 3-Axio Digital Gryo 以及 3-Axio Digital Accelerometer 這二個 sensor 就可以組成飛行器了，但是，仔細看了 Arch ELE 的資料，發現 Arch ELE 只有一個組 Serial 的 Pin 腳，所以最後還是決定用 LPC1768 來實做我們的「手作」土製飛行器。第一階段的整合測試目標將 3-Axio Digital Gryo 以及 3-Axio Digital Accelerometer 的資料透過 WebSocket Client 傳送到 Server ，制作飛行器儀表板。

因為要透過 WebSocket Client 將資料傳出，因此需要 wifi Shield ，然後用 Mbed Shield 來代替麵包板讓 LPC1768 闕發板與 3-Axio Digital Gryo 丶 3-Axio Digital Accelerometer 及  wifi Shield 連結，這時候接出來的圖如下

![土製飛行器sensor](http://i.imgur.com/5hYrd0V.jpg)

突然一個想法閃過，加工之後，下圖就是我們的「霍爾的移動城堡」啦

![霍爾的移動飛機](http://i.imgur.com/AE6ktnp.jpg)

霍爾移動城堡的儀表板：
* Block #1： 3-Axio Digital Gryo ＋ 3-Axio Digital Accelerometer ＋Sensor + WebSocket Client 
 * node.js Websocket server
* Block #2：AutomationJS
 * Frontend
 * Backbone with Virtual DOM

<a href="http://www.youtube.com/watch?feature=player_embedded&v=BoEDQUnGsSw" target="_blank"><img src="http://img.youtube.com/vi/BoEDQUnGsSw/0.jpg" alt="飛行器儀表板" width="240" height="180" border="10" /></a>
