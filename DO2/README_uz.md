# Linux Network (Linux tarmog'i)

Virtual mashinalarda Linux tarmoqlarini sozlash.

💡 [Bu yerni bosing](https://new.oprosso.net/p/4cb31ec3f47a4596bc758ea1861fb624) **va loyiha haqida fikr-mulohazangizni qoldiring**. So'rovnoma anonim va jamoamizga o'quv tajribangizni yaxshilashda yordam beradi. So'rovnomani loyihadan so'ng darhol to'ldirishni tavsiya qilamiz.

## Mundarija

1. [I bob](#i-bob)
2. [II bob](#ii-bob) \
   2.1. [TCP/IP protokollar steki](#tcpip-protokollar-steki) \
   2.2. [Manzillash](#manzillash) \
   2.3. [Marshrutlash](#marshrutlash)
3. [III bob](#iii-bob) \
   3.1. [ipcalc utilitasi](#1-qism-ipcalc-utilitasi) \
   3.2. [Ikki mashina o'rtasida statik marshrutlash](#2-qism-ikki-mashina-orasida-statik-marshrutlash) \
   3.3. [iperf3 utilitasi](#3-qism-iperf3-utilitasi) \
   3.4. [Tarmoq firewall'i](#4-qism-tarmoq-firewalli) \
   3.5. [Statik tarmoq marshrutlash](#5-qism-statik-tarmoq-marshrutlash) \
   3.6. [DHCP yordamida dinamik IP sozlash](#6-qism-dhcp-yordamida-dinamik-ip-sozlash) \
   3.7. [NAT](#7-qism-nat) \
   3.8. [Bonus. SSH tunnellariga kirish](#8-qism-bonus-ssh-tunnellariga-kirish)
4. [IV bob](#iv-bob)

## Ko'rsatmalar

«School 21 (21-Maktab)»da qanday o'qish kerak:

- Bu yerda erkinligi katta noyob o'quv tajribasi sizni kutmoqda. Sizga topshiriq beriladi va uni yechish yo'lini o'zingiz topasiz: Internet yoki GigaChat kabi AI vositalaridan — sizga qaysi biri qulay bo'lsa, o'shandan foydalanishingiz mumkin. Faqat axborot sifatiga e'tiborli bo'ling: tekshiring, tanqidiy fikrlang, tahlil qiling va solishtiring.
- Peer-to-peer (P2P) o'qish — tengdoshlar o'rtasida bilim va tajriba almashinuvi bo'lib, unda har kim ham ustoz, ham o'quvchi bo'ladi. Bu yondashuv bir-biringizdan o'rganib, materialni chuqurroq tushunishga yordam beradi.
- Yordam so'rashdan tortinmang: atrofingizda bu yo'lni birinchi marta bosib o'tayotgan tengdoshlaringiz bor. O'z tajribangiz va g'oyalaringiz bilan o'rtoqlashing. Hamjamiyatning so'nggi e'lonlaridan xabardor bo'lish uchun Rocket.Chat'ga qo'shiling.
- Birovning yechimini ko'chirib olsangiz, o'qishingiz ma'nosiz bo'lib qoladi. Boshqalardan yordam olganingizda, yechimning «nima uchun», «qanday» va «nima maqsadda» ekanini to'liq tushunib olganingizga ishonch hosil qiling. Xato qilishdan qo'rqmang.
- Topshiriq imkonsizdek tuyulyaptimi? Tanaffus qiling, toza havodan nafas oling va fikringizni tozalang — bu ko'pchilikka yordam bergan. Ehtimol, shundan keyin yechim o'zi keladi.
- O'qish jarayoni natija kabi muhim. Gap faqat topshiriqni bajarishda emas — uni QANDAY yechishni tushunishda.

Loyiha bilan qanday ishlash kerak:

- Boshlashdan oldin loyihani GitLab'dan o'sha nomdagi repozitoriyga klon qiling.
- Barcha fayllar klon qilingan repozitoriyning _src/_ papkasi ichida yaratilishi kerak.
- Loyihani klon qilgandan so'ng _develop_ branch'ini yarating va barcha ishni shu yerda bajaring. Keyin _develop_ branch'ini GitLab'ga push qiling.
- Katalogingizda topshiriqlarda ko'rsatilganlardan boshqa fayllar bo'lmasligi kerak.

## I bob

![linux_network](misc/images/linux_network.png)

Yer sayyorasi, Seb'ning jazz klubi, hozirgi kun.

\> *Barda yangi jazz guruhi chalayapti. Ularning jazzi siz o'rganganingizdan sal energiyaliroq, lekin iste'dodli ekanliklariga shubha yo'q.*

— Sebastian, bir haftadan beri ofisda stol ortida o'tiribsan. Linux'dan foydalanishni o'rgandim, deb o'ylaysanmi? Lekin hafta o'rtasida meni chaqirganingga qaraganda, javobni allaqachon bilaman shekilli...

— Asta-sekin o'rganyapman, lekin o'zim xohlagandek tez emas.

— Ertaga ishga borishga tayyormisan?

— Tushunmayapman, shunchaki tushunmayapman, do'stim. Menga tarmoq sozlash bilan shug'ullan, deyishdi. Lekin bular men uchun shunchaki so'zlar. Sysadmin ishiga kirgan o'sha ahmoq bola bo'lgan yosh o'zim bilan uchrashib, uni bu ishdan qaytargim, hamma narsani tushuntirgim keladi, lekin bunisi imkonsiz. Nima qilay, do'stim?

— Qani, umidsizlikka tushma. Tarmoqlarni sozlash unchalik yomon emas. Senga mamnuniyat bilan aytib beraman, faqat bitta savolimga javob ber: otang seni nega birinchi navbatda sysadmin qilib ishga qo'ydi? Bu axir uning bari, nega bu yerda emas? Bu osonroq ish bo'lardi.

— Kim biladi, chol nima o'ylayotganini. Mustaqil bo'lish, ongingni kengaytirish haqida nimadir deydi...

— Unda ongingni kengaytiramiz. Noutbukingni ol, virtual mashinani yoq, nima nimaligini ko'rsataman.

\> *Odatdagi guruh yangisining o'rnini egallaydi, musiqa sekinlashadi, ofitsiant esa buyurtmangizni hali ham olib kelmadi.*

\> *Sebastian virtual mashinani yoqishga ikkilanayotgan paytda, siz Linux'dagi tarmoqlar haqida ba'zi asosiy ma'lumotlarni aytib berishga qaror qilasiz.*


## II bob

### TCP/IP protokollar steki

Tarmoq nima? Tarmoq — kamida 2 ta kompyuterning qandaydir aloqa liniyalari yoki murakkabroq hollarda tarmoq uskunalari orqali ulanishi. Ular o'rtasida ma'lumotlar ma'lum qoidalar bo'yicha almashinadi, bu qoidalarni esa **TCP/IP** protokollar steki «dikta qiladi».

TCP/IP — Transmission Control Protocol/Internet Protocol degan ma'noni bildiradi. Sodda qilib aytganda, bu turli sathlardagi aloqa protokollari to'plami (har bir sath o'z qo'shnisi bilan muloqot qiladi, ya'ni «ulanadi», shu sababli «stek» deb ataladi), ularga muvofiq tarmoqda ma'lumot almashiladi.
Demak, **TCP/IP** protokollar steki — qoidalar to'plamlarining to'plami :) Bu yerda o'rinli savol tug'iladi: nega bunchalik ko'p protokol kerak? Hammasini bitta protokol orqali almashib bo'lmaydimi?

Gap shundaki, har bir protokol unga ajratilgan qoidalarnigina qat'iy tavsiflaydi. Bundan tashqari, protokollar funksionallik sathlariga bo'lingan, bu esa tarmoq uskunalari va dasturiy ta'minotga o'z doirasidagi vazifalarni ancha sodda, aniq bajarish imkonini beradi.
Protokollar to'plamini sathlarga bo'lish uchun 1978 yilda **OSI** modeli (Open Systems Interconnection Basic Reference Model — ochiq tizimlarning o'zaro aloqasi bazaviy etalon modeli) ishlab chiqilgan.
**OSI** modeli yetti xil sathdan iborat. Har bir sath aloqa tizimlari ishining ma'lum bir sohasiga javob beradi, qo'shni sathlarga bog'liq emas — faqat ma'lum xizmatlarni taqdim etadi. Har bir sath o'z vazifasini protokol deb ataladigan qoidalar to'plamiga muvofiq bajaradi.

### Manzillash

**TCP/IP** protokollar steki asosidagi tarmoqda har bir host'ning (tarmoqqa ulangan kompyuter yoki qurilma) IP manzili bor. IP manzil — 32 bitli son. U odatda nuqtali o'nlik yozuvda, nuqtalar bilan ajratilgan, har biri 0 dan 255 gacha bo'lgan to'rtta o'nlik sondan iborat ko'rinishda ifodalanadi, masalan, *192.168.0.1*.
Umuman olganda, IP manzil ikki qismga bo'linadi: tarmoq (subnet) manzili va host manzili:

![subnetwork_mask](misc/images/subnetwork_mask.png)

Rasmda ko'rib turganingizdek, tarmoq (network) va subnet degan tushunchalar bor.
Bu so'zlarning ma'nosidan ko'rinib turibdiki, IP manzillar tarmoqlarga, tarmoqlar esa subnet mask yordamida subnet'larga bo'linadi
(aniqrog'i: host manzilini subnet'larga bo'lish mumkin).

Host manzilidan tashqari, **TCP/IP** tarmog'ida port degan tushuncha ham bor. Port — qandaydir tizim resursining raqamli tavsifi.
Port tarmoq host'ida ishlayotgan ilovaga boshqa tarmoq host'larida ishlayotgan ilovalar (shu jumladan, o'sha host'dagi boshqa ilovalar) bilan aloqa qilish uchun beriladi. Dasturiy ta'minot nuqtai nazaridan port — OS yadrosi orqali xizmat boshqaradigan ma'lumotlar buferiga bog'langan identifikator deb qarash mumkin.

IP protokoli protokollar ierarxiyasida **TCP** va **UDP** ostida joylashgan va tarmoqda axborotni uzatish hamda marshrutlash uchun javob beradi.
Buning uchun IP har bir axborot bo'lagini (**TCP** yoki **UDP** paketi) boshqa paketga — manba, manzil va marshrut haqida sarlavha saqlaydigan IP paket yoki IP datagramma'ga o'raydi.

Real hayot bilan o'xshatish uchun: **TCP/IP** tarmog'i — shahar. Ko'chalar va tor ko'chalarning nomlari — tarmoqlar va subnet'lar. Bino raqamlari — host manzillari.
Binolarda ofis/kvartira raqamlari — portlar. Aniqrog'i, portlar — oluvchilar (xizmatlar) o'z xatlarini kutib turgan pochta qutilaridir. Shunga ko'ra, 1, 2 va hokazo ofis port raqamlari odatda imtiyozli bo'lgan direktorlar va rahbarlarga beriladi, oddiy xodimlar esa kattaroq raqamli ofislarni oladi. Yozishmalarni jo'natish va yetkazish uchun axborot konvertlarga (ip-paketlarga) joylanadi,
konvertlarda jo'natuvchining manzili (IP va port) hamda oluvchining manzili (IP va port) bo'ladi.

Shuni ta'kidlash kerakki, IP protokolida port tushunchasi yo'q, portlarni talqin qilish **TCP** va **UDP** zimmasida; xuddi shunday, **TCP** va **UDP** IP manzillarni qayta ishlamaydi.

### Marshrutlash

![network_route](misc/images/network_route.png)

Savol tug'ilishi mumkin: bir kompyuter boshqasiga qanday ulanadi? U paketlarni qayerga jo'natishni qayerdan biladi?

Bu masalani hal qilish uchun tarmoqlar gateway'lar (marshrutizatorlar, router'lar) bilan bog'lanadi.
Gateway — host bilan bir xil, lekin ikki yoki undan ortiq tarmoqqa ulanishi bor, tarmoqlar o'rtasida axborot uzata oladi va paketlarni boshqa tarmoqqa jo'nata oladi.
Rasmda gateway'lar — ananas va papaya, ularning har biri turli tarmoqlarga ulangan 2 tadan interfeysga ega.

IP paketlar marshrutini aniqlash uchun manzilning tarmoq qismidan (subnet mask) foydalanadi.
Marshrutni aniqlash uchun tarmoqdagi har bir kompyuterda tarmoqlar va shu tarmoqlar uchun gateway'lar ro'yxatini saqlaydigan marshrutlash jadvali bor.
IP o'tib ketayotgan paketning manzil tarmoq qismini «o'qiydi» va marshrutlash jadvalida shu tarmoq uchun yozuv bo'lsa, paketni mos gateway'ga jo'natadi.

Linux'da operatsion tizim yadrosi marshrutlash jadvalini */proc/net/route* faylida saqlaydi.
Joriy marshrutlash jadvalini `netstat -rn` (r — marshrutlash jadvali, n — IP'larni nomlarga o'girmaslik), `route` yoki `ip r` buyruqlari bilan ko'rish mumkin.

Mana eggplant host'i uchun marshrutlash jadvaliga misol:
```
[root@eggplant ~]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
128.17.75.0      128.17.75.20   255.255.255.0   UN        1500 0          0 eth0
default          128.17.75.98   0.0.0.0         UGN       1500 0          0 eth0
127.0.0.1        127.0.0.1      255.0.0.0       UH        3584 0          0 lo
128.17.75.20     127.0.0.1      255.255.255.0   UH        3584 0          0 lo
```

Ustunlarning ma'nosi:
- Destination — manzil tarmoqlarining (host'larning) manzillari. Tarmoq ko'rsatilgan bo'lsa, manzil odatda nol bilan tugaydi;
- Gateway — birinchi ustunda ko'rsatilgan host/tarmoq uchun gateway manzili; uchinchi ustun — bu marshrut ishlaydigan subnet mask;
- Flags — manzil haqida ma'lumot (U — marshrut ishlayapti, N — tarmoq uchun marshrut, H — host uchun marshrut va hokazo);
- MSS — bir vaqtning o'zida yuborilishi mumkin bo'lgan baytlar soni;
- Window — tasdiq olinguncha yuborilishi mumkin bo'lgan frame'lar soni;
- irtt — marshrutdan foydalanish statistikasi;
- Iface — marshrut uchun ishlatiladigan tarmoq interfeysini ko'rsatadi (eth0, eth1 va hokazo).

\> *Avvalgi safardagidek, yanada ko'proq foydali ma'lumotni materials papkasiga saqlab qo'yasiz.*


## III bob

Ish natijasi sifatida bajarilgan topshiriqlar bilan hisobot taqdim etishingiz kerak. Topshiriqning har bir qismida u bajarilgandan so'ng hisobotga nima qo'shish kerakligi tasvirlangan. Bu savollarga javoblar, skrinshotlar va hokazo bo'lishi mumkin.
- .md kengaytmali hisobot repozitoriyga, src papkasiga yuklanishi kerak;
- Topshiriqning barcha qismlari hisobotda ikkinchi darajali sarlavhalar (level 2 heading) sifatida ajratilishi kerak;
- Topshiriqning bir qismi doirasida hisobotga qo'shiladigan hamma narsa ro'yxat ko'rinishida bo'lishi kerak;
- Hisobotdagi har bir skrinshotga qisqacha izoh yozilishi kerak (skrinshotda nima bor);
- Barcha skrinshotlar faqat ekranning kerakli qismi ko'rinadigan qilib kesilgan bo'lishi kerak;
- Bir skrinshotda bir nechta topshiriq bandini ko'rsatish mumkin, lekin ularning barchasi izohda tasvirlangan bo'lishi shart;
- Topshiriq davomida yaratiladigan barcha virtual mashinalarga **Ubuntu 24.04 Server LTS** o'rnating.
- Zarur bo'lsa, o'zini-o'zi tekshirish va namoyish uchun virtual mashina dump'larini saqlang, lekin ularni repozitoriyga yuklamang.

Utilitalar ro'yxati: `ipcalc`, `ip`, `netplan`, `netstat`, `iperf3`, `iptables`, `ping`, `nmap`, `sysctl`, `tcpdump`, `traceroute`, `systemctl`, `telnet`, `dhclient`, `isc-dhcp-server`, `apache2`.

## 1-qism. **ipcalc** utilitasi

— Xo'sh, tarmoqlarning ajoyib olamiga sho'ng'ishni IP manzillar bilan tanishishdan boshlaymiz. Buning uchun **ipcalc** utilitasidan foydalanamiz.

**== Topshiriq ==**

##### Virtual mashinani ishga tushiring (bundan buyon — ws1)

#### 1.1. Tarmoqlar va maskalar
##### Aniqlang va hisobotga yozing:
##### 1) *192.167.38.54/13* ning tarmoq manzili
##### 2) quyidagi maska konvertatsiyasi: *255.255.255.0* ni prefiks va binar ko'rinishga, */15* ni oddiy va binar ko'rinishga, *11111111.11111111.11111111.11110000* ni oddiy va prefiks ko'rinishga
##### 3) *12.167.38.4* tarmog'ida quyidagi maskalar bilan minimal va maksimal host: */8*, *11111111.11111111.00000000.00000000*, *255.255.254.0* va */4*

#### 1.2. localhost
##### Quyidagi IP'lar orqali localhost'da ishlayotgan ilovaga kirish mumkinligini aniqlang va hisobotga yozing: *194.34.23.100*, *127.0.0.2*, *127.1.0.1*, *128.0.0.1*

#### 1.3. Tarmoq diapazonlari va segmentlari
##### Aniqlang va hisobotga yozing:
##### 1) sanab o'tilgan IP'lardan qaysilari public sifatida, qaysilari faqat private sifatida ishlatilishi mumkin: *10.0.0.45*, *134.43.0.2*, *192.168.4.2*, *172.20.250.4*, *172.0.2.1*, *192.172.0.1*, *172.68.0.2*, *172.16.255.255*, *10.10.10.10*, *192.169.168.1*
##### 2) *10.10.0.0/18* tarmog'i uchun sanab o'tilgan gateway IP manzillaridan qaysilari mumkin: *10.0.0.1*, *10.10.0.2*, *10.10.10.10*, *10.10.100.1*, *10.10.1.255*

## 2-qism. Ikki mashina orasida statik marshrutlash

— Endi ikkita mashinani statik marshrutlash yordamida qanday bog'lashni aniqlaymiz.

**== Topshiriq ==**

##### Ikkita virtual mashinani ishga tushiring (bundan buyon — ws1 va ws2)

##### `ip a` buyrug'i bilan mavjud tarmoq interfeyslarini ko'ring
- Ishlatilgan buyruq va uning natijasi tushirilgan skrinshotni hisobotga qo'shing.
##### Ikkala mashinada ichki tarmoqqa mos keladigan tarmoq interfeysini tavsiflang va quyidagi manzil hamda maskalarni o'rnating: ws1 — *192.168.100.10*, maska */16*, ws2 — *172.24.116.8*, maska */12*
- Har bir mashina uchun o'zgartirilgan *etc/netplan/00-installer-config.yaml* faylining skrinshotini hisobotga qo'shing.
##### Tarmoq xizmatini qayta ishga tushirish uchun `netplan apply` buyrug'ini bajaring
- Ishlatilgan buyruq va uning natijasi tushirilgan skrinshotni hisobotga qo'shing.

#### 2.1. Statik marshrutni qo'lda qo'shish
##### `ip r add` buyrug'i yordamida bir mashinadan ikkinchisiga va orqaga statik marshrut qo'shing.
##### Mashinalar o'rtasidagi ulanishni ping qiling
- Ishlatilgan buyruqlar va ularning natijasi tushirilgan skrinshotni hisobotga qo'shing.

#### 2.2. Statik marshrutni saqlash bilan qo'shish
##### Mashinalarni qayta ishga tushiring
##### */etc/netplan/00-installer-config.yaml* fayli yordamida bir mashinadan ikkinchisiga statik marshrut qo'shing
- O'zgartirilgan */etc/netplan/00-installer-config.yaml* faylining skrinshotlarini hisobotga qo'shing.
##### Mashinalar o'rtasidagi ulanishni ping qiling
- Ishlatilgan buyruq va uning natijasi tushirilgan skrinshotni hisobotga qo'shing.

## 3-qism. **iperf3** utilitasi

— Endi ikkita mashinani bog'ladik. Ayt-chi, mashinalar o'rtasida axborot uzatishda eng muhimi nima?

— Ulanish tezligimi?

— To'g'ri. Uni **iperf3** utilitasi bilan tekshiramiz.

**== Topshiriq ==**

* Bu topshiriqda *2-qism*dagi ws1 va ws2 dan foydalanishingiz kerak.

#### 3.1. Ulanish tezligi
##### Konvertatsiya qiling va natijalarni hisobotga yozing: 8 Mbps ni MB/s ga, 100 MB/s ni Kbps ga, 1 Gbps ni Mbps ga

#### 3.2. **iperf3** utilitasi
##### ws1 va ws2 o'rtasidagi ulanish tezligini o'lchang
- Ishlatilgan buyruqlar va ularning natijasi tushirilgan skrinshotlarni hisobotga qo'shing.

## 4-qism. Tarmoq firewall'i

— Mashinalarni ulagandan so'ng, keyingi vazifamiz — ulanish orqali o'tayotgan axborot oqimini nazorat qilish. Buning uchun firewall'lardan foydalanamiz.

**== Topshiriq ==**

* Bu topshiriqda *2-qism*dagi ws1 va ws2 dan foydalanishingiz kerak.

#### 4.1. **iptables** utilitasi
##### ws1 va ws2 da firewall'ni simulyatsiya qiluvchi */etc/firewall.sh* faylini yarating:
```shell
#!/bin/sh

# "filter" jadvalidagi (default) barcha qoidalarni o'chirish.
iptables -F
iptables -X
```
##### Faylga quyidagi qoidalar ketma-ket qo'shilishi kerak:
##### 1) ws1 da taqiqlovchi qoida boshida, ruxsat beruvchi qoida oxirida yoziladigan strategiyani qo'llang (bu 4 va 5-bandlarga tegishli);
##### 2) ws2 da ruxsat beruvchi qoida boshida, taqiqlovchi qoida oxirida yoziladigan strategiyani qo'llang (bu 4 va 5-bandlarga tegishli);
##### 3) mashinalarda 22-port (ssh) va 80-port (http) uchun kirishni oching;
##### 4) *echo reply*'ni rad eting (mashina ping bo'lmasligi kerak, ya'ni OUTPUT'da blokirovka bo'lishi kerak);
##### 5) *echo reply*'ga ruxsat bering (mashinani ping qilish mumkin bo'lishi kerak);
- Har bir mashina uchun */etc/firewall* faylining skrinshotlarini hisobotga qo'shing.
##### Fayllarni ikkala mashinada `chmod +x /etc/firewall.sh` va `/etc/firewall.sh` buyruqlari bilan ishga tushiring.
- Ikkala faylning ishga tushirilishi skrinshotlarini hisobotga qo'shing;
- Birinchi va ikkinchi fayllarda qo'llanilgan strategiyalar orasidagi farqni hisobotda tasvirlang.

#### 4.2. **nmap** utilitasi
##### **ping** buyrug'i yordamida ping bo'lmaydigan mashinani toping, so'ng **nmap** utilitasi bilan shu mashina host'ining ishlayotganini ko'rsating
*Tekshiruv: nmap natijasida `Host is up` yozuvi bo'lishi kerak*.
- **ping** va **nmap** buyruqlarining chaqirilishi va natijasi skrinshotlarini hisobotga qo'shing.

## 5-qism. Statik tarmoq marshrutlash

— Hozircha biz faqat ikkita mashinani bog'ladik, endi butun tarmoqni statik marshrutlash vaqti keldi.

**== Topshiriq ==**

Tarmoq: \
![part5_network](misc/images/part5_network.png)

##### Beshta virtual mashinani ishga tushiring (3 ta workstation (ws11, ws21, ws22) va 2 ta router (r1, r2))

#### 5.1. Mashina manzillarini sozlash
##### Mashina konfiguratsiyalarini *etc/netplan/00-installer-config.yaml* faylida rasmdagi tarmoqqa muvofiq sozlang.
- Har bir mashina uchun *etc/netplan/00-installer-config.yaml* faylining skrinshotlarini hisobotga qo'shing.
##### Tarmoq xizmatini qayta ishga tushiring. Xatolar bo'lmasa, mashina manzili to'g'riligini `ip -4 a` buyrug'i bilan tekshiring. Shuningdek, ws21 dan ws22 ni ping qiling. Xuddi shunday ws11 dan r1 ni ping qiling.
- Ishlatilgan buyruqlar va ularning natijasi tushirilgan skrinshotlarni hisobotga qo'shing.

#### 5.2. IP forwarding'ni yoqish.
##### IP forwarding'ni yoqish uchun router'larda quyidagi buyruqni bajaring:
`sysctl -w net.ipv4.ip_forward=1`.

*Bu usulda forwarding tizim qayta yuklangandan keyin ishlamaydi.*
- Ishlatilgan buyruq va uning natijasi tushirilgan skrinshotni hisobotga qo'shing.

##### */etc/sysctl.conf* faylini oching va quyidagi qatorni qo'shing:
`net.ipv4.ip_forward = 1`
*Bu usulda IP forwarding doimiy yoqiladi.*
- O'zgartirilgan */etc/sysctl.conf* faylining skrinshotini hisobotga qo'shing.

#### 5.3. Default marshrutni sozlash
Gateway qo'shilgandan keyin `ip r` buyrug'i natijasiga misol:
```
default via 10.10.0.1 dev eth0
10.10.0.0/18 dev eth0 proto kernel scope link src 10.10.0.2
```

##### Workstation'lar uchun default marshrutni (gateway) sozlang. Buning uchun konfiguratsiya faylida router IP'sidan oldin `default` qo'shing
- *etc/netplan/00-installer-config.yaml* faylining skrinshotini hisobotga qo'shing.
##### `ip r` ni chaqiring va marshrutlash jadvaliga marshrut qo'shilganini ko'rsating
- Ishlatilgan buyruq va uning natijasi tushirilgan skrinshotni hisobotga qo'shing.
##### ws11 dan r2 router'ni ping qiling va r2 da ping yetib kelayotganini ko'rsating. Buning uchun `tcpdump -tn -i eth0`
buyrug'idan foydalaning.
- Ishlatilgan buyruqlar va ularning natijasi tushirilgan skrinshotlarni hisobotga qo'shing.

#### 5.4. Statik marshrutlarni qo'shish
##### Konfiguratsiya faylida r1 va r2 ga statik marshrutlar qo'shing. Mana r1 uchun 10.20.0.0/26 ga marshrut misoli:
```shell
# eth1 tarmoq interfeysi tavsifi oxiriga qo'shing:
- to: 10.20.0.0/26
  via: 10.100.0.12
```

- Har bir router uchun o'zgartirilgan *etc/netplan/00-installer-config.yaml* faylining skrinshotlarini hisobotga qo'shing.

##### `ip r` ni chaqiring va ikkala router'dagi marshrutlash jadvallarini ko'rsating. Mana r1 jadvaliga misol:
```
10.100.0.0/16 dev eth1 proto kernel scope link src 10.100.0.11
10.20.0.0/26 via 10.100.0.12 dev eth1
10.10.0.0/18 dev eth0 proto kernel scope link src 10.10.0.1
```
- Ishlatilgan buyruq va uning natijasi tushirilgan skrinshotni hisobotga qo'shing.
##### ws11 da `ip r list 10.10.0.0/[netmask]` va `ip r list 0.0.0.0/0` buyruqlarini bajaring.
- Ishlatilgan buyruqlar va ularning natijasi tushirilgan skrinshotni hisobotga qo'shing.
- Hisobotda 10.10.0.0/\[netmask\] uchun default marshrut bo'lishi mumkin bo'lsa-da, nima uchun 0.0.0.0/0 dan boshqa marshrut tanlanganini tushuntiring.

#### 5.5. Router'lar ro'yxatini tuzish
Gateway qo'shilgandan keyin **traceroute** utilitasi natijasiga misol:
```
1 10.10.0.1 0 ms 1 ms 0 ms
2 10.100.0.12 1 ms 0 ms 1 ms
3 10.20.0.10 12 ms 1 ms 3 ms
```
##### r1 da `tcpdump -tnv -i eth0` dump buyrug'ini ishga tushiring
##### ws11 dan ws21 gacha yo'ldagi router'lar ro'yxatini olish uchun **traceroute** utilitasidan foydalaning
- Ishlatilgan buyruqlar (tcpdump va traceroute) va ularning natijasi tushirilgan skrinshotlarni hisobotga qo'shing;
- r1 dump natijasiga asoslanib, hisobotda **traceroute** yordamida yo'l qurilishi qanday ishlashini tushuntiring.

#### 5.6. Marshrutlashda **ICMP** protokolidan foydalanish
##### r1 da eth0 orqali o'tayotgan tarmoq trafigini
`tcpdump -n -i eth0 icmp` buyrug'i bilan yozib oling.

##### ws11 dan mavjud bo'lmagan IP'ni (masalan, *10.30.0.111*)
`ping -c 1 10.30.0.111` buyrug'i bilan ping qiling.
- Ishlatilgan buyruqlar va ularning natijasi tushirilgan skrinshotni hisobotga qo'shing.

## 6-qism. **DHCP** yordamida dinamik IP sozlash

— Keyingi qadamimiz — o'zingizga tanish **DHCP** xizmati haqida ko'proq bilish.

**== Topshiriq ==**

*Bu topshiriqda 5-qismdagi virtual mashinalardan foydalanishingiz kerak.*

##### r2 uchun */etc/dhcp/dhcpd.conf* faylida **DHCP** xizmatini sozlang:

##### 1) Default router manzili, DNS-server va ichki tarmoq manzilini ko'rsating. Mana r2 uchun fayl misoli:
```shell
subnet 10.100.0.0 netmask 255.255.0.0 {}

subnet 10.20.0.0 netmask 255.255.255.192
{
    range 10.20.0.2 10.20.0.50;
    option routers 10.20.0.1;
    option domain-name-servers 10.20.0.1;
}
```
##### 2) *resolv.conf* fayliga `nameserver 8.8.8.8` deb yozing
- O'zgartirilgan fayllarning skrinshotlarini hisobotga qo'shing.
##### **DHCP** xizmatini `systemctl restart isc-dhcp-server` bilan qayta ishga tushiring. ws21 mashinasini `reboot` bilan qayta yuklang va `ip a` bilan unda manzil paydo bo'lganini ko'rsating. Shuningdek, ws21 dan ws22 ni ping qiling.
- Ishlatilgan buyruqlar va ularning natijasi tushirilgan skrinshotni hisobotga qo'shing.

##### ws11 da MAC manzilni ko'rsating, buning uchun *etc/netplan/00-installer-config.yaml* ga qo'shing:
`macaddress: 10:10:10:10:10:BA`, `dhcp4: true`
- O'zgartirilgan *etc/netplan/00-installer-config.yaml* faylining skrinshotini hisobotga qo'shing.
##### r1 ni r2 bilan bir xil sozlang, lekin manzillarni berishni qat'iy MAC-manzilga (ws11) bog'lang. Xuddi shunday testlarni bajaring
- Ushbu qismni hisobotda r2 dagi kabi tasvirlang.
##### ws21 dan IP manzilni yangilashni so'rang
- Yangilanishdan oldingi va keyingi IP skrinshotlarini hisobotga qo'shing;
- Hisobotda shu bandda qaysi **DHCP** server opsiyalari ishlatilganini tasvirlang.

## 7-qism. **NAT**

— Va nihoyat, tort ustidagi gilos — tarmoq manzillarini trasnlyatsiya qilish (network address translation) mexanizmi haqida aytib beraman.

**== Topshiriq ==**

*Bu topshiriqda 5-qismdagi virtual mashinalardan foydalanishingiz kerak*

##### ws22 va r1 da */etc/apache2/ports.conf* faylidagi `Listen 80` qatorini `Listen 0.0.0.0:80` ga o'zgartiring, ya'ni Apache2 serverini ommaviy (public) qiling
- O'zgartirilgan faylning skrinshotini hisobotga qo'shing
##### ws22 va r1 da Apache veb-serverini `service apache2 start` buyrug'i bilan ishga tushiring
- Ishlatilgan buyruq va uning natijasi tushirilgan skrinshotlarni hisobotga qo'shing.

##### r2 da 4-qismdagi firewall'ga o'xshash tarzda yaratilgan firewall'ga quyidagi qoidalarni qo'shing:
##### 1) filter jadvalidagi qoidalarni o'chirish — `iptables -F`
##### 2) «NAT» jadvalidagi qoidalarni o'chirish — `iptables -F -t nat`
##### 3) barcha marshrutlanadigan paketlarni tashlab yuborish (drop) — `iptables --policy FORWARD DROP`
##### Faylni 4-qismdagidek ishga tushiring
##### ws22 va r1 o'rtasidagi ulanishni `ping` buyrug'i bilan tekshiring
*Bu qoidalar bilan fayl ishga tushirilganda ws22 r1 dan ping bo'lmasligi kerak*
- Ishlatilgan buyruq va uning natijasi tushirilgan skrinshotlarni hisobotga qo'shing.
##### Faylga yana bitta qoida qo'shing:
##### 4) **ICMP** protokolining barcha paketlarini marshrutlashga ruxsat bering
##### Faylni 4-qismdagidek ishga tushiring
##### ws22 va r1 o'rtasidagi ulanishni `ping` buyrug'i bilan tekshiring
*Bu qoidalar bilan fayl ishga tushirilganda ws22 r1 dan ping bo'lishi kerak*
- Ishlatilgan buyruq va uning natijasi tushirilgan skrinshotlarni hisobotga qo'shing.
##### Faylga yana ikkita qoida qo'shing:
##### 5) **SNAT**'ni yoqing, ya'ni r2 ortidagi lokal tarmoqning (5-qismda aniqlanganidek — 10.20.0.0 tarmog'i) barcha lokal IP'larini masquerade qiling
*Maslahat: ichki paketlarni marshrutlash, shuningdek, o'rnatilgan ulanishli tashqi paketlarni marshrutlash haqida o'ylab ko'rish kerak*
##### 6) r2 mashinasining 8080-portida **DNAT**'ni yoqing va ws22 da ishlayotgan Apache veb-serveriga tashqi tarmoq orqali kirishni qo'shing
*Maslahat: ulanishga urinayotganda ws22 va 80-port uchun yangi tcp ulanish bo'lishini hisobga oling*
- O'zgartirilgan faylning skrinshotini hisobotga qo'shing
##### Faylni 4-qismdagidek ishga tushiring
*Sinashdan oldin VirtualBox'da **NAT** tarmoq interfeysini o'chirib qo'yish tavsiya etiladi (uning mavjudligini `ip a` buyrug'i bilan tekshirish mumkin), agar u yoqilgan bo'lsa*
##### **SNAT** uchun TCP ulanishni ws22 dan r1 dagi Apache serveriga `telnet [address] [port]` buyrug'i bilan ulanib tekshiring
##### **DNAT** uchun TCP ulanishni r1 dan ws22 dagi Apache serveriga `telnet` buyrug'i bilan (r2 manzili va 8080-port) ulanib tekshiring
- Ishlatilgan buyruqlar va ularning natijasi tushirilgan skrinshotlarni hisobotga qo'shing.

## 8-qism. Bonus. **SSH tunnellariga** kirish

— Xo'sh, hozircha shu. Boshqa savollaring bormi?

— Ha, yana bitta narsani so'ramoqchi edim. Ishda yurganimda kompaniyamizda qandaydir o'quv loyihalari borligini eshitib qoldim. Tafsilotlarini bilmayman, lekin ularga bir qarab ko'rgim kelardi... Foydali bo'lishi mumkin.

— Ha, bu juda qiziq, lekin bunda senga qanday yordam bera olaman?

— Muammo shundaki, bu loyihalarga yetib borish uchun yopiq tarmoqqa kirish kerak. Bu borada menga biror maslahat bera olasanmi?

— Vooy, bu rostdan ham qiyin masala... Qanchalik yordam bo'lishini bilmayman, lekin sanga **SSH tunnellari** haqida aytib berishim mumkin.

**== Topshiriq ==**

*Bu topshiriqda 5-qismdagi virtual mashinalardan foydalanishingiz kerak.*

##### r2 da 7-qismdagi qoidalar bilan firewall'ni ishga tushiring
##### ws22 da **Apache** veb-serverini faqat localhost'da ishga tushiring (ya'ni */etc/apache2/ports.conf* faylida `Listen 80` qatorini `Listen localhost:80` ga o'zgartiring)
##### ws21 dan ws22 ga *Local TCP forwarding* yordamida ws21 dan ws22 dagi veb-serverga kiring
##### ws11 dan ws22 ga *Remote TCP forwarding* yordamida ws11 dan ws22 dagi veb-serverga kiring
##### Oldingi ikki qadamda ulanish ishlaganini tekshirish uchun ikkinchi terminalga (masalan, Alt + F2 bilan) o'ting va `telnet 127.0.0.1 [local port]` buyrug'ini bajaring.
- Hisobotda shu 4 qadamni bajarish uchun kerak bo'ladigan buyruqlarni tasvirlang va ularning chaqirilishi hamda natijasi skrinshotlarini qo'shing.

## IV bob

— Yordamingiz uchun katta rahmat!

— Arzimaydi! Administratorlik asoslarini eslab olish men uchun ham foydali bo'ldi. Aytgancha, men DevOps'ga o'tishga qaror qildim.

— Voy! Ish topib oldingmi?

— Ha, lekin ko'chib o'tishimga to'g'ri keladi. Shuning uchun keyingi safar hammasini o'zing o'rganishingga to'g'ri keladi.

— Baribir bir kun kelib o'zim boshlashim kerak edi, ehtimol shunisi yaxshidir. Aloqada bo'l, ishlaring qanday ketayotganini aytib tur!

— Sen ham!

\> *Siz bir muddat boshqa mavzularda gaplashasiz, yoqimli musiqa tinglaysiz va ichimliklaringizni tugatasiz, so'ngra xayrlashasiz...*
