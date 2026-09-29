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

#### Langkah 1 : Instalasi BIND9 di prab dan tedd

Buka konsol node prab dan tedd, lalu instal paket bind9 (resolver awal sudah `192.168.122.1` dari Soal 3):
```bash
echo "nameserver 192.168.122.1" > /etc/resolv.conf
apt-get update
apt-get install bind9 dnsutils -y
```

#### Langkah 2 : Konfigurasi Master DNS pada prab (ns1)

Edit `/etc/bind/named.conf.options`, dengan forwarders mengarah ke `192.168.122.1`:
```bash
cat > /etc/bind/named.conf.options << 'EOF'
options {
    directory "/var/cache/bind";
    forwarders {
        192.168.122.1;
    };
    dnssec-validation no;
    listen-on { any; };
    allow-query { any; };
};
EOF
```

Daftarkan zona `k53.com` sebagai master di `/etc/bind/named.conf.local`, lengkap dengan `allow-transfer` dan `notify` ke tedd (10.90.2.3):
```bash
cat > /etc/bind/named.conf.local << 'EOF'
zone "k53.com" {
    type master;
    file "/etc/bind/k53/k53.com";
    allow-transfer { 10.90.2.3; };
    notify yes;
};
EOF
```

Buat direktori dan file zona `/etc/bind/k53/k53.com` (SOA, NS, A record prab dan tedd, serta A record apex `k53.com` yang mengarah ke penny, yaitu 10.90.3.2):
```bash
mkdir -p /etc/bind/k53
cat > /etc/bind/k53/k53.com << 'EOF'
$TTL 604800
@   IN  SOA prab.k53.com. root.k53.com. (
        2026092901 ; Serial
        604800     ; Refresh
        86400      ; Retry
        2419200    ; Expire
        604800 )   ; Negative Cache TTL

@       IN  NS  prab.k53.com.
@       IN  NS  tedd.k53.com.

prab    IN  A   10.90.2.2
tedd    IN  A   10.90.2.3
@       IN  A   10.90.3.2
EOF
```

Cek sintaks config dan zona sebelum menjalankan named:
```bash
named-checkconf
named-checkzone k53.com /etc/bind/k53/k53.com
```

Jalankan named (matikan dulu proses lama supaya config baru terbaca dan tidak jalan dobel):
```bash
pkill named
sleep 1
named
```

Cek apakah named sudah jalan (harus hanya satu proses):
```bash
ps aux | grep named
```

#### Langkah 3 : Konfigurasi Slave DNS pada tedd (ns2)

Samakan `/etc/bind/named.conf.options` dengan prab:
```bash
cat > /etc/bind/named.conf.options << 'EOF'
options {
    directory "/var/cache/bind";
    forwarders {
        192.168.122.1;
    };
    dnssec-validation no;
    listen-on { any; };
    allow-query { any; };
};
EOF
```

Daftarkan zona `k53.com` sebagai slave yang mengambil data dari master prab (10.90.2.2) di `/etc/bind/named.conf.local`:
```bash
cat > /etc/bind/named.conf.local << 'EOF'
zone "k53.com" {
    type slave;
    file "/var/cache/bind/k53.com";
    masters { 10.90.2.2; };
};
EOF
```

Cek config, lalu jalankan named:
```bash
named-checkconf
pkill named
sleep 1
named
```

Pastikan zona berhasil di-transfer dari prab:
```bash
ls -l /var/cache/bind/k53.com
```

#### Langkah 4 : Pembaruan Resolver pada Seluruh Node Non-Router

Setelah DNS internal hidup, perbarui urutan resolver pada seluruh node non-router (alpha, beta, gamma, delta, epsilon, prab, tedd, abbey, penny, obladi, desmond, oblada, molly) menjadi: prab (10.90.2.2), tedd (10.90.2.3), lalu 192.168.122.1.

Jalankan di setiap node:
```bash
echo -e "nameserver 10.90.2.2\nnameserver 10.90.2.3\nnameserver 192.168.122.1" > /etc/resolv.conf
```

#### Langkah 5 : Verifikasi

Uji dari klien (misalnya alpha) bahwa apex maupun hostname dijawab dengan benar:
```bash
dig k53.com
dig prab.k53.com
dig tedd.k53.com
```
Ketiganya harus mengembalikan `NOERROR` dengan flag `aa`, dengan jawaban `10.90.3.2`, `10.90.2.2`, dan `10.90.2.3`.

Pastikan tedd juga menjawab secara authoritative dan serial SOA di kedua server sama:
```bash
dig @10.90.2.2 k53.com SOA +short
dig @10.90.2.3 k53.com SOA +short
```

Pastikan akses internet lewat forwarders tetap berfungsi:
```bash
ping -c 2 google.com
```
![](assets/ping-google.png)
