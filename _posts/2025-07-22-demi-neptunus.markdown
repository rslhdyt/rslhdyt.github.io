---
tags: ["story"]
date: 2025-07-22 00:00:00 +0000
description: Sebuah cerita ringan dari balik layar dunia software engineer, tentang betapa seriusnya urusan menamai project. Dari mitologi Yunani hingga sistem autentikasi berkepala tiga, tulisan ini membuktikan bahwa “nama adalah doa” dan kadang doanya terlalu terkabul 😀
published: true
collection: posts
category: blog
id: 2388e9d5-00d5-8077-a202-ce38cfb511fe
title: Demi Neptunus!!!
created_time: 2025-07-22T13:58:00.000Z
cover: 
icon: 
last_edited_time: 2025-07-22T14:07:00.000Z
archived: false
created_by_object: user
created_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
last_edited_by_object: user
last_edited_by_id: 6b2c6a42-5dc5-4108-b726-4c02437b814d
---

![](/assets/images/posts/4c9912a9-f982-4009-a52e-165842c054f5-King_Neptune_Pose.webp)
<em>[https://spongebob.fandom.com/id/wiki/Raja_Neptunus](https://spongebob.fandom.com/id/wiki/Raja_Neptunus)</em>

Ini adalah blog pertama saya yang ditulis dalam bahasa Indonesia. Saya tidak akan membahas hal-hal teknis secara langsung. Tapi tenang saja, masih ada hubungannya dengan profesi saya sebagai software engineer. Karena walau sedang tidak menulis tentang ruby, docker, atau AI, tulisan ini tetap berada dalam *universe* yang sama, *universe* developer yang penuh keanehan.

Dalam kebudayaan software engineer, kita mengenal pepatah sakral dari Martin Fowler yang katanya cuma ada [dua hal yang benar-benar sulit](https://martinfowler.com/bliki/TwoHardThings.html) dalam *universe* ini: *cache invalidation* dan *naming things*. Saya ingin bercerita soal poin nomor dua.

Di kantor saya yang sekarang, ada tradisi unik: hampir semua project baru dinamai berdasarkan tokoh atau konsep dari mitologi Yunani. Beberapa contohnya: Athena, Elysium, Phoenix dan Lexicon. Keren, kan? Saya pikir, di balik setiap penamaan ini, tersimpan niat dan doa yang tak kalah keren. Seperti orang Indonesia bilang, "Nama adalah doa." Dan itu betul, doanya bisa baik, atau justru sesuai kenyataan, suka atau tidak.

![](/assets/images/posts/83002294-1871-4e02-a3d6-285d737cf3ad-Cerberus_Hell_Hound.jpg)
<em>[https://www.ancient-origins.net/myths-legends-europe/cerberus-legendary-hell-hound-underworld-003142](https://www.ancient-origins.net/myths-legends-europe/cerberus-legendary-hell-hound-underworld-003142)</em>

Baru-baru ini saya ditugaskan untuk melakukan assessment teknis terhadap salah satu service bernama Cerberus. Kalau kamu pernah baca mitologi Yunani, kamu mungkin tahu siapa Cerberus. Ya, anjing berkepala tiga yang menjaga gerbang dunia bawah tanah agar tidak ada yang kabur dari neraka. Nah, di konteks kerjaan kami, Cerberus adalah authentication service yang bertugas menjaga sistem dari user tidak dikenal. Dengan kata lain, dia penjaga gerbang, *literally*. Nama yang sesuai, bukan?

Singkat cerita, muncul inisiatif untuk menggantikan Cerberus dengan sistem open source identity and access management (IAM). Secara teori, sistem baru ini lebih scalable, modern, dan tentunya enterprise ready. Beberapa aplikasi baru sudah mengadopsi sistem ini. Tapi ya, kamu tahu sendiri bagaimana rasanya mengintegrasikan sesuatu yang baru ke dalam sistem *legacy* yang sudah menempel erat seperti hubungan Ibu dan Anak.

Karena integrasi penuh belum bisa dilakukan tanpa risiko besar, akhirnya kami mengambil keputusan bijak khas dunia nyata: biarkan dua-duanya hidup berdampingan. Alhasil, user nantinya bisa login lewat sistem lama atau sistem baru. Alias: kita punya dua mekanisme autentikasi yang berjalan paralel. Alias DUA KEPALA.

Saya iseng berkelakar ke rekan saya, Tie, "Tie, sepertinya project Cerberus sudah mulai menjiwai namanya. Sekarang udah punya dua kepala. Tinggal cari satu lagi buat lengkapin..." Dengan cepat dia menimpali, "Kalau hitung portal aplikasi juga, bisa dibilang Cerberus kita udah punya tiga kepala loh." Dan di situlah saya sadar kita benar-benar telah menciptakan Cerberus yang sesungguhnya.

Demi Neptunus!!! (eh... Neptunus memang bukan dewa Yunani, tapi Roman, sepupu Poseidon. Tapi sudahlah...)


