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
