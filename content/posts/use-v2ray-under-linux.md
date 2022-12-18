---
title: "استفاده از v2ray در لینوکس - کلاینت v2ray"
date: 2022-12-17T01:47:03+03:30
draft: true
---

از حدود دو ماه پیش که با کلاینتای مختلف `v2ray` سر و کله میزدم. بنظرم زیاد جالب نبودن تصمیم گرفتم کلا به صورت خام از خود core و دستی نوشتن کانفیگا ازش استفاده کنم.
<br>
<br>

## نصب v2ray {#id #install-v2ray}
از لینک زیر v2ray-core رو دانلود کنید
<br>
<br>
```bash
https://github.com/v2ray/v2ray-core/releases/
```
\
<br>
از حالت زیپ خارج کنیدش
<br>
<br>
```bash
unzip v2ray-linux-64.zip -d v2ray-linux-64
```
\
<br>
بعد از خارج کردن از حالت زیپ فایل های مربوطه رو به صورت زیر با دستور `mv` جابجا کنید
<br>
<br>
```bash
v2ray -> /usr/local/bin/v2ray
geoip.dat -> /usr/local/share/v2ray/geoip.dat
geosite.dat -> /usr/local/share/v2ray/geosite.dat
config.json -> /usr/local/etc/v2ray/config.json
v2ray.service -> /etc/systemd/system/v2ray.service
v2ray@.service -> /etc/systemd/system/v2ray@.service
```
\
<br>

## فایل کانفیگ
این فقط یه نمونه است که ببینید فایل کانفیگ چجوریه
برای اینکه درست کار کنه باید کانفیگ خودتون رو توی قسمت outbounds اضافه کنید
برای مثال من از پرتکل vmess روی وب سوکت استفاده میکنم.
اگر شما هم همین پرتکل رو استفاده میکنید صرفا مقادیر آدرس، پورت و آیدی(فرمت uuid v4) رو عوض کنید.
\
<br>  

```json
{
    "routing": {
        "domainStrategy": "Asls"
    },
    "inbounds": [
        {
            "sniffing": {
                "enabled": false
            },
            "listen": "127.0.0.1",
            "protocol": "socks",
            "settings": {
                "udp": true,
                "auth": "noauth",
                "userLevel": 8
            },
            "tag": "socks",
            "port": 10808
        }
    ],
    "outbounds": [
        {
            "mux": {
                "enabled": false
            },
            "streamSettings": {
                "network": "ws",
                "wsSettings": {
                    "path": "/",
                    "headers": {
                        "Host": ""
                    }
                },
                "security": "none"
            },
            "protocol": "vmess",
            "settings": {
                "vnext": [
                    {
                        "address": "domain-or-ip.com",
                        "users": [
                            {
                                "id": "uuid-v4",
                                "alterId": 0,
                                "security": "auto",
                                "level": 8
                            }
                        ],
                        "port": 80
                    }
                ]
            }
        }
    ],
    "log": {
        "loglevel": "none"
    },
    "dns": {
        "servers": [
            "8.8.8.8",
            "8.8.4.4"
        ]
    }
}
```
\
<br>

## استفاده از v2ray {#id #use-v2ray}
\
<br>
```bash
# راه اندازی v2ray
sudo systemctl start v2ray

# بررسی وضعیت v2ray
sudo systemctl status v2ray

# فعال کردن v2ray
sudo systemctl enable v2ray
```
\
<br>
اگر کانفیگتون سالم باشه(فیلتر نشده باشه) و درست فایل جیسون رو ادیت زده باشید. روی پورت `10808` یه پروکسی SOCKS5 دارید.
حالا میتونید از این پروکسی توی تلگرام یا کل سیستم استفاده کنید.
مثلا من که سیستمی که دارم باهاش این پست رو مینویسم اوبنتو هستش توی مسیر زیر میتونم پروکسی رو فعال کنم
\
<br>
```bash
# مسیر تنظیم:
Settings > Network > Network Proxy > Manual

Socks Host: localhost
Port: 10808
```
\
<br>
موفق باشید