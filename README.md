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
Langkah 2: Verifikasi Akses Internet pada Host Klien\
Kita memastikan setiap host non-router menambahkan resolver sementara 192.168.122.1 pada file /etc/resolv.conf agar akses untuk mengunduh paket instalasi dari internet dapat tersedia sejak awal. Buka konsol salah satu klien (misalnya alpha), lalu ketik:
```bash
echo "nameserver 192.168.122.1" > /etc/resolv.conf
ping -c 3 google.com
```
![](assets/alpha-ping-internet.png)

