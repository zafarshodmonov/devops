# Linux Network — to'liq o'quv qo'llanma

> Bu qo'llanma School 21 "Linux Network" loyihasini tushunib bajarish uchun kerak bo'ladigan barcha nazariya va amaliyotni qamrab oladi: IP manzillash, marshrutlash, `netplan`, `iperf3`, `iptables`, DHCP, NAT va SSH tunnellar. Har bir bo'limda **nima**, **nima uchun** va **qanday** degan savollarga javob beriladi.

## Mundarija

1. [Tarmoq nima va TCP/IP, OSI modellari](#1-tarmoq-nima-va-tcpip-osi-modellari)
2. [IP manzillash va subnet maskalar](#2-ip-manzillash-va-subnet-maskalar)
3. [Maxsus diapazonlar: private, loopback va boshqalar](#3-maxsus-diapazonlar-private-loopback-va-boshqalar)
4. [Port, TCP va UDP](#4-port-tcp-va-udp)
5. [Marshrutlash](#5-marshrutlash)
6. [Ubuntu 24.04 da tarmoq sozlash: ip va netplan](#6-ubuntu-2404-da-tarmoq-sozlash-ip-va-netplan)
7. [Diagnostika vositalari: ping, traceroute, tcpdump, nmap](#7-diagnostika-vositalari-ping-traceroute-tcpdump-nmap)
8. [Tezlik birliklari va iperf3](#8-tezlik-birliklari-va-iperf3)
9. [Firewall: iptables](#9-firewall-iptables)
10. [DHCP](#10-dhcp)
11. [NAT: SNAT, MASQUERADE, DNAT](#11-nat-snat-masquerade-dnat)
12. [SSH tunnellar](#12-ssh-tunnellar)
13. [VirtualBox stendini tayyorlash](#13-virtualbox-stendini-tayyorlash)
14. [Xatolarni topish (troubleshooting)](#14-xatolarni-topish-troubleshooting)
15. [Tezkor ma'lumotnoma (cheat sheet)](#15-tezkor-malumotnoma-cheat-sheet)

---

## 1. Tarmoq nima va TCP/IP, OSI modellari

**Tarmoq** — kamida ikkita qurilmaning aloqa kanali (kabel, Wi-Fi, virtual adapter) orqali bog'lanib, ma'lumot almashishi. Almashinuv qoidalari **protokollar** deb ataladi.

### 1.1. Nima uchun sathlar kerak?

Bitta "katta protokol" o'rniga ko'p kichik protokollar ishlatiladi, chunki:

- har bir protokol faqat o'z vazifasi uchun javob beradi (modullilik);
- bir sathni almashtirish boshqasiga ta'sir qilmaydi (masalan, Wi-Fi'dan kabelga o'tganda brauzer o'zgarmaydi);
- uskunalar va dasturlar soddalashadi.

### 1.2. OSI va TCP/IP

| OSI sathi | TCP/IP sathi | Nima qiladi | Misol | Ma'lumot birligi |
|---|---|---|---|---|
| 7. Application | Application | Ilova bilan aloqa | HTTP, SSH, DHCP, DNS | Data |
| 6. Presentation | Application | Kodlash, shifrlash | TLS | Data |
| 5. Session | Application | Sessiyalar | — | Data |
| 4. Transport | Transport | Ilovalar o'rtasida yetkazish, portlar | TCP, UDP | Segment / Datagram |
| 3. Network | Internet | Manzillash va marshrutlash | IP, ICMP | Packet |
| 2. Data Link | Link | Bir tarmoq ichida yetkazish, MAC | Ethernet, ARP | Frame |
| 1. Physical | Link | Signal uzatish | Kabel, radio | Bit |

Loyihada asosan 2–4-sathlar bilan ishlaysiz: **MAC (2)**, **IP va ICMP (3)**, **TCP/UDP va portlar (4)**.

### 1.3. Encapsulation (o'rash)

Ma'lumot yuborilganda har bir sath o'z sarlavhasini (header) qo'shadi:

```
Ilova ma'lumoti:           [ DATA ]
Transport (TCP):         [ TCP | DATA ]
Network (IP):        [ IP | TCP | DATA ]
Link (Ethernet): [ ETH | IP | TCP | DATA | FCS ]
```

Qabul qiluvchi tomonda teskari jarayon (decapsulation) bo'ladi. Shuning uchun `tcpdump` chiqishida bir paketda bir nechta sath ma'lumotini ko'rasiz.

---

## 2. IP manzillash va subnet maskalar

### 2.1. IPv4 manzil

IPv4 manzil — **32 bitli** son. Odamlar uchun 8 bitdan (oktet) to'rt guruhga bo'lib, o'nlik sanoq tizimida yoziladi:

```
11000000.10101000.00000001.00000001   (binar)
   192  .  168   .   1    .   1       (o'nlik)
```

Har oktet 0–255 oralig'ida (chunki 2⁸ = 256 qiymat).

### 2.2. Binar ↔ o'nlik

Oktet bitlarining og'irligi: `128 64 32 16 8 4 2 1`.

Misol: `167` ni binarga o'tkazish:
- 167 − 128 = 39 → 128 biti **1**
- 39 < 64 → 64 biti **0**
- 39 − 32 = 7 → 32 biti **1**
- 7 < 16 → **0**, 7 < 8 → **0**
- 7 − 4 = 3 → **1**, 3 − 2 = 1 → **1**, 1 − 1 = 0 → **1**

Natija: `10100111`.

### 2.3. Subnet mask va prefiks (CIDR)

IP manzil ikki qismdan iborat: **tarmoq qismi** va **host qismi**. Ular o'rtasidagi chegarani **maska** belgilaydi: maskada `1` bitlar — tarmoq qismi, `0` bitlar — host qismi. Maskada 1'lar doim chapdan uzluksiz keladi.

```
Manzil:   192.168.1.10   = 11000000.10101000.00000001.00001010
Maska:    255.255.255.0  = 11111111.11111111.11111111.00000000
                           |---- tarmoq (24 bit) ----|-host (8)|
```

**Prefiks** (CIDR yozuvi) — maskadagi birlar soni: `192.168.1.10/24`.

Maskaning oktet qiymatlari faqat quyidagilardan biri bo'ladi (esda tuting!):

| Birlar soni oktetda | Binar | O'nlik |
|---|---|---|
| 0 | 00000000 | 0 |
| 1 | 10000000 | 128 |
| 2 | 11000000 | 192 |
| 3 | 11100000 | 224 |
| 4 | 11110000 | 240 |
| 5 | 11111000 | 248 |
| 6 | 11111100 | 252 |
| 7 | 11111110 | 254 |
| 8 | 11111111 | 255 |

#### Prefiks → maska

`/15` = 15 ta bir = `8 + 7`: birinchi oktet to'liq (255), ikkinchi oktetda 7 ta bir (254):

```
/15 → 11111111.11111110.00000000.00000000 → 255.254.0.0
```

#### Maska → prefiks

`255.255.255.240` → `255` = 8 bit, 8 bit, 8 bit, `240` = `11110000` = 4 bit → 8+8+8+4 = **/28**.

### 2.4. Tarmoq manzili, broadcast, host diapazoni

Berilgan IP va maska uchun:

- **Network address** = IP **AND** maska (host bitlari hammasi 0).
- **Broadcast address** = host bitlari hammasi 1.
- **Birinchi host** = network + 1.
- **Oxirgi host** = broadcast − 1.
- **Hostlar soni** = 2^(host bitlari) − 2 (network va broadcast manzillar hostga berilmaydi).

#### Misol: `192.167.38.54/13`

1. /13 = 8 + 5 → 2-oktetda 5 ta bir: maska `255.248.0.0`.
2. Chegara 2-oktetda. `167 = 10100111`, maska `248 = 11111000`:
   ```
   10100111
   11111000  AND
   --------
   10100000 = 160
   ```
3. Tarmoq: `192.160.0.0/13`.
4. Broadcast: 2-oktetda host bitlari (oxirgi 3 bit) = 1 → `10100111` = 167 → `192.167.255.255`.
5. Hostlar: `192.160.0.1 – 192.167.255.254`, soni 2¹⁹ − 2 = 524 286.

#### "Sehrli son" usuli (tez hisoblash)

Chegara oktetidagi blok o'lchami = `256 − maska_oktet_qiymati`. Masalan, `/23` → maska `255.255.254.0`, chegara 3-oktetda, blok = 256 − 254 = **2**. Tarmoqlar 3-oktetda 0, 2, 4, ..., 38, 40... dan boshlanadi. IP `12.167.38.4` uchun 38 shu ro'yxatda → tarmoq `12.167.38.0`, broadcast `12.167.39.255`, hostlar `12.167.38.1 – 12.167.39.254`.

#### Misol: `12.167.38.4` turli maskalarda

| Maska | Prefiks | Tarmoq | Min host | Max host |
|---|---|---|---|---|
| /8 | 8 | 12.0.0.0 | 12.0.0.1 | 12.255.255.254 |
| 11111111.11111111.00000000.00000000 | /16 | 12.167.0.0 | 12.167.0.1 | 12.167.255.254 |
| 255.255.254.0 | /23 | 12.167.38.0 | 12.167.38.1 | 12.167.39.254 |
| /4 | 4 | 0.0.0.0 | 0.0.0.1 | 15.255.255.254 |

`/4` da: `12 = 00001100`, birinchi 4 bit `0000` → tarmoq `0.0.0.0`, 4 ta host biti 1 bo'lsa `00001111 = 15`.

### 2.5. `ipcalc` utilitasi

```bash
sudo apt update && sudo apt install ipcalc -y

ipcalc 192.167.38.54/13
ipcalc 192.167.38.54 255.248.0.0
ipcalc 12.167.38.4/23
```

Muhim qatorlar: `Address`, `Netmask`, `Wildcard`, `Network`, `HostMin`, `HostMax`, `Broadcast`, `Hosts/Net`.

> Ipcalc'ning turli versiyalari chiqish formatini bir oz boshqacha ko'rsatadi (Debian/Ubuntu'dagi `ipcalc` va Red Hat'dagi `ipcalc` boshqa dasturlar). Natijani har doim o'zingiz ham hisoblab tekshiring.

### 2.6. Python bilan tekshirish (dasturchi uchun foydali)

`ipaddress` moduli standart kutubxonada bor:

```python
import ipaddress as ip

iface = ip.ip_interface("192.167.38.54/13")
net = iface.network
print(net)                      # 192.160.0.0/13
print(net.netmask)              # 255.248.0.0
print(net.broadcast_address)    # 192.167.255.255
print(net.network_address + 1)  # birinchi host
print(net.broadcast_address - 1)  # oxirgi host
print(net.num_addresses - 2)    # hostlar soni

print(ip.ip_address("172.20.250.4").is_private)   # True
```

> Ehtiyot bo'ling: `list(net.hosts())` ni `/4` kabi katta tarmoqda chaqirmang — 268 milliondan ortiq element xotirani to'ldiradi. Faqat `network_address + 1` va `broadcast_address - 1` dan foydalaning.

**Pseudokod** (qo'lda hisoblash algoritmi):

```
FUNKSIYA hisobla(ip, prefiks):
    mask    = (0xFFFFFFFF << (32 - prefiks)) & 0xFFFFFFFF
    network = ip AND mask
    bcast   = network OR (NOT mask & 0xFFFFFFFF)
    minHost = network + 1
    maxHost = bcast - 1
    soni    = 2^(32 - prefiks) - 2
    QAYTAR network, bcast, minHost, maxHost, soni
```

---

## 3. Maxsus diapazonlar: private, loopback va boshqalar

### 3.1. Private (xususiy) manzillar — RFC 1918

Internetda marshrutlanmaydi, ichki tarmoqlarda erkin ishlatiladi:

| Diapazon | Prefiks | Oraliq |
|---|---|---|
| 10.0.0.0 | /8 | 10.0.0.0 – 10.255.255.255 |
| 172.16.0.0 | /12 | 172.16.0.0 – **172.31.255.255** |
| 192.168.0.0 | /16 | 192.168.0.0 – 192.168.255.255 |

Eng ko'p xato qilinadigan joy — `172.x`: faqat **172.16 – 172.31** private. `172.0.2.1` va `172.68.0.2` — **public**.

Boshqa maxsus diapazonlar:

- `127.0.0.0/8` — loopback (quyida);
- `169.254.0.0/16` — link-local (DHCP topilmasa tizim o'zi oladi);
- `100.64.0.0/10` — CGNAT (provayderlar ishlatadi);
- `224.0.0.0/4` — multicast.

### 3.2. Loopback / localhost

`127.0.0.0/8` — butun blok loopback hisoblanadi: `127.0.0.1`, `127.0.0.2`, `127.1.0.1`, `127.255.255.254` — hammasi **shu kompyuterning o'ziga** yo'naladi va tarmoqqa chiqmaydi. Shu sababli:

| IP | localhost'dagi ilovaga kirish mumkinmi? |
|---|---|
| 194.34.23.100 | Yo'q (oddiy public manzil) |
| 127.0.0.2 | Ha (127.0.0.0/8 ichida) |
| 127.1.0.1 | Ha (127.0.0.0/8 ichida) |
| 128.0.0.1 | Yo'q (127 emas, 128) |

Nuans: ilova faqat `127.0.0.1` ga bog'langan (bind) bo'lsa, `127.0.0.2` orqali javob bermasligi mumkin. Lekin manzil nuqtai nazaridan hammasi loopback.

### 3.3. Gateway manzili qanday bo'lishi kerak

Gateway — host'ning **o'z tarmog'idagi** router manzili. Shartlar:

1. manzil shu tarmoq diapazonida bo'lishi kerak;
2. network manzil ham, broadcast manzil ham bo'lmasligi kerak.

`10.10.0.0/18` uchun: oraliq `10.10.0.0 – 10.10.63.255`, hostlar `10.10.0.1 – 10.10.63.254`. Natija:

| Gateway | Mumkinmi? | Sabab |
|---|---|---|
| 10.0.0.1 | Yo'q | Tarmoqdan tashqarida |
| 10.10.0.2 | Ha | Diapazon ichida |
| 10.10.10.10 | Ha | Diapazon ichida |
| 10.10.100.1 | Yo'q | 3-oktet 100 > 63 |
| 10.10.1.255 | Ha | /18 da broadcast emas (broadcast — 10.10.63.255), oddiy host |

---

## 4. Port, TCP va UDP

**IP manzil** — qaysi **kompyuter**, **port** — shu kompyuterdagi qaysi **ilova (xizmat)**. Port — 16 bitli son (0–65535).

| Diapazon | Nomi | Izoh |
|---|---|---|
| 0–1023 | Well-known | Standart xizmatlar, root huquqi kerak |
| 1024–49151 | Registered | Ro'yxatga olingan ilovalar |
| 49152–65535 | Dynamic/ephemeral | Mijoz tomonidan vaqtincha olinadi |

Muhim portlar: `22` SSH, `53` DNS, `67/68` DHCP (UDP), `80` HTTP, `443` HTTPS.

Ulanish to'rtligi: **(manba IP, manba port, manzil IP, manzil port)**. Veb-sahifa ochganingizda: manba port tasodifiy (masalan, 51234), manzil port 80.

| | TCP | UDP |
|---|---|---|
| Ulanish | Oldindan o'rnatiladi (3-way handshake: SYN → SYN-ACK → ACK) | Yo'q |
| Ishonchlilik | Yo'qolgan paket qayta yuboriladi | Kafolat yo'q |
| Tezlik | Sekinroq | Tezroq |
| Misol | HTTP, SSH | DHCP, DNS, video chaqiruv |

**IP** portni bilmaydi — portlarni **TCP/UDP** sathi talqin qiladi. **ICMP** esa portga ega emas (u servis xabarlari uchun: ping, "unreachable" va h.k.).

---

## 5. Marshrutlash

### 5.1. Asosiy g'oya

Host paket jo'natayotganda bitta savolga javob beradi: **"manzil mening tarmog'imdami?"**

- **Ha** — paket to'g'ridan-to'g'ri qabul qiluvchiga (ARP orqali MAC topiladi);
- **Yo'q** — paket **gateway**'ga (router) yuboriladi, router esa uni keyingi qadamga uzatadi.

```
 ws11 ───── r1 ═════════ r2 ───── ws21
10.10.0.2  .1  .11   .12  .1   10.20.0.10
 (10.10.0.0/18) (10.100.0.0/16)  (10.20.0.0/26)
```

### 5.2. Marshrutlash jadvali

Har bir host va router'da jadval bor. Ko'rish:

```bash
ip r            # eng qulay
ip route show
route -n        # net-tools kerak
netstat -rn     # net-tools kerak
```

Misol:

```
default via 10.10.0.1 dev enp0s3
10.10.0.0/18 dev enp0s3 proto kernel scope link src 10.10.0.2
```

- `10.10.0.0/18 dev enp0s3 ... scope link` — **connected route**: bu tarmoq to'g'ridan-to'g'ri interfeysga ulangan, gateway kerak emas. Interfeysga IP berilganda yadro uni **o'zi** qo'shadi (`proto kernel`).
- `default via 10.10.0.1` — boshqa hamma narsa uchun gateway. `default` = `0.0.0.0/0`.

### 5.3. Longest prefix match

Bir nechta marshrut mos kelsa, **eng uzun prefiksli** (eng aniq) tanlanadi.

```
FUNKSIYA marshrut_tanla(dest):
    mos = []
    HAR BIR yozuv UCHUN jadvalda:
        AGAR dest yozuv.tarmoq ichida BO'LSA:
            mos.qo'sh(yozuv)
    AGAR mos BO'SH BO'LSA: QAYTAR "Network is unreachable"
    QAYTAR mos ichidan eng katta prefiksli yozuv
```

Misol: ws11 da `10.10.0.1` ga paket yuboriladi. Ikki marshrut mos:

- `10.10.0.0/18 dev enp0s3` (prefiks 18)
- `default via 10.10.0.1` (prefiks 0)

18 > 0, shuning uchun to'g'ridan-to'g'ri interfeys ishlatiladi. **Hisobotdagi savolning javobi shu.**

### 5.4. Statik marshrut qo'shish

```bash
sudo ip r add 10.20.0.0/26 via 10.100.0.12      # gateway orqali
sudo ip r add 172.16.0.0/12 dev enp0s8          # to'g'ridan-to'g'ri interfeys orqali (on-link)
sudo ip r del 10.20.0.0/26
ip r get 10.20.0.10                             # qaysi marshrut tanlanishini ko'rsatadi
```

`via` manzili **bevosita ulangan** tarmoqda bo'lishi shart. Aks holda: `Error: Nexthop has invalid gateway`.

`ip r add` bilan qo'shilgan marshrut **vaqtinchalik** — reboot yoki `netplan apply` dan keyin yo'qolishi mumkin. Doimiy qilish uchun netplan'ga yoziladi (6-bo'lim).

### 5.5. Ikki mashina, har xil subnet (2-qism uchun)

ws1 — `192.168.100.10/16` (tarmoq `192.168.0.0/16`), ws2 — `172.24.116.8/12` (tarmoq `172.16.0.0/12`). Ular bir fizik (virtual) kanalda, lekin **mantiqiy** har xil tarmoqda. Har biri boshqasini "o'z tarmog'im emas" deb hisoblaydi va default gateway ham yo'q. Yechim: ikkala tomonda ham boshqa tarmoqqa **on-link** marshrut qo'shish:

```bash
# ws1:
sudo ip r add 172.16.0.0/12 dev enp0s8
# ws2:
sudo ip r add 192.168.0.0/16 dev enp0s8
```

Ikkala tomon ham kerak: so'rov bir tomondan boradi, **javob** esa qaytish marshrutini talab qiladi. Bitta tomonda marshrut bo'lmasa, ping javobsiz qoladi.

### 5.6. IP forwarding

Linux odatda o'ziga mo'ljallanmagan paketni **tashlab yuboradi**. Router bo'lishi uchun forwarding yoqiladi:

```bash
sudo sysctl -w net.ipv4.ip_forward=1     # vaqtinchalik (reboot'gacha)
cat /proc/sys/net/ipv4/ip_forward        # 1 bo'lishi kerak
```

Doimiy qilish: `/etc/sysctl.conf` ga `net.ipv4.ip_forward = 1` qo'shib, `sudo sysctl -p` bajariladi.

### 5.7. Default marshrut

Tashqi dunyo uchun yagona chiqish eshigi. Workstation'larda faqat bitta default bo'lishi kerak. Netplan'da:

```yaml
routes:
  - to: default
    via: 10.10.0.1
```

> Agar mashinada VirtualBox **NAT** adapteri ham yoqilgan bo'lsa (DHCP orqali o'z default marshrutini oladi), ikkita default paydo bo'lib, test natijalarini buzadi. 5–8-qismlarda NAT adapterni o'chiring.

---

## 6. Ubuntu 24.04 da tarmoq sozlash: ip va netplan

### 6.1. Interfeys nomlari

Ubuntu 24.04 VirtualBox'da odatda `enp0s3`, `enp0s8`, `enp0s9`... nomlarini beradi (README'dagi `eth0`/`eth1` — shartli nomlar). Haqiqiy nomni ko'rish:

```bash
ip a            # barcha interfeyslar
ip -4 a         # faqat IPv4
ip -br a        # qisqa jadval ko'rinishida
ip link         # MAC manzillar
```

Tartib VirtualBox'dagi adapterlar tartibiga mos: 1-adapter → `enp0s3`, 2-adapter → `enp0s8` va h.k.

### 6.2. `ip` buyrug'i bilan vaqtinchalik sozlash

```bash
sudo ip addr add 192.168.100.10/16 dev enp0s8
sudo ip link set enp0s8 up
sudo ip addr del 192.168.100.10/16 dev enp0s8
```

Reboot'dan keyin yo'qoladi. Doimiy sozlash — netplan.

### 6.3. Netplan

Netplan — YAML fayl asosidagi sozlovchi; u `systemd-networkd` uchun konfiguratsiya generatsiya qiladi. Loyiha `00-installer-config.yaml` nomini talab qiladi:

```bash
ls /etc/netplan/
sudo nano /etc/netplan/00-installer-config.yaml
```

> Ubuntu 24.04 da boshqa `.yaml` fayllar (masalan, `50-cloud-init.yaml`) ham bo'lishi mumkin. Ikki fayl bir interfeysni sozlasa, ziddiyat chiqadi. Ortiqcha faylni zaxiraga ko'chiring yoki bitta fayl bilan ishlang.

#### Asosiy struktura

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true            # NAT adapteri — internet uchun
    enp0s8:
      addresses:
        - 192.168.100.10/16  # IP/prefiks
      routes:
        - to: 172.16.0.0/12
          scope: link        # on-link (gateway'siz) marshrut
```

Qoidalar:

- **Faqat bo'sh joy (space)** bilan chekinish, **Tab ishlatmang**. Bir daraja = 2 bo'sh joy.
- Ro'yxat elementlari `-` bilan boshlanadi.
- `addresses` har doim **prefiks bilan** (`/16`).
- Fayl huquqlari `600` bo'lishi kerak, aks holda ogohlantirish chiqadi:
  ```bash
  sudo chmod 600 /etc/netplan/00-installer-config.yaml
  ```

#### Statik marshrutlar

```yaml
      routes:
        - to: 10.20.0.0/26      # qaysi tarmoq
          via: 10.100.0.12      # qaysi gateway orqali
```

#### Default marshrut

```yaml
      routes:
        - to: default
          via: 10.10.0.1
```

(Eski `gateway4:` kalit so'zi eskirgan — `routes` ishlating.)

#### DHCP va MAC

```yaml
    enp0s3:
      macaddress: "10:10:10:10:10:BA"
      dhcp4: true
```

MAC manzilni qo'shtirnoqda yozish xavfsizroq: YAML ba'zi `XX:XX:XX` ko'rinishlarini son sifatida o'qishi mumkin.

#### Qo'llash

```bash
sudo netplan try        # 120 soniya sinov, tasdiqlanmasa orqaga qaytadi (SSH uchun xavfsiz)
sudo netplan apply      # qo'llash
sudo netplan --debug apply   # xatoni batafsil ko'rsatadi
```

Tipik xatolar: chekinish noto'g'ri, interfeys nomi xato, `/prefiks` unutilgan, fayl huquqlari ochiq.

### 6.4. Kerakli paketlar

Ubuntu Server minimal; ba'zi vositalar alohida o'rnatiladi (internet — NAT adapteri yoqiq vaqtda o'rnating):

```bash
sudo apt update
sudo apt install -y ipcalc iperf3 nmap traceroute tcpdump telnet net-tools \
                    apache2 isc-dhcp-server isc-dhcp-client openssh-server
```

- `net-tools` — `netstat`, `route`, `ifconfig` uchun;
- `isc-dhcp-client` — `dhclient` uchun (24.04 da sukut bo'yicha yo'q);
- `iperf3` o'rnatishda "start as daemon?" so'rasa — **No** tanlang.

---

## 7. Diagnostika vositalari: ping, traceroute, tcpdump, nmap

### 7.1. ping va ICMP

`ping` **ICMP Echo Request** (type 8) yuboradi, javobda **Echo Reply** (type 0) keladi.

```bash
ping -c 3 192.168.100.10      # 3 ta so'rov
ping -c 1 -W 2 10.30.0.111    # 2 soniya kutish
```

Muhim ICMP turlari:

| Type | Code | Nomi | Qachon |
|---|---|---|---|
| 8 | 0 | Echo Request | ping so'rovi |
| 0 | 0 | Echo Reply | ping javobi |
| 3 | 0 | Net Unreachable | Router marshrut topa olmadi |
| 3 | 1 | Host Unreachable | Host javob bermadi |
| 3 | 3 | Port Unreachable | UDP port yopiq |
| 11 | 0 | Time Exceeded (TTL) | TTL 0 ga yetdi |

`ping` natijalari:

- `64 bytes from ...` — ishlayapti;
- `Destination Net Unreachable` — kimdir (odatda router) marshrut yo'qligini aytdi;
- jimlik / `100% packet loss` — paket yo'qoldi yoki firewall tashladi.

### 7.2. traceroute va TTL

Har bir IP paketda **TTL** (Time To Live) maydoni bor. Har bir router uni 1 ga kamaytiradi; 0 bo'lsa paketni tashlab, jo'natuvchiga **ICMP Time Exceeded** yuboradi. `traceroute` shundan foydalanadi:

```
1) TTL=1 bilan paket yuboradi → 1-router TTL ni 0 qiladi → "Time Exceeded" qaytaradi → 1-router manzili ma'lum
2) TTL=2 bilan yuboradi        → 2-router javob beradi
3) TTL=3 ...                   → manzilga yetib borgach, u "Port Unreachable" (UDP rejimida) yoki Echo Reply (ICMP rejimida) qaytaradi → tugadi
```

Sukut bo'yicha Linux `traceroute` yuqori UDP portlarga (33434 dan) paket yuboradi.

```bash
traceroute 10.20.0.10
traceroute -n 10.20.0.10     # DNS'ga murojaat qilmasdan (tezroq)
traceroute -I 10.20.0.10     # ICMP rejimi
```

Misol natija:

```
1  10.10.0.1    0.5 ms   0.4 ms   0.4 ms     ← r1
2  10.100.0.12  1.1 ms   0.9 ms   0.9 ms     ← r2
3  10.20.0.10   1.5 ms   1.2 ms   1.2 ms     ← ws21
```

r1 da `tcpdump -tnv` bilan ko'rsangiz: ws11 dan kelayotgan paketlarda `ttl 1`, so'ng `ttl 2` (r1 ularni uzatganda `ttl 1` bo'lib chiqadi), r1 dan ws11 ga ketayotgan `ICMP time exceeded in-transit` xabarlari. Hisobotda aynan shu mexanizmni tushuntirasiz.

### 7.3. tcpdump

Tarmoq interfeysidan o'tayotgan paketlarni ko'rsatadi (root kerak).

```bash
sudo tcpdump -i enp0s3                  # hamma paket
sudo tcpdump -n -i enp0s3 icmp          # faqat ICMP, nomlarga o'girmasdan
sudo tcpdump -tn -i enp0s3              # -t: vaqt belgisiz, -n: raqamli manzillar
sudo tcpdump -tnv -i enp0s3             # -v: IP sarlavha tafsiloti (ttl, id, ...)
sudo tcpdump -n -i enp0s3 tcp port 80   # filtr: TCP va port 80
```

Filtr misollari: `host 10.10.0.2`, `net 10.20.0.0/26`, `port 22`, `icmp`, `udp`.

Odatda tcpdump'ni bir terminalda ishga tushirib, ikkinchi mashinadan test qilasiz. To'xtatish: `Ctrl+C`.

### 7.4. nmap

Port skaner va host aniqlash vositasi.

```bash
nmap 192.168.100.10             # host aniqlash + eng ko'p ishlatiladigan 1000 port
nmap -sn 192.168.100.10         # faqat "host tirikmi" (portsiz)
nmap -Pn 192.168.100.10         # host aniqlashsiz, to'g'ridan-to'g'ri port skaneri
nmap -p 22,80 192.168.100.10    # aniq portlar
```

Nima uchun ping o'tmasa ham nmap `Host is up` deydi? Standart host aniqlashda nmap faqat ICMP echo emas, balki yana **TCP SYN → 443**, **TCP ACK → 80**, **ICMP timestamp** so'rovlarini yuboradi. Echo reply bloklangan bo'lsa-da, TCP so'rovga host javob (RST yoki SYN-ACK) beradi — demak host tirik. (Bir xil L2 segmentda nmap ARP'dan ham foydalanadi.)

Port holatlari: `open`, `closed` (host javob beryapti, lekin xizmat yo'q), `filtered` (firewall tashlayapti, javob yo'q).

---

## 8. Tezlik birliklari va iperf3

### 8.1. Birliklar

- **bit** (kichik `b`) va **bayt** (katta `B`): **1 bayt = 8 bit**.
- Tarmoq tezligi odatda **bit/s** (Mbps, Gbps), fayl hajmi — **bayt** (MB).
- Prefikslar (tarmoq sohasida o'nlik): 1 Kbps = 1000 bps, 1 Mbps = 1000 Kbps, 1 Gbps = 1000 Mbps.

| Konvertatsiya | Hisob | Natija |
|---|---|---|
| 8 Mbps → MB/s | 8 / 8 | **1 MB/s** |
| 100 MB/s → Kbps | 100 × 8 = 800 Mbps; 800 × 1000 | **800 000 Kbps** |
| 1 Gbps → Mbps | 1 × 1000 | **1000 Mbps** |

(Agar ikkilik prefikslar — 1 Gbit = 1024 Mbit — ishlatilsa, 1 Gbps = 1024 Mbps bo'ladi; hisobotda qaysi tizimdan foydalanganingizni yozing. Tarmoqda odatda 1000.)

### 8.2. iperf3

Client–server modelida ishlaydi: bir mashina tinglaydi, ikkinchisi yuklaydi.

```bash
# ws2 (server):
iperf3 -s

# ws1 (client):
iperf3 -c 172.24.116.8
iperf3 -c 172.24.116.8 -t 5        # 5 soniya
iperf3 -c 172.24.116.8 -R          # teskari yo'nalish
iperf3 -c 172.24.116.8 -u -b 100M  # UDP, 100 Mbit/s
```

Natijada `Bitrate` ustuni va oxirgi `sender` / `receiver` qatorlari muhim. Server standart **5201** TCP porti bilan ishlaydi — firewall bo'lsa shu portga ruxsat kerak (shu sabab iperf3 ni firewall'dan **oldin** (3-qism) sinaymiz).

---

## 9. Firewall: iptables

### 9.1. Tuzilma: jadval → zanjir → qoida

`iptables` — Linux yadrosidagi **netfilter** uchun interfeys.

| Jadval | Vazifasi |
|---|---|
| `filter` (sukut) | Paketga ruxsat berish/taqiqlash |
| `nat` | Manzillarni almashtirish (SNAT/DNAT) |
| `mangle` | Paket sarlavhalarini o'zgartirish |

`filter` jadvalidagi zanjirlar:

| Zanjir | Qaysi paketlar |
|---|---|
| `INPUT` | Shu mashinaga **kelayotgan** |
| `OUTPUT` | Shu mashinadan **chiqayotgan** |
| `FORWARD` | Shu mashina **orqali o'tayotgan** (router sifatida) |

`nat` jadvalida: `PREROUTING` (kirishda, marshrutlashdan **oldin**) — DNAT uchun, `POSTROUTING` (chiqishda, marshrutlashdan **keyin**) — SNAT/MASQUERADE uchun.

```
               ┌───────────────┐
 kirish ──► PREROUTING (nat/DNAT) ──► marshrut qarori ─┬─► INPUT ─► mahalliy jarayon
                                                       │                │
                                                       │            OUTPUT
                                                       ▼                │
                                                    FORWARD             │
                                                       │                │
                                                       └──► POSTROUTING (nat/SNAT) ──► chiqish
```

### 9.2. Qoidalar tartibi — eng muhim qoida

Zanjirdagi qoidalar **yuqoridan pastga** tekshiriladi. **Birinchi mos kelgan qoida** ishlaydi va tekshiruv shu yerda to'xtaydi (`ACCEPT`/`DROP`/`REJECT` uchun). Hech biri mos kelmasa — zanjirning **policy**'si (sukut bo'yicha `ACCEPT`).

### 9.3. Buyruqlar

```bash
sudo iptables -L -n -v                    # filter qoidalari (raqamli, hisoblagichlar bilan)
sudo iptables -L -n -v --line-numbers     # qator raqamlari bilan
sudo iptables -t nat -L -n -v             # nat jadvali
sudo iptables -F                          # filter qoidalarini tozalash (policy o'zgarmaydi!)
sudo iptables -F -t nat                   # nat jadvalini tozalash
sudo iptables -X                          # foydalanuvchi zanjirlarini o'chirish
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # oxiriga qo'shish (Append)
sudo iptables -I INPUT 1 -p tcp --dport 22 -j ACCEPT  # boshiga qo'shish (Insert)
sudo iptables -D INPUT 1                  # 1-qoidani o'chirish
sudo iptables --policy FORWARD DROP       # zanjir policy'sini o'zgartirish (-P)
```

Qoida tuzilishi: `iptables [-t jadval] -A ZANJIR [shartlar] -j HARAKAT`.

Ko'p ishlatiladigan shartlar:

| Parametr | Ma'nosi |
|---|---|
| `-p tcp/udp/icmp` | Protokol |
| `-s`, `-d` | Manba / manzil IP yoki tarmoq |
| `--sport`, `--dport` | Manba / manzil port (`-p tcp/udp` bilan) |
| `-i`, `-o` | Kiruvchi / chiquvchi interfeys |
| `--icmp-type echo-request` | ICMP turi |
| `-m conntrack --ctstate ESTABLISHED,RELATED` | Ulanish holati |

Harakatlar (`-j`): `ACCEPT` (ruxsat), `DROP` (jimgina tashlash), `REJECT` (rad etib, xabar yuborish), `MASQUERADE`, `SNAT`, `DNAT`.

**DROP** da jo'natuvchi javob olmaydi (timeout). **REJECT** da darhol "unreachable" xabari keladi.

> `iptables -F` policy'ni o'zgartirmaydi. Agar siz `-P INPUT DROP` qo'ygan bo'lsangiz va tozalasangiz — mashinaga SSH orqali kira olmay qolishingiz mumkin. Bu loyihada INPUT/OUTPUT policy'ni o'zgartirmaysiz, shuning uchun xavfsiz.

### 9.4. 4-qism: ikki strategiya

**Strategiya A — "avval taqiqla, keyin ruxsat ber" (ws1)**: taqiqlovchi qoida yuqorida. Mos paket birinchi (`DROP`) qoidaga tushadi, shuning uchun pastdagi `ACCEPT` hech qachon ishlamaydi.

**Strategiya B — "avval ruxsat ber, keyin taqiqla" (ws2)**: ruxsat beruvchi qoida yuqorida; paket `ACCEPT` ga tushadi, pastdagi `DROP` ishlamaydi.

Echo reply (ping javobi) uchun:

```bash
# ws1 — taqiq birinchi:
iptables -A OUTPUT -p icmp --icmp-type echo-reply -j DROP     # 1-chi: ishlaydi
iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT   # hech qachon yetib bormaydi
# Natija: ws1 ping'ga javob BERMAYDI.

# ws2 — ruxsat birinchi:
iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT   # 1-chi: ishlaydi
iptables -A OUTPUT -p icmp --icmp-type echo-reply -j DROP     # hech qachon yetib bormaydi
# Natija: ws2 ping'ga javob BERADI.
```

Nega `OUTPUT`? Echo reply — shu mashina **jo'natadigan** paket. Bloklash uchun chiquvchi zanjir kerak.

### 9.5. Firewall skripti

```bash
sudo nano /etc/firewall.sh
sudo chmod +x /etc/firewall.sh
sudo /etc/firewall.sh
sudo iptables -L -n -v        # tekshirish
```

Skript birinchi qatorida `#!/bin/sh` bo'lishi shart. Qoidalar reboot'dan keyin **yo'qoladi** (loyiha uchun bu normal; doimiy qilish uchun `iptables-persistent` kerak bo'ladi).

### 9.6. Holatli firewall (stateful) va conntrack

Yadro har bir ulanishni kuzatadi (`conntrack`). Paket holatlari:

- `NEW` — yangi ulanishning birinchi paketi;
- `ESTABLISHED` — allaqachon o'rnatilgan ulanish davomi;
- `RELATED` — bog'liq (masalan, ICMP xato xabari).

`FORWARD DROP` bo'lsa, javob paketlari uchun ham ruxsat kerak. Qulay yechim: ichkaridan chiqishga ruxsat + tashqaridan faqat `ESTABLISHED,RELATED` ga ruxsat (11-bo'limga qarang).

---

## 10. DHCP

### 10.1. Nima va nima uchun

**DHCP** (Dynamic Host Configuration Protocol) — mijozlarga **avtomatik** IP, maska, gateway, DNS beradi. Manzillar **ijara** (lease) asosida beriladi.

UDP portlar: server **67**, mijoz **68**. Mijozda hali IP yo'q, shuning uchun xabarlar broadcast (`255.255.255.255`) orqali yuboriladi — **DHCP faqat bitta L2 segment ichida ishlaydi** (router broadcast'ni o'tkazmaydi).

### 10.2. DORA jarayoni

```
Mijoz                              Server
  │── DHCPDISCOVER (broadcast) ───►│   "Kim DHCP server?"
  │◄── DHCPOFFER ──────────────────│   "Mana senga 10.20.0.2"
  │── DHCPREQUEST ────────────────►│   "Shuni olaman"
  │◄── DHCPACK ────────────────────│   "Tasdiqlandi, ijara muddati ..."
```

### 10.3. Server sozlash (isc-dhcp-server)

```bash
sudo apt install isc-dhcp-server
```

**1-qadam: qaysi interfeysda tinglash** — `/etc/default/isc-dhcp-server`:

```
INTERFACESv4="enp0s8"      # r2 ning ichki tarmoq interfeysi
```

Bu qadam unutilsa, server ishga tushmaydi yoki mijozlarga javob bermaydi.

**2-qadam: `/etc/dhcp/dhcpd.conf`**:

```
# Server interfeyslaridan biri ulangan tarmoq (manzil berilmaydi, e'lon uchun)
subnet 10.100.0.0 netmask 255.255.0.0 {}

# Manzil beriladigan tarmoq
subnet 10.20.0.0 netmask 255.255.255.192
{
    range 10.20.0.2 10.20.0.50;                # beriladigan manzillar havzasi (pool)
    option routers 10.20.0.1;                  # default gateway
    option domain-name-servers 10.20.0.1;      # DNS
}
```

Muhim qoida: server **faqat o'zi ulangan** subnet'lar uchun `subnet` bloklarini talab qiladi — shuning uchun `10.100.0.0` bo'sh bloki yoziladi. Aks holda ISC server xato beradi: "No subnet declaration for ...".

**3-qadam: ishga tushirish va tekshirish**:

```bash
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf     # sintaksisni tekshirish
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server
sudo journalctl -u isc-dhcp-server -e      # xato bo'lsa — jurnal
cat /var/lib/dhcp/dhcpd.leases             # berilgan manzillar
```

### 10.4. `resolv.conf`

`/etc/resolv.conf` — DNS serverlar ro'yxati:

```
nameserver 8.8.8.8
```

Ubuntu 24.04 da bu fayl odatda `systemd-resolved` ga ishora qiluvchi symlink bo'ladi (`ls -l /etc/resolv.conf`) va qo'lda yozilgan o'zgarish qayta yozilishi mumkin. Loyiha shuni talab qilgani uchun fayl ichida yozib, skrinshot olishning o'zi yetarli.

### 10.5. Mijoz (ws21) sozlash

Netplan'da statik manzil o'rniga:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
```

```bash
sudo netplan apply
ip -4 a                  # DHCP bergan manzil
ip r                     # default via 10.20.0.1 (DHCP'dan)
```

Manzil `range` (10.20.0.2–10.20.0.50) ichidan bo'ladi.

### 10.6. MAC bo'yicha qat'iy bog'lash (r1 + ws11)

**Mijoz tomoni** (ws11 netplan):

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      macaddress: "10:10:10:10:10:BA"
      dhcp4: true
```

VirtualBox'da shu adapterning MAC'ini ham mos qilib qo'yish tavsiya etiladi (Settings → Network → Advanced → MAC Address: `1010101010BA`), aks holda ikki xil MAC chalkashligi bo'lishi mumkin.

**Server tomoni** (r1, `/etc/dhcp/dhcpd.conf`):

```
subnet 10.100.0.0 netmask 255.255.0.0 {}

subnet 10.10.0.0 netmask 255.255.192.0
{
    option routers 10.10.0.1;
    option domain-name-servers 10.10.0.1;
}

host ws11 {
    hardware ethernet 10:10:10:10:10:ba;   # mijoz MAC'i
    fixed-address 10.10.0.2;               # unga doim shu manzil
}
```

Server ham `INTERFACESv4="enp0s3"` (r1 ning ws11 tomonidagi interfeysi) bilan sozlanadi. Bunda manzil faqat shu MAC egasiga beriladi.

### 10.7. Manzilni yangilash

```bash
sudo dhclient -r enp0s3      # joriy ijarani bo'shatish (release)
sudo dhclient -v enp0s3      # qayta so'rash (-v: DORA bosqichlarini ko'rsatadi)
ip -4 a
```

Muqobil (networkd bilan): `sudo networkctl renew enp0s3`.

Hisobotdagi savol — "qaysi DHCP opsiyalari ishlatildi": `subnet`/`netmask`, `range`, `option routers`, `option domain-name-servers`, (r1 uchun) `host`, `hardware ethernet`, `fixed-address`.

---

## 11. NAT: SNAT, MASQUERADE, DNAT

### 11.1. Muammo

Private manzillar (10.20.0.0/26) internetda marshrutlanmaydi. Ichki mashina tashqariga chiqqanda, javob qaytishi uchun manba manzili **router'ning tashqi manzili** bilan almashtirilishi kerak. Bu — **NAT** (Network Address Translation). Teskari holat: tashqaridan ichkaridagi xizmatga kirish uchun router'ning manzil:porti ichki mashinaga yo'naltiriladi.

### 11.2. Turlari

| Tur | Nimani o'zgartiradi | Zanjir | Qachon |
|---|---|---|---|
| **SNAT** | **Manba** manzilni | POSTROUTING | Ichkaridan tashqariga chiqish |
| **MASQUERADE** | Manbani, chiquvchi interfeys IP'si bilan (dinamik IP uchun) | POSTROUTING | Xuddi SNAT, lekin IP o'zgaruvchan bo'lsa |
| **DNAT** | **Manzil** (destination) ni | PREROUTING | Tashqaridan ichkaridagi xizmatga kirish (port forwarding) |

Yadro conntrack jadvalida o'zgarishni eslab qoladi, shu sababli **javob paketlari avtomatik teskari tarjima qilinadi** — buning uchun alohida qoida kerak emas.

### 11.3. 7-qism uchun r2 firewall skripti (bosqichma-bosqich)

Interfeys nomlari: `enp0s3` — r1 tomonga (10.100.0.12, "tashqi"), `enp0s8` — ichki tarmoq (10.20.0.1). Sizda boshqacha bo'lsa — almashtiring.

**1-versiya: 1–3 qoidalar**

```bash
#!/bin/sh
iptables -F
iptables -F -t nat
iptables --policy FORWARD DROP
```

Natija: `FORWARD` hammasini tashlaydi → ws22 dan r1 ga ping **o'tmaydi**.

**2-versiya: + ICMP**

```bash
iptables -A FORWARD -p icmp -j ACCEPT
```

Natija: ping **o'tadi**.

**3-versiya: + SNAT va DNAT**

```bash
# SNAT: r2 ortidagi 10.20.0.0/26 tashqariga r2 ning manzili bilan chiqadi
iptables -t nat -A POSTROUTING -s 10.20.0.0/26 -o enp0s3 -j MASQUERADE

# Ichkaridan tashqariga forwarding
iptables -A FORWARD -i enp0s8 -o enp0s3 -s 10.20.0.0/26 -j ACCEPT
# Tashqaridan faqat o'rnatilgan ulanish javoblari
iptables -A FORWARD -i enp0s3 -o enp0s8 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# DNAT: r2:8080 -> ws22:80
iptables -t nat -A PREROUTING -i enp0s3 -p tcp --dport 8080 -j DNAT --to-destination 10.20.0.20:80
# DNAT dan keyin paket FORWARD da ws22:80 ga ketayotgan ko'rinadi — unga ruxsat
iptables -A FORWARD -i enp0s3 -o enp0s8 -p tcp -d 10.20.0.20 --dport 80 -j ACCEPT
```

Nega shuncha qoida kerak? Chunki `FORWARD` policy'i `DROP`:

1. ichki → tashqi so'rovlar o'tishi kerak (SNAT uchun);
2. tashqi → ichki **javoblar** o'tishi kerak (`ESTABLISHED,RELATED`) — maslahatdagi "o'rnatilgan ulanishli tashqi paketlar" shu;
3. DNAT'dagi yangi ulanish ("ws22 va 80-port uchun **yangi** tcp ulanish") uchun alohida ruxsat kerak, chunki `NEW` paket `ESTABLISHED` qoidasiga tushmaydi. Filter jadvali DNAT'dan **keyin** ishlagani sababli, qoidada manzil `10.20.0.20:80` yoziladi (`8080` emas).

### 11.4. Apache

```bash
sudo nano /etc/apache2/ports.conf          # Listen 80  →  Listen 0.0.0.0:80
sudo service apache2 start                 # yoki: sudo systemctl start apache2
sudo ss -tlnp | grep :80                   # 0.0.0.0:80 da tinglayaptimi?
```

`Listen localhost:80` bo'lsa Apache faqat `127.0.0.1` da tinglaydi va tashqaridan kirib bo'lmaydi (8-qismda aynan shu kerak).

### 11.5. Tekshirish

**SNAT (ws22 → r1):**

```bash
telnet 10.100.0.11 80        # ws22 dan
# ulanganidan keyin:  GET / HTTP/1.0   (Enter ikki marta)
```

r1 da manbani ko'rish uchun: `sudo tail /var/log/apache2/access.log` — manba IP `10.100.0.12` (r2) bo'lishi kerak, `10.20.0.20` emas. Yoki `sudo tcpdump -n -i enp0s8 tcp port 80`.

**DNAT (r1 → ws22):**

```bash
telnet 10.100.0.12 8080      # r1 dan, r2 ning manzili va 8080-port
GET / HTTP/1.0
```

Apache'ning ws22 dagi sahifasi (HTML) kelishi kerak.

Sinashdan oldin VirtualBox'dagi **NAT adapter**ni o'chiring: u o'zining default marshrutini qo'shib, paketlarni boshqa yo'ldan chiqarib yuborishi mumkin.

---

## 12. SSH tunnellar

### 12.1. G'oya

SSH nafaqat masofaviy terminal, balki **shifrlangan kanal** ham. Shu kanal ichida boshqa TCP ulanishlarni "tunnel" qilib uzatish mumkin. Foydasi: yopiq tarmoqdagi yoki faqat `localhost` da tinglayotgan xizmatga xavfsiz kirish.

Oldindan: ikkala tomonda `openssh-server` bo'lishi kerak (`sudo systemctl status ssh`).

### 12.2. Local forwarding (`-L`)

**"Mening mashinamdagi portni masofaviy tomondagi xizmatga ulash."**

```
ssh -L [lokal_port]:[nishon_host]:[nishon_port] user@ssh_server
```

SSH **mijoz** tomonida port ochiladi; unga kelgan trafik server orqali `nishon_host:nishon_port` ga boradi (nishon_host — **SSH serverning nuqtai nazaridan** manzil).

8-qism uchun (ws21 → ws22):

```bash
# ws21 da (1-terminal):
ssh -L 9090:127.0.0.1:80 user@10.20.0.20
# yoki fonda, shell ochmasdan:
ssh -N -L 9090:127.0.0.1:80 user@10.20.0.20

# ws21 da (2-terminal, Alt+F2):
telnet 127.0.0.1 9090
GET / HTTP/1.0
```

Oqim: `ws21:9090` → SSH tunnel → `ws22` → `127.0.0.1:80` (ws22 ning o'z localhost'idagi Apache).

### 12.3. Remote forwarding (`-R`)

**"Masofaviy tomonda port ochib, uni o'z (mijoz) tomonimdagi xizmatga ulash."**

```
ssh -R [masofaviy_port]:[nishon_host]:[nishon_port] user@ssh_server
```

Port **server** tomonida ochiladi, trafik mijoz orqali nishonga boradi.

8-qism uchun: maqsad — ws11 dan ws22 dagi veb-serverga kirish. Xizmat ws22 da, shuning uchun SSH ulanishini **ws22 dan ws11 ga** o'rnatamiz:

```bash
# ws22 da (1-terminal):
ssh -R 9091:127.0.0.1:80 user@10.10.0.2

# ws11 da (2-terminal):
telnet 127.0.0.1 9091
GET / HTTP/1.0
```

Oqim: `ws11:9091` → SSH tunnel → `ws22` → `127.0.0.1:80`.

Nega r2 firewall'i bu bilan ishlaydi? ws22 → ws11 ga SSH (22-port) r2 orqali o'tadi. 7-qismdagi qoidalar ichkaridan tashqariga forwarding'ga va SNAT'ga ruxsat beradi, shuning uchun ulanish o'rnatiladi. Qoidalar bo'lmasa (`FORWARD DROP`) ssh ulana olmasdi.

### 12.4. Dynamic forwarding (`-D`) — qo'shimcha

`ssh -D 1080 user@server` — lokal **SOCKS5 proksi** ochadi; brauzerni `127.0.0.1:1080` ga ulab, butun trafikni server orqali o'tkazish mumkin. Loyihada talab qilinmaydi, lekin bilib qo'yish foydali.

### 12.5. Foydali flaglar

| Flag | Ma'nosi |
|---|---|
| `-N` | Buyruq bajarmaslik, faqat tunnel |
| `-f` | Fonga o'tish |
| `-L`, `-R`, `-D` | Forwarding turlari |
| `-v` | Debug |

Tunnelni to'xtatish: `Ctrl+C` yoki `exit`. Port band bo'lsa (`bind: Address already in use`) boshqa port tanlang.

---

## 13. VirtualBox stendini tayyorlash

### 13.1. Adapter turlari

| Adapter | Vazifasi |
|---|---|
| **NAT** | Internetga chiqish (paket o'rnatish uchun). Mashinalar bir-birini ko'rmaydi. |
| **Internal Network** | Faqat shu nomdagi tarmoqqa ulangan mashinalar o'rtasida L2 aloqa. Loyihada asosiy. |
| **Host-only** | Host kompyuter va VM'lar o'rtasida (kerak bo'lsa) |

### 13.2. 2–4-qismlar (ws1, ws2)

- Adapter 1: **NAT** (`enp0s3`, DHCP) — paket o'rnatish uchun;
- Adapter 2: **Internal Network**, nomi bir xil (masalan, `lan1`) → `enp0s8` — statik manzil shu yerda.

### 13.3. 5–8-qismlar (5 ta mashina)

Har bir **L2 segment** uchun **alohida** Internal Network nomi kerak:

| Segment | Internal Network nomi (misol) | Qaysi interfeyslar |
|---|---|---|
| ws11 ↔ r1 | `net_10_10` | ws11, r1 (ws11 tomoni) |
| r1 ↔ r2 | `net_link` | r1 (r2 tomoni), r2 (r1 tomoni) |
| r2 ↔ ws21, ws22 | `net_10_20` | r2 (ichki), ws21, ws22 |

Adapterlar soni: ws11 — 1, ws21 — 1, ws22 — 1, r1 — 2, r2 — 2. Paketlarni (apache2, traceroute...) o'rnatish uchun avval vaqtincha NAT adapter qo'shing, o'rnatib bo'lgach o'chiring.

> README'dagi `part5_network.png` rasmi menga yuklanmagan. Quyidagi manzillar README matnidagi misollardan (traceroute, `ip r`, dhcpd.conf) olingan: **ws11 = 10.10.0.2/18**, **r1 = 10.10.0.1/18 va 10.100.0.11/16**, **r2 = 10.100.0.12/16 va 10.20.0.1/26**, **ws21 = 10.20.0.10/26**. **ws22 = 10.20.0.20/26** esa taxmin: rasmga qarab tekshiring.

### 13.4. Foydali maslahatlar

- Bitta toza Ubuntu 24.04 Server'ni o'rnatib, kerakli paketlarni o'rnating, so'ng **klonlang** (Clone → "Generate new MAC addresses"). Klonda `hostname` va (kerak bo'lsa) `machine-id` ni o'zgartiring.
- Snapshot oling — xato qilganda orqaga qaytish oson.
- Hisobot uchun skrinshotlar: hamma buyruq va uning natijasi bitta kadrda ko'rinsin; kesib, faqat kerakli qismini qoldiring.

---

## 14. Xatolarni topish (troubleshooting)

Standart yondashuv — **pastdan yuqoriga** tekshirish:

1. **Interfeys tirikmi, IP to'g'rimi?** `ip -br a` → `UP` va kutilgan manzil.
2. **Marshrut bormi?** `ip r` va `ip r get <manzil>`.
3. **Yaqin qo'shni ko'rinadimi?** `ping <gateway>`.
4. **Router forwarding yoqilganmi?** `cat /proc/sys/net/ipv4/ip_forward` → `1`.
5. **Firewall tashlayaptimi?** `sudo iptables -L -n -v` (hisoblagichlarga qarang) va `-t nat`.
6. **Paket aslida qayergacha boryapti?** Yo'l bo'ylab har bir router'da `tcpdump`.
7. **Xizmat tinglayaptimi?** `sudo ss -tlnp`.

| Belgi | Ehtimoliy sabab |
|---|---|
| Ping so'rovi boradi, javob kelmaydi (tcpdump'da so'rov ko'rinadi) | Qaytish marshruti yo'q yoki firewall |
| `Destination Net Unreachable` | Router'da shu tarmoq uchun marshrut yo'q |
| `Network is unreachable` | Mashinaning o'zida mos marshrut (va default) yo'q |
| `Connection refused` | Xizmat ishlamayapti yoki port yopiq (RST keldi) |
| `Connection timed out` | Firewall DROP qilmoqda yoki marshrut muammosi |
| `netplan apply` xato beradi | YAML chekinish, interfeys nomi, `/prefiks` |
| DHCP manzil bermaydi | `INTERFACESv4` noto'g'ri, `subnet` bloki yo'q, sintaksis xatosi (`dhcpd -t`) |
| NAT ishlamaydi | NAT adapter yoqiq, `ip_forward=0`, FORWARD'da ruxsat yo'q |
| Ikkita default marshrut | NAT adapter DHCP orqali o'zinikini qo'shgan |

---

## 15. Tezkor ma'lumotnoma (cheat sheet)

```bash
# --- Interfeys va manzil ---
ip -br a                       ip link
sudo netplan apply             sudo netplan try

# --- Marshrut ---
ip r                           ip r get 10.20.0.10
sudo ip r add 10.20.0.0/26 via 10.100.0.12
sudo ip r add 172.16.0.0/12 dev enp0s8
sudo sysctl -w net.ipv4.ip_forward=1

# --- Diagnostika ---
ping -c 3 <ip>                 traceroute -n <ip>
sudo tcpdump -n -i <iface> icmp
nmap <ip>                      sudo ss -tlnp

# --- Tezlik ---
iperf3 -s                      iperf3 -c <server_ip>

# --- Firewall ---
sudo iptables -L -n -v         sudo iptables -t nat -L -n -v
sudo iptables -F               sudo iptables -F -t nat
sudo iptables --policy FORWARD DROP

# --- DHCP ---
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
sudo systemctl restart isc-dhcp-server
sudo dhclient -r <iface>; sudo dhclient -v <iface>

# --- SSH tunnel ---
ssh -N -L 9090:127.0.0.1:80 user@host      # local
ssh -N -R 9091:127.0.0.1:80 user@host      # remote
```

Asosiy formulalar:

- Hostlar soni = 2^(32 − prefiks) − 2
- Blok o'lchami = 256 − maska_oktet
- Bayt = 8 bit; 1 Gbps = 1000 Mbps = 125 MB/s
- Marshrut tanlash: **eng uzun prefiks yutadi**
- iptables: **birinchi mos qoida ishlaydi**
- DNAT → `PREROUTING`; SNAT/MASQUERADE → `POSTROUTING`

Omad! Tushunmay qolgan joyda avval 14-bo'limdagi tekshiruv ketma-ketligidan o'ting.
