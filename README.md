# Jarkom_Modul3_Lapres_C11

1. 05111840000131 - Aaron Astonvilla Rompis
2. 05111840000151 - Nikodemus Siahaan

## No 1, Topologi 
dengan konfigurasi sbb
```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &
uml_switch -unix switch3 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.76.49 eth1=daemon,,,switch1 eth2=daemon,,,switch3 eth3=daemon,,,switch2 mem=256M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=160M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T TUBAN -e linux ubd0=TUBAN,jarkom umid=TUBAN eth0=daemon,,,switch2 mem=128M &

# Klien
xterm -T BANYUWANGI -e linux ubd0=BANYUWANGI,jarkom umid=BANYUWANGI eth0=daemon,,,switch3 mem=64M &
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch1 mem=64M &
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch1 mem=64M &
xterm -T MADIUN -e linux ubd0=MADIUN,jarkom umid=MADIUN eth0=daemon,,,switch3 mem=64M &

```
![gambar1](/Gambar/Screenshot_184.png)



lalu lakukan seperti pengenalan UML yaitu
  
nano /etc/sysctl.conf (surabaya)
uncomment net.ipv4.ip_forward=1
sysctl -p

lalu ubah interface setiap uml menjadi

![gambar1](/Gambar/Screenshot_185.png)

![gambar1](/Gambar/Screenshot_186.png)

![gambar1](/Gambar/Screenshot_187.png)

![gambar1](/Gambar/Screenshot_188.png)

![gambar1](/Gambar/Screenshot_189.png)

![gambar1](/Gambar/Screenshot_190.png)

![gambar1](/Gambar/Screenshot_191.png)

![gambar1](/Gambar/Screenshot_192.png)



## 2  Karena TUBAN jauh dari client, maka SURABAYA ditunjuk sebagai perantara (DHCP Relay) antara DHCP Server dan client.

- Install DHCP di Tuban
- Setting Config isc-dhcp-server Tuban

![gambar1](/Gambar/Screenshot_193.png)

- Install DHCP Relay di surabaya
- Setting Config isc-dhcp-relay Surabaya,

![gambar1](/Gambar/Screenshot_194.png)

Kemudian tambahkan subnet NID_DMZ seperti berikut agar dhcp relay bisa berjalan, 

![gambar1](/Gambar/Screenshot_195.png)

- Kemudian ubah semua client yang awalnya static menjadi menggunakan dhcp.
nb: ss sudah langsung masuk ke dhcp karena sudah diubah (coment)

## 3 dan 4. Atur Client mendapatkan range tertentu
```
(3) Client pada subnet 1 mendapatkan range IP dari 192.168.0.10 sampai 192.168.0.100 dan
192.168.0.110 sampai 192.168.0.200.
(4) Client pada subnet 3 mendapatkan range IP dari 192.168.1.50 sampai 192.168.1.70.
```

- Ubah Config /etc/dhcp/dhcpd.conf, **Gambar :**

![gambar1](/Gambar/Screenshot_196.png)

## 5. Client mendapatkan DNS Malang dan DNS 202.46.129.2 dari DHCP
- nano /etc/dhcp/dhcpd.conf pada Tuban tambahkan rangenya, **Gambar :**

![gambar1](/Gambar/Screenshot_196.png)

dan hasilnya

![gambar1](/Gambar/Screenshot_197.png)

## 6. Client di subnet 1 mendapatkan peminjaman alamat IP selama 5 menit, sedangkan subnet 3 dapat 10 menit
- nano /etc/dhcp/dhcpd.conf pada Tuban tambahkan, **Gambar :**

![gambar1](/Gambar/Screenshot_196.png)

-Sudah di test waktu demo



### Dynamic Host Configuration

### Proxy
## No 7, Membuat autentikasi milik Anri
Pertama, akses ke proxy hanya bisa dilakukan oleh Anri sendiri sebagai user TA. User autentikasi milik Anri memiliki format: User : userta_c11, Password : inipassw0rdta_c11
- Pada UML **MOJOKERTO** install squid dengan perintah ```apt-get install squid```
- Pada konfigurasi ```nano /etc/squid/squid.conf``` tambahkan
  ```
  http_port 8080
  visible_hostname mojokerto
  ```
- Lalukan restart squid dengan perintah ```service squid restart```
- Atur proxy pada browser 
- Pada UML **MOJOKERTO** install apache2-utils dengan perintah ```apt-get install apache2-utils```
- Dengan perintah ```htpasswd -c /etc/squid/passwd userta_c11``` membuat user dan password.
- Buka file konfigurasi dengan perintah ```nano /etc/squid/squid.conf``` untuk menambahkan
  ```
  auth_param basic program /usr/lib/squid/ncsa_auth /etc/squid/passwd
  auth_param basic children 5
  auth_param basic realm Proxy
  auth_param basic credentialsttl 2 hours
  auth_param basic casesensitive on
  acl USERS proxy_auth REQUIRED
  ```
- Lakukan restart squid dengan perintah ```service squid restart```, dan saat membuka browser akan muncul perintah autentikasi seperti berikut:
![gambar7](/Gambar/7.png)


## No 8 9, Mengakses hanya pada hari dan waktu tertentu
- untuk mempermudah, maka sebelumnya buat dulu file untuk konfigurasi waktu dengan menggunakan perintah ```nano /etc/squid/acl.conf``` pada UML **MOJOKERTO**
- lalu, isi file tersebut dengan 
```
acl AVAILABLE_WORKING time TW 13:00-18:00
acl AVAILABLE_WORKING2 time TWH 21:00-23:59
acl AVAILABLE_WORKING3 time WHF 00:00-09:00
```
- setelah itu, tambahkan konfigurasi berikut pada ```squid.conf``` dengan
```
include /etc/squid/acl.conf #ini untuk mengakses file acl.conf yang sudah dibuat tadi

http_access allow USERS AVAILABLE_WORKING
http_access allow USERS AVAILABLE_WORKING2
http_access allow USERS AVAILABLE_WORKING3
```
- setelah sudah semua, jangan lupa untuk merestart squid dengan perintah ```service squid restart````

### No 10, Ketika mengakses Google akan diarahkan ke monta.if.its.ac.id
- buka kembali file konfigurasi squid dengan ```nano /etc/squid/squid.conf``` pada UML **MOJOKERTO** 
- lalu, tambahkan konfigurasi seperti berikut 
```
acl REDIRSITE dstdomain google.com
http_access deny REDIRSITE
deny_info 301:http://monta.if.its.ac.id REDIRSITE
```
- simpan, dan restart squid ```service squid restart```
![gambar10](/Gambar/10.jpg)

### No 11, Mengubah error page.
- download dulu file yang dibutuhkan dengan menggunakan perintah ```wget``` pada UML **MOJOKERTO**
- setelah itu copy file yang didownload tadi dengan melakukan perintah berikut ``` cp -r ERR_ACCESS_DENIED /usr/share/squid/errors/English/ ```
- jangan lupa mendefinisikan file error tadi pada ```squid.conf``` dengan menambahkan
```
error_directory /usr/share/squid/errors/English
```
- dan ketika dicoba akan menjadi:
![gambar11](/Gambar/11.png)

### No 12, mengubah address proxy.
- Pada nomor ini diharuskan mengonfigurasi BIND pada UML **MALANG**
- Sebelumnya install dulu BIND9 dengan ``` apt-get install bind9 ```
- Lalu konfigurasi named.conf.local dengan ```nano /etc/bind/named.conf.local``` yang berisi
```
zone "janganlupa-ta.c11.pw" {
 type master;
 file "/etc/bind/jarkom/janganlupa-ta.c11.pw";
};
```
- Setelah itu, buat directory jarkom dengan menggunakan ``` mkdir /etc/bind/jarkom/ ```
- Lalu copy file cb.local folder jarkom dengan nama janganlupa-ta.c11.pw  dengan menggunakan ```cp /etc/bind/db.local /etc/bind/jarkom/janganlupa-ta.c11.pw```
- Lalu ubah konfigurasi menjadi 
```
@ IN NS janganlupa-ta.c11.pw.
@ IN A 10.151.77.99 ;IP MOJOKERTO
```
- Setelah itu restart bind dengan ```service bind9 restart```
- Lalu coba mengganti proxy server browser dengan ``` janganlupa-ta.c11.pw dan port:8080 ```
- Dan ketika mencoba akses akan keluar seperti di bawah ini:
![gambar12](/Gambar/12.png)
