---
title: "رد کردن اینترنت ملی بوسیله ssh"
date: 2022-10-07T08:16:29+03:30
---

به دلیل مسدود بودن اینترنت در ایران، در صورت عدم دسترسی به اینترنت آزاد باید 2 عدد وی پی اس برای پرش از دستگاه با IP ایرانی به هر دستگاه دیگری که IP خارجی دارد (بهتر است اروپایی باشد) آماده کنید.
\
این قطعه یک پروکسی Socks5 را در شبکه محلی شما باز می کند:
<br>
<br>
```bash
ssh -C -N -J user@iran user@foreign -D 0.0.0.0:port
```
\
کلاینت برای اتصال در تلفن های همراه:
- Brook - برای کاربران iOS
- Socks Client - برای کاربران اندروید

\
ترجیحا سرور ایرانی که تهیه می‌کنید در دیتاسنتر برج میلاد مستقر باشد.