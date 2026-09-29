## Jarkom-Modul-2-2026-K-53
Data Communication and Computer Networks Practicum

### Soal 1 :

![](assets/Topologi.png)

1. Router (rootkit)
Buka konsol rootkit, lalu ketik perintah berikut:
```bash
# Memasang IP dan mengaktifkan interface untuk semua subnet
ip addr add 10.90.2.1/24 dev eth1
ip addr add 10.90.4.1/24 dev eth2
ip addr add 10.90.3.1/24 dev eth3
ip addr add 10.90.1.1/24 dev eth4
ip addr add 10.90.5.1/24 dev eth5

ip link set eth1 up
ip link set eth2 up
ip link set eth3 up
ip link set eth4 up
ip link set eth5 up

# Memastikan IP Forwarding aktif
sysctl -w net.ipv4.ip_forward=1
```
2. Sayap Kiri (alpha, beta, gamma) - Subnet 10.90.1.x (Gateway: 10.90.1.1)\
alpha:
```bash
ip addr flush dev eth0
ip addr add 10.90.1.2/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.1.1
```
beta:
```Bash
ip addr flush dev eth0
ip addr add 10.90.1.3/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.1.1
```
gamma:
```Bash
ip addr flush dev eth0
ip addr add 10.90.1.4/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.1.1
```
3. Sayap Kanan (delta, epsilon) - Subnet 10.90.5.x (Gateway: 10.90.5.1)\
delta:
```Bash
ip addr flush dev eth0
ip addr add 10.90.5.2/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.5.1
```
epsilon:
```Bash
ip addr flush dev eth0
ip addr add 10.90.5.3/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.5.1
```
4. Gerbang Penyaring (abbey, penny)\
abbey (Subnet 10.90.4.x | Gateway: 10.90.4.1):
```Bash
ip addr flush dev eth0
ip addr add 10.90.4.2/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.4.1
```
penny (Subnet 10.90.3.x | Gateway: 10.90.3.1):
```Bash
ip addr flush dev eth0
ip addr add 10.90.3.2/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.3.1
```
5. Area Bawah (prab, tedd, obladi, desmond, oblada, molly) - Subnet 10.90.2.x (Gateway: 10.90.2.1)\
prab:
```Bash
ip addr flush dev eth0
ip addr add 10.90.2.2/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.2.1
```
tedd:
```Bash
ip addr flush dev eth0
ip addr add 10.90.2.3/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.2.1
```
obladi:
```Bash
ip addr flush dev eth0
ip addr add 10.90.2.4/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.2.1
```
desmond:
```Bash
ip addr flush dev eth0
ip addr add 10.90.2.5/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.2.1
```
oblada:
```Bash
ip addr flush dev eth0
ip addr add 10.90.2.6/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.2.1
```
molly:
```Bash
ip addr flush dev eth0
ip addr add 10.90.2.7/24 dev eth0
ip link set eth0 up
ip route add default via 10.90.2.1
```
### Soal 2 : Konfigurasi NAT & Akses Internet\
Langkah 1: Konfigurasi pada Router (rootkit)\
Buka konsol rootkit, lalu kita jalankan perintah berikut untuk mengaktifkan masquerading (NAT) dan meneruskan paket data ke semua interface internal (eth1 sampai eth5):
```bash
# Memasang IP secara manual dari jaringan NAT GNS3
ip addr add 192.168.122.100/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.122.1

# Konfigurasi iptables untuk NAT / Masquerading
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth2 -j ACCEPT
iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth3 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth4 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth5 -j ACCEPT
```
### Soal 3 : Routing Internal & Resolver Awal\
Kita memastikan setiap host non-router menambahkan resolver sementara 192.168.122.1 pada file /etc/resolv.conf agar akses untuk mengunduh paket instalasi dari internet dapat tersedia sejak awal.
```bash
# Uji ping ke gateway dari klien (alpha)
ping -c 3 10.90.1.1

# Pengaturan resolver dan uji akses internet
echo "nameserver 192.168.122.1" > /etc/resolv.conf
ping -c 3 google.com
```
![](assets/alpha-ping-internet.png)

### Soal 4 : Konfigurasi DNS Server Master-Slave

#### Langkah 1 : Instalasi BIND9 di Prab dan Tedd

Kita buka konsol node prab dan tedd, lalu melakukan instalasi paket bind9:
```bash
echo "nameserver 8.8.8.8" > /etc/resolv.conf
apt-get update
apt-get install bind9 -y
```
#### Langkah 2 : Konfigurasi Master DNS pada prab (ns1)

Edit file konfigurasi utama (/etc/bind/named.conf.options):\
Mengatur bagian forwarders mengarah ke 192.168.122.1:
```bash
options {
    directory "/var/cache/bind";
    forwarders {
        8.8.8.8;
    };
    dnssec-validation auto;
    listen-on { any; };
};
```
Menambahkan definisi zona di /etc/bind/named.conf.local:\
Kita daftarkan domain k53.com sebagai master dan berikan izin allow-transfer ke IP stedd (10.90.2.3):
```bash
zone "k53.com" {
    type master;
    file "/etc/bind/k53/k53.com";
    allow-transfer { 10.90.2.3; };
    notify yes;
};
```
Membuat direktori dan file zona (/etc/bind/k53/k53.com):\
Membuat folder /etc/bind/k53, 
```bash
mkdir -p /etc/bind/k53
nano /etc/bind/k53/k53.com
```
lalu buat file zona dengan isi rekaman DNS (SOA, NS, A record untuk prab, stedd, dan apex k53.com yang mengarah ke IP penny yaitu 10.90.3.2):
```bash
$TTL 604800
@   IN  SOA prab.k53.com. root.k53.com. (
        2026092901 ; Serial
        604800     ; Refresh
        86400      ; Retry
        2419200    ; Expire
        604800 )   ; Negative Cache TTL

@   IN  NS  prab.k53.com.
@   IN  NS  tedd.k53.com.

prab    IN  A   10.90.2.2
tedd   IN  A   10.90.2.3
@       IN  A   10.90.3.2
```
Restart layanan BIND9 di prab:
```bash
named
```
Kita bisa cek apakah sudah jalan:
```bash
ps aux | grep named
```
#### Langkah 3 : Konfigurasi Slave DNS pada tedd (ns2)

Edit file konfigurasi zona di /etc/bind/named.conf.local:\
Daftarkan zona k53.com sebagai slave yang mengambil data dari master prab (10.90.2.2):
```bash
zone "k53.com" {
    type slave;
    file "/var/cache/bind/k53.com";
    masters { 10.90.2.2; };
};
```
Restart layanan BIND9 di tedd:
```bash
named
```
(Catatan: Setelah di-restart, kita cek /var/cache/bind/k53.com di node tedd untuk memastikan file zona berhasil di-transfer dari prab)

#### Langkah 4 : Pembaruan Resolver pada Seluruh Node Non-Router

Setelah DNS internal hidup, kita perbarui urutan resolver pada seluruh Entitas non-router menjadi: IP prab (10.90.2.2), IP stedd (10.90.2.3), lalu 192.168.122.1.\
Jalankan perintah ini di setiap node klien (misalnya alpha):
```bash
echo -e "nameserver 10.90.2.2\nnameserver 10.90.2.3\nnameserver 192.168.122.1" > /etc/resolv.conf
```
#### Langkah 5 : Verifikasi

Uji dari node klien (alpha) apakah query domain apex maupun hostname dijawab dengan benar:
```bash
nslookup k53.com
nslookup prab.k53.com
nslookup tedd.k53.com
```
![](assets/coba.png)
