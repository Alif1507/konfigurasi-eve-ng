```

2. Konfigurasi IP Address (Netplan) Mengatur IP statis IPv4 `192.168.24.10/24` dan IPv6 `2001:db8:24::10/64` serta menentukan gateway.

```bash
sudo nano /etc/netplan/50-cloud-init.yaml

```

```yaml
network:
  version: 2
  ethernets:
    ens3:
      addresses:
        - 192.168.24.10/24
        - 2001:db8:24::10/64
      routes:
        - to: default
          via: 192.168.24.1
        - to: "::/0"
          via: "2001:db8:24::1"
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1

```

```bash
sudo netplan apply

```

3. Instalasi & Setup Layanan 

```bash
sudo apt update
sudo apt install apache2 openssh-server bind9 dnsutils -y

```

Membuat halaman web biodata HTML:

```bash
sudo nano /var/www/html/index.html

```

```html
<h1>Ujian Praktik Pra ASS XI SIJA 2 Tahun 2026</h1>
<p>Nama : Muhammad Alif Wahyudi</p>
<p>Kelas : XI SIJA 2</p>
<p>Absen: 24</p>
<p>Network Engineer - Keren!</p>

```

---

TUGAS 2 & 3: Cisco Router (IPv4 & IPv6) 

Berfungsi sebagai penghubung jaringan server dan MikroTik.

1. Setup Interface & IP Address 

```text
enable
configure terminal
hostname R-CISCO-24
ipv6 unicast-routing

! Arah ke Linux Server (fa0/0)
interface FastEthernet0/0
ip address 192.168.24.1 255.255.255.0
ipv6 address 2001:db8:24::1/64
no shutdown
exit

! Arah ke MikroTik (fa1/0)
interface FastEthernet1/0
ip address 10.24.1.1 255.255.255.0
ipv6 address 2001:db8:24:1::1/64
no shutdown
exit

```

2. Static Routing 

```text
! Routing statis ke jaringan Client
ip route 172.16.24.0 255.255.255.0 10.24.1.2
ipv6 route 2001:db8:24:2::/64 2001:db8:24:1::2

! Default route ke Internet
ip route 0.0.0.0 0.0.0.0 10.24.1.2
do write

```

---

TUGAS 4: MikroTik Router 

Menangani NAT , DHCP Server untuk Client , dan menjembatani akses ke koneksi Cloud/Internet.

1. Identity & Interface Setup 

```text
/system identity set name="MT-24"
/ipv6 settings set forward=yes

! IP ke arah Cisco (ether1)
/ip address add address=10.24.1.2/24 interface=ether1
/ipv6 address add address=2001:db8:24:1::2/64 interface=ether1

! IP ke arah Client (ether2)
/ip address add address=172.16.24.1/24 interface=ether2
/ipv6 address add address=2001:db8:24:2::1/64 interface=ether2

! Ambil IP Internet EVE-NG otomatis (ether3)
/ip dhcp-client add interface=ether3 disabled=no add-default-route=yes

```

2. NAT & Routing 

```text
/ip dns set servers=8.8.8.8 allow-remote-requests=yes

! NAT Masquerade
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
/ip firewall nat add chain=srcnat out-interface=ether3 action=masquerade

! Routing IPv4 & IPv6
/ip route add dst-address=192.168.24.0/24 gateway=10.24.1.1
/ipv6 route add dst-address=::/0 gateway=2001:db8:24:1::1
/ipv6 route add dst-address=2001:db8:24::/64 gateway=2001:db8:24:1::1

```

3. DHCP Server (Untuk Client) Memberikan IP otomatis rentang `.100 - .150`.

```text
/ip pool add name=dhcp_pool ranges=172.16.24.100-172.16.24.150
/ip dhcp-server add name=dhcp1 interface=ether2 address-pool=dhcp_pool disabled=no
/ip dhcp-server network add address=172.16.24.0/24 gateway=172.16.24.1 dns-server=192.168.24.10,8.8.8.8

```

---

TUGAS 5: Pengujian (Client) 

Buktikan Client berhasil melakukan Ping IPv4 dan IPv6 ke Mikrotik, Cisco, dan Linux.

**1. Mendapatkan IP Address**

```bash
# Request DHCP IPv4 dari MikroTik
sudo dhclient e0

# Menambahkan IP IPv6 dan Gateway secara manual
sudo ip -6 addr add 2001:db8:24:2::10/64 dev e0
sudo ip -6 route add default via 2001:db8:24:2::1

```

**2. Perintah Pengujian (Testing)**

```bash
# Ping IPv4 ke luar & lokal
ping -4 google.com
ping 192.168.24.10

# Ping IPv6 lokal
ping6 2001:db8:24:2::1
ping6 2001:db8:24::1
ping6 2001:db8:24::10

# Cek Web Server
curl [http://192.168.24.10](http://192.168.24.10)

# Pengecekan Rute (Traceroute)
traceroute 192.168.24.10

# Pengecekan DNS (NSLookup)
nslookup server24.sija.local 192.168.24.10

```

```

```Biar dokumentasi lu buat persiapan LKS Wilayah Jaktim nanti makin rapi dan gampang dibaca, ini *full raw script* Markdown-nya dari awal sampai akhir pengujian. Lu tinggal klik **Copy code** di pojok kanan atas blok ini, terus *save* jadi `laporan_terintegrasi_24.md` bro.

```markdown
# [cite_start]Laporan Tugas Akhir Praktik Terintegrasi XI SIJA 2026 [cite: 1]

[cite_start]**Identitas Murid** [cite: 13]
* [cite_start]**Nama:** Muhammad Alif Wahyudi [cite: 14, 15]
* [cite_start]**Kelas:** XI SIJA 2 [cite: 16, 17]
* **No. [cite_start]Absen:** 24 [cite: 20]
* **Topologi:** Linux Server - Cisco - MikroTik - Linux Client (EVE-NG)

---

## [cite_start]TUGAS 1: Linux Server [cite: 192, 193]
[cite_start]Server ini bertindak sebagai Web Server dan SSH Server[cite: 57, 216, 217], serta DNS Server lokal.

**1. [cite_start]Mengatur Hostname** [cite: 194, 195]
```bash
sudo hostnamectl set-hostname server-absen24

```

2. Konfigurasi IP Address (Netplan) Mengatur IP statis IPv4 `192.168.24.10/24` dan IPv6 `2001:db8:24::10/64` serta menentukan gateway.

```bash
sudo nano /etc/netplan/50-cloud-init.yaml

```

```yaml
network:
  version: 2
  ethernets:
    ens3:
      addresses:
        - 192.168.24.10/24
        - 2001:db8:24::10/64
      routes:
        - to: default
          via: 192.168.24.1
        - to: "::/0"
          via: "2001:db8:24::1"
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1

```

```bash
sudo netplan apply

```

3. Instalasi & Setup Layanan 

```bash
sudo apt update
sudo apt install apache2 openssh-server bind9 dnsutils -y

```

Membuat halaman web biodata HTML:

```bash
sudo nano /var/www/html/index.html

```

```html
<h1>Ujian Praktik Pra ASS XI SIJA 2 Tahun 2026</h1>
<p>Nama : Muhammad Alif Wahyudi</p>
<p>Kelas : XI SIJA 2</p>
<p>Absen: 24</p>
<p>Network Engineer - Keren!</p>

```

---

TUGAS 2 & 3: Cisco Router (IPv4 & IPv6) 

Berfungsi sebagai penghubung jaringan server dan MikroTik.

1. Setup Interface & IP Address 

```text
enable
configure terminal
hostname R-CISCO-24
ipv6 unicast-routing

! Arah ke Linux Server (fa0/0)
interface FastEthernet0/0
ip address 192.168.24.1 255.255.255.0
ipv6 address 2001:db8:24::1/64
no shutdown
exit

! Arah ke MikroTik (fa1/0)
interface FastEthernet1/0
ip address 10.24.1.1 255.255.255.0
ipv6 address 2001:db8:24:1::1/64
no shutdown
exit

```

2. Static Routing 

```text
! Routing statis ke jaringan Client
ip route 172.16.24.0 255.255.255.0 10.24.1.2
ipv6 route 2001:db8:24:2::/64 2001:db8:24:1::2

! Default route ke Internet
ip route 0.0.0.0 0.0.0.0 10.24.1.2
do write

```

---

TUGAS 4: MikroTik Router 

Menangani NAT , DHCP Server untuk Client , dan menjembatani akses ke koneksi Cloud/Internet.

1. Identity & Interface Setup 

```text
/system identity set name="MT-24"
/ipv6 settings set forward=yes

! IP ke arah Cisco (ether1)
/ip address add address=10.24.1.2/24 interface=ether1
/ipv6 address add address=2001:db8:24:1::2/64 interface=ether1

! IP ke arah Client (ether2)
/ip address add address=172.16.24.1/24 interface=ether2
/ipv6 address add address=2001:db8:24:2::1/64 interface=ether2

! Ambil IP Internet EVE-NG otomatis (ether3)
/ip dhcp-client add interface=ether3 disabled=no add-default-route=yes

```

2. NAT & Routing 

```text
/ip dns set servers=8.8.8.8 allow-remote-requests=yes

! NAT Masquerade
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
/ip firewall nat add chain=srcnat out-interface=ether3 action=masquerade

! Routing IPv4 & IPv6
/ip route add dst-address=192.168.24.0/24 gateway=10.24.1.1
/ipv6 route add dst-address=::/0 gateway=2001:db8:24:1::1
/ipv6 route add dst-address=2001:db8:24::/64 gateway=2001:db8:24:1::1

```

3. DHCP Server (Untuk Client) Memberikan IP otomatis rentang `.100 - .150`.

```text
/ip pool add name=dhcp_pool ranges=172.16.24.100-172.16.24.150
/ip dhcp-server add name=dhcp1 interface=ether2 address-pool=dhcp_pool disabled=no
/ip dhcp-server network add address=172.16.24.0/24 gateway=172.16.24.1 dns-server=192.168.24.10,8.8.8.8

```

---

TUGAS 5: Pengujian (Client) 

Buktikan Client berhasil melakukan Ping IPv4 dan IPv6 ke Mikrotik, Cisco, dan Linux.

**1. Mendapatkan IP Address**

```bash
# Request DHCP IPv4 dari MikroTik
sudo dhclient e0

# Menambahkan IP IPv6 dan Gateway secara manual
sudo ip -6 addr add 2001:db8:24:2::10/64 dev e0
sudo ip -6 route add default via 2001:db8:24:2::1

```

**2. Perintah Pengujian (Testing)**

```bash
# Ping IPv4 ke luar & lokal
ping -4 google.com
ping 192.168.24.10

# Ping IPv6 lokal
ping6 2001:db8:24:2::1
ping6 2001:db8:24::1
ping6 2001:db8:24::10

# Cek Web Server
curl [http://192.168.24.10](http://192.168.24.10)

# Pengecekan Rute (Traceroute)
traceroute 192.168.24.10

# Pengecekan DNS (NSLookup)
nslookup server24.sija.local 192.168.24.10

```

```

```
