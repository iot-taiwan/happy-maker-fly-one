razyflie 2.0 - Spare 7x16 mm coreless DC motor with connector 這款馬達，這款馬達無法直接使用，Seeed Studio 的工程師協助我們，除了畫電路圖給我們，還幫我們焊了第一個測試用馬達，電路圖及測試馬達如下圖：

![馬達電路圖](http://i.imgur.com/QeWPaHT.jpg)

![測試馬達](http://i.imgur.com/gmDEXnM.jpg)

## 於 ARM mbed 上控制直流有刷馬達
有了馬達後，農場的伙伴們立即著手馬達程式設計，直流有刷馬達好處在為控制速度方面比較簡單，可以在不受電源頻率的限制下，使用電壓大小控制轉速。ARM mbed 使用 PwmOut Pin 腳來控制直流有刷馬達。

## 腳位接法
由於 LPC1768 有電路保護，因此馬達的供應電源需要再額外供應。

| Motor pin | Mbed pin   |
| --------- | ---------- |
| 1 - Pwm   | P21 - SCL  |
| 3 - Vcc   | Vout - 3.3V|
| 4 - GND   | GND        |

## 程式碼
開始撰寫 ARM mbed 程式碼。首先，必須引入 mbed.h 標頭檔，接著將連接腳位（這邊要特別注意馬達的腳位宣告要使用 PwmOut ）。並且我們利用 LED 燈來表示轉速的變化。

```
#include "mbed.h"

DigitalOut ledForPower1(LED1);
DigitalOut ledForPower2(LED2);
DigitalOut ledForPower3(LED3);
DigitalOut ledForPower4(LED4);
PwmOut motor(p21);

int main() {
        
        motor=0;
        wait(2);
        
        motor=0.4;
        ledForPower1=1;
        wait(2);
        
        motor=0.6;
        ledForPower2=1;
        wait(2);
        
        motor=0.8;
        ledForPower3=1;
        wait(2);
        
        motor=1;
        ledForPower4=1;
    
}

```
再來就是馬達加速的演算法，我們使用的演算法是「和室關門法」，有了程式賦予馬達靈魂，我們的飛機即將起航。
馬達測試：

<a href="http://www.youtube.com/watch?feature=player_embedded&v=jnIPDK8h8t4" target="_blank"><img src="http://img.youtube.com/vi/jnIPDK8h8t4/0.jpg" alt="馬達測試" width="240" height="180" border="10" /></a>

在我們要去焊四個馬達的電路時 Seeed Studio 的工程師建議我們使用某一塊板子，這樣除了不需要焊的那麼辛苦，體積也小了很多，我們查了一下，它是 Grove - Mini Fan 這款產品的零件之一，整組的零件內容如下：

![Grove - Mini Fan](http://i.imgur.com/OfMPGUC.jpg)

很快的在 Seeed Studio 的協助下我們領到了四份 Grove - Mini Fan ，於是我們立即完成了四個馬達的加工，如下圖：

# 電源
電源的部份我們使用了 Crazyflie 2.0 - Spare 240mAh LiPo battery 這款電源

![Crazyflie 2.0 電源](http://i.imgur.com/oPAprbE.jpg)

# 起航
馬達準備就緒後，第一版我們決定暫時先使用 Crazyflie 2.0 的翅膀，等回台灣後再用 3D 列印制作我們自己的「手作」土製飛行器的翅膀。經過加工後，第一版「手作」土製飛行器完成，如下圖：

![第一版「手作」土製飛行器](http://i.imgur.com/P5UVVLo.jpg)

「手作」土製飛行器起航：

<a href="http://www.youtube.com/watch?feature=player_embedded&v=zCPqoqeyu4M" target="_blank"><img src="http://img.youtube.com/vi/zCPqoqeyu4M/0.jpg" alt="飛行器起航" width="240" height="180" border="10" /></a>
