# Jarkom_Modul3_Lapres_C11

1. 05111840000131 - Aaron Astonvilla Rompis
2. 05111840000151 - Nikodemus Siahaan

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
- Dengan perintah ```htpasswd -c /etc/squid/passwd userta_c12``` membuat user dan password.
- Buka file konfigurasi dengan perintah ```nano /etc/squid/squid.conf``` untuk menambahkan
  ```
  auth_param basic program /usr/lib/squid/ncsa_auth /etc/squid/passwd
  auth_param basic children 5
  auth_param basic realm Proxy
  auth_param basic credentialsttl 2 hours
  auth_param basic casesensitive on
  acl USERS proxy_auth REQUIRED
  http_access allow USERS
  ```
- Lakukan restart squid dengan perintah ```service squid restart```, dan saat membuka browser akan muncul perintah autentikasi seperti berikut:

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
- 

### No 11, Mengubah error page.
- download dulu file yang dibutuhkan dengan menggunakan perintah ```wget``` pada UML **MOJOKERTO**
- setelah itu copy file yang didownload tadi dengan melakukan perintah berikut ``` cp -r ERR_ACCESS_DENIED /usr/share/squid/errors/English/ ```
- jangan lupa mendefinisikan file error tadi pada ```squid.conf``` dengan menambahkan
```
error_directory /usr/share/squid/errors/English
```
- dan ketika dicoba akan menjadi:


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
- Lalu copy file cb.local folder jarkom dengan nama janganlupa-ta.c11.pw  dengan menggunakan ```cp /etc/bind/db.local /etc/bind/jarkom/janganlupa-ta.e01.pw```
- Lalu ubah konfigurasi menjadi 
```
@ IN NS janganlupa-ta.c11.pw.
@ IN A 10.151.71.19 ;IP MOJOKERTO
```
- Setelah itu restart bind dengan ```service bind9 restart```
- Lalu coba mengganti proxy server browser dengan ``` janganlupa-ta.c11.pw dan port:8080 ```
- Dan ketika mencoba akses akan keluar seperti di bawah ini:
