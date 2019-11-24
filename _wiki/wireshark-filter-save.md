---
layout: wiki
title: Wireshark 保存过滤后的报文
categories: wireshark
description: Wireshark 保存过滤后的报文
keywords: wireshark, filter
---

定位问题或是了解某个协议的时候，经常会用到 Wireshark 抓包，然后进行分析。 Wireshark 抓包通常是指定某个网络接口，抓取这个网络接口的所有流量，这个时候抓的包经常会有一些杂音，需要剔除的，这个时候需要用到 Wireshark 的过滤功能，通常会根据 IP 或是端口来进行过滤。如果需要保存过滤后的报文为单独的文件，以便后续分析使用，可以使用 Wireshark 的 “Export Specified Packets” 功能来保存过滤后的报文。
大体的流程如下：  
1. 启动 Wireshark 抓取某个网口的报文。
2. Wireshark 过滤报文。
3. “Export Specified Packets” 保存过滤后的报文为单独的文件。

## 启动 Wireshark 抓取某个网口的报文
点击 “Capture Options” 进去后选择需要抓取报文的网口，点击 “Start” 开始抓包。

## Wireshark 过滤报文
在 Filter 窗口输入过滤条件，常用的过滤条件有根据 IP 来进行过滤。  
1. 比如要查看一个 WebSocket 的交互过程，使用一个与 ws://echo.websocket.org 服务器交互的 demo 来进行测试并抓包。echo.websocket.org 在本机 DNS 解析成 174.129.224.73。使用以下的过滤语句可以过滤去其他无关的报文。  
    ```bash
    ip.addr == 174.129.224.73
    ```

2. 某个服务器的 Web 服务的端口是9696，需要抓取与 Web 服务(9696)的交互，可以通过端口号来进行过滤。  
    ```bash
    tcp.port == 9696
    ```

    <img src="/images/wiki/wireshark/filter-save01.png">

## “Export Specified Packets” 保存过滤后的报文为单独的文件
File -> Export Specified Packets -> Displayed， 这样可以只保存过滤后报文

<img src="/images/wiki/wireshark/filter-save02.png">
<img src="/images/wiki/wireshark/filter-save03.png">