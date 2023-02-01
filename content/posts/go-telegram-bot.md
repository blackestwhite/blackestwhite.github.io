---
title: "محیط توسعه و دیپلوی من برای ربات تلگرام با گو"
date: 2023-02-01T09:08:38+03:30
draft: false
---

اول فوریه هستش من قصد داشتم از سال میلادی جدید (۲۰۲۳) بیشتر بنویسم ولی همین اولش خورد تو ذوقم سر بی نظمی هام و عدم درس خوندن در طول ترم و خواب و برنامه و زندگیم همه بهم ریخت ولی خب جون سالم به در بردم.
<br>
خب قبل از هرچیزی بگم استک من:

۱- git
۲- github
۳- bash
۴- Go (Golang)
۵- MongoDB
۶- Docker (با docker compose)
۷- Redis (اگر ریت لیمیت و کش نیاز بود)
<br>
<br>
برای ربات از پکیج `https://github.com/tucnak/telebot` استفاده می‌کنم.
نمونه ساده استفاده‌ش:
<br>
<br>

```go

package main

import (
	"log"
	"os"
	"time"

	tele "gopkg.in/telebot.v3"
)

func main() {
	pref := tele.Settings{
		Token:  os.Getenv("TOKEN"),
		Poller: &tele.LongPoller{Timeout: 10 * time.Second},
	}

	b, err := tele.NewBot(pref)
	if err != nil {
		log.Fatal(err)
		return
	}

	b.Handle("/hello", func(c tele.Context) error {
		return c.Send("Hello!")
	})

	b.Start()
}

```
\
<br>
حالا ازونجایی که تو ایران تلگرام فیلتره باید روی سیستمی که داریم توسعه میدیم قابلیت پروکسی رو اد کنیم و روی کلاینت HTTP ربات اضافه کنیم.
\
برای مثال من که از v2ray استفاده می‌کنم طوری کلاینتمو تنظیم کردم که روی پورت ۱۰۸۰۸ یه ساکس فایو باز کنه. پس توی کانفیگ کلاینت رباتم میگم که روی لوکال هاست پورت ۱۰۸۰۸ وصل شه و فقط این در صورتی اجرا میشه که من انتهای دستور اجرا یه کلمه local هم اضافه کنم. (چون داخل دیپلوی سرورمون نیاز به این نداریم)
<br>
<br>

```go

package main

import (
	"log"
	"os"
	"time"

	tele "gopkg.in/telebot.v3"
)

func main() {
	transport := &http.Transport{}
	if len(os.Args) > 1 && os.Args[1] == "local" {
		proxyUrl := "localhost:10808"
		dialer, err := proxy.SOCKS5("tcp", proxyUrl, nil, proxy.Direct)
		if err != nil {
			log.Fatal(err)
			return
		}
		transport.DialContext = func(ctx context.Context, network, address string) (net.Conn, error) {
			return dialer.Dial(network, address)
		}
	}
	cl := &http.Client{Transport: transport}

	pref := tele.Settings{
		Token:  config.TG_BOT_TOKEN,
		Poller: &tele.LongPoller{Timeout: 10 * time.Second},
		Client: cl,
	}

	b, err := tele.NewBot(pref)
	if err != nil {
		log.Fatal(err)
		return
	}

	b.Handle("/hello", func(c tele.Context) error {
		return c.Send("Hello!")
	})

	b.Start()
}

```
\
<br>
فایل `dockerfile` رو اینجوری مینویسم
<br>
<br>

```dockerfile

FROM golang:1.19.0-alpine3.16 as builder

WORKDIR /app

COPY go.* ./
RUN go mod download

COPY . .
RUN go build -o /app/app .

FROM alpine:3.13.1
RUN apk add --no-cache tzdata
ENV TZ=Asia/Tehran
WORKDIR /app
COPY .env .
COPY --from=builder /app/app .

CMD ["/app/app"]

```
\
<br>
فایل `docker-compose.yml` رو اینجوری مینویسم
<br>
<br>

```yaml

version: '3'
services: 
  mongodb:
    image: mongo
    volumes:
      - /vol/project-name/data/mongo-bot-backend:/data/db
    restart: always
  app:
    build: .
    depends_on:
      - mongodb
    links:
      - mongodb
    restart: always

```
\
<br>
خب حالا که همه اینا امادست پوش میکنیم به گیتهاب عزیز.
حالا یه حرکت ریز که کلی زمان واستون سیو میکنه اینه که بیاید یه اسکریپت بنویسید که توی سرورتون هر ۱۰ ثانیه git pull رو اجرا کنه.
بعدش یه git hooks براش تعریف میکنید که بعد از مرج شدن یه دستوری اجرا بشه.
\
ما قراره این puller رو تو فایل `puller.sh` بسازیم.
<br>
<br>

```bash

while true; do
    git pull &> /dev/null
    sleep 10
done

```
\
<br>
حالا نکته اساسی اینه، داخل سرورتون ریپازیتوری رو کلون کنید برید داخلش و فایل `post-merge` رو داخل hooks های ریپازیتوریتون ایجاد کنید.
مسیرش اینطوری میشه:
<br>
<br>

```bash

.git/hooks/post-merge

```
\
<br>
و محتویاتش باید این باشه:
<br>
<br>

```bash

#!/bin/bash
docker compose down
docker compose up -d --build

```
\
<br>
کاری که این اسکریپت میکنه، این هستش که کانتینر های قبلی رو میاره پایین و جدیدا رو بیلد میگیره و میاره بالا.
\
یادتون نره پرمیشن اجرا بدین به اسکریپت ها به وسیله دستور chmod +x با اسم فایل
\
حالا همه چی امادست فقط کافیه دستورای زیر اجرا کنید که پولر ران شه
<br>
<br>

```bash

./puller.sh &
# بهتون یه پروسس آیدی میده که باید بدید به دستور زیر، عدد رو بدید به دیس اون
disown pid

```
\
<br>
حالا میتونید از سرور برید بیرون و از این به بعد هرچی به برنچ اصلیتون پوش کنید اتومات پول میشه بعدش گیت هوکز ران میشه و کانتینر هاتون رو میاره پایین، سپس دوباره بیلد میگیره و میاره بالا.
\
در طی نوشتن این پست فهمیدم، همچین چیزایی رو اگه ویدیو بگیرم بهتره تا اینکه بنویسم... به هر حال موفق باشید.