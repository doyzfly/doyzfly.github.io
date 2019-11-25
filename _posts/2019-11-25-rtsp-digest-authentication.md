---
layout: post
title: RTSP 认证
categories: rtsp
description: RTSP 认证
keywords: rtsp, authentication
---



RTSP 的认证基本认证 ```basic authentication``` 和摘要认证 ```digest authentication``` 。摘要认证是 http 1.1 提出的基本认证的替代方案，其消息经过 MD5 哈希转换因此具有更高的安全性。

## 交互过程
1. DESCRIBE 环节，客户发送与不需要认证一样的请求。服务器端返回401错误，提示未认证，并且在响应报文里面返回 nonce 。
**DESCRIBE Request**

```bash
rtsp://admin:admin123@127.0.0.1:554/cam/playback?channel=2&subtype=0&starttime=2019_08_05_18_13_13 RTSP/1.0
CSeq: 2
User-Agent: ./testRTSPClient (LIVE555 Streaming Media v2019.07.27)
Accept: application/sdp
```


**DESCRIBE Response**

```bash
Received 139 new bytes of response data.
Received a complete DESCRIBE response:
RTSP/1.0 401 Unauthorized
CSeq: 2
WWW-Authenticate: Digest realm="Login to 4M05CC9FAC28B0B", nonce="c5cdebfb791630ad189ac43081c7c01e"
```

2. DESCRIBE 环节，客户端以用户名，密码，nonce，请求方法（OPTIONS/DESCRIBE/SETUP/PLAY），请求的 URI 等信息为基础产生 response 信息进行反馈：
**DESCRIBE Request**

```bash
rtsp://admin:admin123@127.0.0.1:554/cam/playback?channel=2&subtype=0&starttime=2019_08_05_18_13_13 RTSP/1.0
CSeq: 3
Authorization: Digest username="admin", realm="Login to 4M05CC9FAC28B0B", nonce="c5cdebfb791630ad189ac43081c7c01e", uri="rtsp://admin:admin123@127.0.0.1:554/cam/playback?channel=2&subtype=0&starttime=2019_08_05_18_13_13", response="5e607d3405777309839c3d4fe4509e4e"
User-Agent: ./testRTSPClient (LIVE555 Streaming Media v2019.07.27)
Accept: application/sdp
```


**DESCRIBE Response 认证通过**

```bash
RTSP/1.0 200 OK
CSeq: 3
x-Accept-Dynamic-Rate: 1
Content-Base: rtsp://admin:admin123@127.0.0.1:554/cam/playback?channel=2&subtype=0&starttime=2019_08_05_18_13_13/
Cache-Control: must-revalidate
Content-Length: 409
Content-Type: application/sdp
.......

```

**DESCRIBE Response 认证不通过**

```bash
RTSP/1.0 401 Unauthorized
CSeq: 2
WWW-Authenticate: Digest realm="Login to 4M05CC9FAC28B0B", nonce="f015500fc594469c59837dcdca716b13"
```

**digest/response 计算方法**

```golang
    hs1 := md5hash(username + ":" + realm + ":" + password)
    hs2 := md5hash(method + ":" + requestUri)
    response := md5hash(hs1 + ":" + nonce + ":" + hs2)
```

- username：用户名
- password： 密码
- realm： 通常一个 server 对应一个 realm
- method：请求方法（OPTIONS/DESCRIBE/SETUP/PLAY）
- requestUri： 请求的 uri
- nonce： 随机字符串，通常一个 session 对应一个 nonce



3. SETUP, PLAY 阶段用相同的方式进行认证



