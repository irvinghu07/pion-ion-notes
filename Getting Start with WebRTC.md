# Getting started with WebRTC

Creating a new application based on the WebRTC technologies can be overwhelming if you're unfamiliar with the APIs. In this section we will show how to get started with the various APIs in the WebRTC standard, by explaining a number of common use cases and code snippets for solving those.

# WebRTC APIs

The WebRTC standard covers, on a high level, two different technologies: **media capture devices** and **peer-to-peer connectivity.**

Media capture devices includes video cameras and microphones, but also screen capturing "devices". For cameras and microphones, we use `navigator.mediaDevices.getUserMedia()` to capture `MediaStreams`. For screen recording, we use `navigator.mediaDevices.getDisplayMedia()` instead.

The peer-to-peer connectivity is handled by the `RTCPeerConnection` interface. This is the central point for establishing and controlling the connection between two peers in WebRTC.

# Service Components

> > **	SFU - Selective Forwarding Unit**
>
> https://webrtcglossary.com/sfu/
>
> **MCU - Multipoint Control Unit**
>
> https://webrtcglossary.com/mcu/
>
> **ISLB - Intelligent Server Load Balancing**
>
> ISLB is a dispatch and control server
>
> **BIZ - Business signal server**
>
> BIZ is a signal server which support business logic

---

[root@happy-slab-1 ~]# sudo ~/.acme.sh/acme.sh --issue -d irvinghuhu.ml --standalone -k ec-256 --force
[2020年 11月 06日 星期五 10:14:13 CST] Using CA: https://acme-v02.api.letsencrypt.org/directory
[2020年 11月 06日 星期五 10:14:13 CST] Standalone mode.
[2020年 11月 06日 星期五 10:14:13 CST] Create account key ok.
[2020年 11月 06日 星期五 10:14:13 CST] Registering account: https://acme-v02.api.letsencrypt.org/directory
[2020年 11月 06日 星期五 10:14:14 CST] Registered
[2020年 11月 06日 星期五 10:14:14 CST] ACCOUNT_THUMBPRINT='ESdXMeuPU2EbhzDk9QH3J6FCt5jjzbtz1aBjEg6eMM0'
[2020年 11月 06日 星期五 10:14:14 CST] Creating domain key
[2020年 11月 06日 星期五 10:14:14 CST] The domain key is here: /root/.acme.sh/irvinghuhu.ml_ecc/irvinghuhu.ml.key
[2020年 11月 06日 星期五 10:14:14 CST] Single domain='irvinghuhu.ml'
[2020年 11月 06日 星期五 10:14:14 CST] Getting domain auth token for each domain
[2020年 11月 06日 星期五 10:14:15 CST] Getting webroot for domain='irvinghuhu.ml'
[2020年 11月 06日 星期五 10:14:15 CST] Verifying: irvinghuhu.ml
[2020年 11月 06日 星期五 10:14:15 CST] Standalone mode server
[2020年 11月 06日 星期五 10:14:19 CST] Success
[2020年 11月 06日 星期五 10:14:19 CST] Verify finished, start to sign.
[2020年 11月 06日 星期五 10:14:19 CST] Lets finalize the order.
[2020年 11月 06日 星期五 10:14:19 CST] Le_OrderFinalize='https://acme-v02.api.letsencrypt.org/acme/finalize/101459038/6061115765'
[2020年 11月 06日 星期五 10:14:20 CST] Downloading cert.
[2020年 11月 06日 星期五 10:14:20 CST] Le_LinkCert='https://acme-v02.api.letsencrypt.org/acme/cert/0487b73581122e6c140d06cd319fa9dc3722'
[2020年 11月 06日 星期五 10:14:21 CST] Cert success.
-----BEGIN CERTIFICATE-----
MIIEiDCCA3CgAwIBAgISBIe3NYESLmwUDQbNMZ+p3DciMA0GCSqGSIb3DQEBCwUA
MEoxCzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MSMwIQYDVQQD
ExpMZXQncyBFbmNyeXB0IEF1dGhvcml0eSBYMzAeFw0yMDExMDYwMTE0MjBaFw0y
MTAyMDQwMTE0MjBaMBgxFjAUBgNVBAMTDWlydmluZ2h1aHUubWwwWTATBgcqhkjO
PQIBBggqhkjOPQMBBwNCAATmRr5lvPu4hxSuNt7sw8W8HIzT+mHZ9sON4Nd5VF8q
pJsfa1yKT19djLDea+/9zvFQt4qxdHOUI/61hWTNzZuVo4ICYzCCAl8wDgYDVR0P
AQH/BAQDAgeAMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAMBgNVHRMB
Af8EAjAAMB0GA1UdDgQWBBRV97HdmpaJw0RJz6BcsxXi1wPZAjAfBgNVHSMEGDAW
gBSoSmpjBH3duubRObemRWXv86jsoTBvBggrBgEFBQcBAQRjMGEwLgYIKwYBBQUH
MAGGImh0dHA6Ly9vY3NwLmludC14My5sZXRzZW5jcnlwdC5vcmcwLwYIKwYBBQUH
MAKGI2h0dHA6Ly9jZXJ0LmludC14My5sZXRzZW5jcnlwdC5vcmcvMBgGA1UdEQQR
MA+CDWlydmluZ2h1aHUubWwwTAYDVR0gBEUwQzAIBgZngQwBAgEwNwYLKwYBBAGC
3xMBAQEwKDAmBggrBgEFBQcCARYaaHR0cDovL2Nwcy5sZXRzZW5jcnlwdC5vcmcw
ggEFBgorBgEEAdZ5AgQCBIH2BIHzAPEAdgBElGUusO7Or8RAB9io/ijA2uaCvtjL
MbU/0zOWtbaBqAAAAXWbVU4iAAAEAwBHMEUCIHvO/lVAG7ywD6lY3bHs/JLt3GYR
SE2W1k7N17zTL+8NAiEAxE63g2CUsuvgeuH0+YG/FGKoqbzoKusbZtHKbTDvi+QA
dwD2XJQv0XcwIhRUGAgwlFaO400TGTO/3wwvIAvMTvFk4wAAAXWbVU4MAAAEAwBI
MEYCIQDb/M6XavClMqGFBaM5oC5vJFiywhQLBvjqnraFedVwlAIhAN1yPUeV12CY
R0Ug4DLVo/wMyfb4Jr96no27MxvdJbglMA0GCSqGSIb3DQEBCwUAA4IBAQCEXdV4
ymiG3mdeHAUWBwd1AF9Do382t6g+I+o/C6ONYlCYPnjvLxIZJwjG1sv4Vhi4L7a0
qmgeqFN/IkwrvSTXVSCGFaHj1jN121kvUE8IE131fO0xtjUwOeuFkmuNgAH19KNu
gb5ovHOb9uom8f6VUUFXgY4s5DO3hzJiGjGcxokq35nmPM5H4VIeA4eDL9mBMybw
ajsCEBtyVzvwLCNsZEGPi8k2r8nTmDWWeTVGcwaLeXbNFWef27M6pEGdyaCemM0j
/8ps6nw77Bz7zfqEDAj+imBcmf1RXxUnPxfnrhMDgIPtEEkSbV47BxI7nVVWD8yq
laNYO6x+SUZWCBYJ
-----END CERTIFICATE-----
[2020年 11月 06日 星期五 10:14:21 CST] Your cert is in  /root/.acme.sh/irvinghuhu.ml_ecc/irvinghuhu.ml.cer
[2020年 11月 06日 星期五 10:14:21 CST] Your cert key is in  /root/.acme.sh/irvinghuhu.ml_ecc/irvinghuhu.ml.key
[2020年 11月 06日 星期五 10:14:21 CST] The intermediate CA cert is in  /root/.acme.sh/irvinghuhu.ml_ecc/ca.cer
[2020年 11月 06日 星期五 10:14:21 CST] And the full chain certs is there:  /root/.acme.sh/irvinghuhu.ml_ecc/fullchain.cer

