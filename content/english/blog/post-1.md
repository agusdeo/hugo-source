---
title: "Mengatasi Editor Website Odoo loading terus"
meta_title: ""
description: "this is meta description"
date: 2022-04-04T05:00:00Z
image: "/images/image-placeholder.png"
categories: ["Application", "Data"]
author: "John Doe"
tags: ["nextjs", "tailwind"]
draft: false
---

## � Fix Odoo + aaPanel: Domain WWW, HTTPS, dan Mixed Content (Real Case)

### � Opening (Hook)

Website sudah pakai HTTPS, redirect sudah benar…
tapi saat buka editor Odoo malah blank + error **Mixed Content**?

Gue ngalamin ini langsung — dan ternyata problemnya bukan di DNS �

---

### � Middle (Authority)

Stack yang dipakai:

* Odoo
* aaPanel (reverse proxy)
* Domain + SSL (Let's Encrypt)

Masalah muncul setelah:

* pindah dari non-www → www
* setup redirect domain

---

### ❌ Problem Utama

Error di browser:

> Mixed Content: halaman HTTPS tapi request ke HTTP

Efeknya:

* Editor Odoo tidak bisa dibuka ❌
* Iframe ke-block browser ❌
* Website tampil tapi tidak fully functional ❌

---

### � Akar Masalah

Odoo **tidak tahu request itu HTTPS**

Karena:

* reverse proxy tidak mengirim header:

  ```
  X-Forwarded-Proto
  ```

� Jadi Odoo tetap generate URL:

```
http://...
```

---

### ✅ Solusi (Fix Utama)

Masuk ke aaPanel → Website → **Proxy**

Edit config:

```nginx
location ^~ /
{
    proxy_pass http://127.0.0.1:8069;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # � FIX UTAMA
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header X-Forwarded-Host $host;
}
```

---

### ⚙️ Tambahan (Best Practice)

Di Odoo (`odoo.conf`):

```
proxy_mode = True
```

System Parameter:

```
web.base.url = https://www.batamtour.id
web.base.url.freeze = True
```

---

### � Hasil Setelah Fix

* ✅ Tidak ada lagi Mixed Content
* ✅ Editor Odoo bisa dibuka
* ✅ Semua asset load via HTTPS
* ✅ Domain www bekerja dengan benar

---

### � Insight Penting

> Masalah bukan di SSL atau DNS
> Tapi di komunikasi antara proxy dan Odoo

---

### � Closing (Refleksi)

Kadang problem paling tricky bukan di yang kelihatan (DNS/SSL),
tapi di layer kecil kayak header proxy.

Dan 1 baris ini bisa bikin semuanya error:

```
proxy_set_header X-Forwarded-Proto https;
```

---

Kalau kamu pakai Odoo + reverse proxy, ini WAJIB.

---

### � Hook Lanjutan

Next: gue bakal share setup Odoo production yang proper (biar gak kejadian kayak gini lagi �)
