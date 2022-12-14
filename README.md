# Laporan Resmi Praktikum Modul 1 Jaringan Komputer 2022

## Anggota Kelompok C10:

| Nama                   | NRP        |
| ---------------------- | ---------- |
| Pierra Muhammad Shobr  | 5025201062 |
| Barhan Akmal Falahudin | 5025201008 |
| Arief Badrus Sholeh    | 5025201228 |

Berikut adalah hasil topologi jaringan yang kami buat.

![Topologi](/screenshot/topologi.png)

> Prefix IP C10 : 192.184

#### Configurasi masing - masing node

- Ostania

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.184.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.184.2.1
	netmask 255.255.255.0

auto eth3
iface eth2 inet static
	address 192.184.2.1
	netmask 255.255.255.0

```

- Wise

```
auto eth0
iface eth0 inet static
	address 192.184.2.2
	netmask 255.255.255.0
	gateway 192.184.2.1
```

- SSS

```
auto eth0
iface eth0 inet static
	address 192.184.1.2
	netmask 255.255.255.0
	gateway 192.184.1.1
```

- Garden

```
auto eth0
iface eth0 inet static
	address 192.184.1.3
	netmask 255.255.255.0
	gateway 192.184.1.1
```

- Berlint

```
auto eth0
iface eth0 inet static
	address 192.184.3.2
	netmask 255.255.255.0
	gateway 192.184.3.1
```

- Eden

```
auto eth0
iface eth0 inet static
	address 192.184.3.3
	netmask 255.255.255.0
	gateway 192.184.3.1
```

WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden.

## Soal 1

> Semua node terhubung pada router Ostania, sehingga dapat mengakses internet.

- Konfigurasi `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.184.0.0/16` pada router Ostania.
- Mengetikkan command `echo nameserver 192.168.122.1 > /etc/resolv.conf` di node ubuntu yang lain dimana 192.168.122.1 merupakan IP DNS.

#### Testing

- Lakukan ping ke `google.com` pada setiap node

![Soal 1](/screenshot/1.png)

> Dapat kita lihat setiap node sudah dapat terhubung dengan internet melalui ping kepada `google.com`

## Soal 2

> Membuat website utama dengan akses `wise.yyy.com` dengan alias `www.wise.yyy.com` pada folder wise.

#### Konfigurasi pada _WISE_

- Konfigurasi domain `wise.c10.com` pada file /etc/bind/named.conf.local menggunakan syntax berikut

```
zone "wise.c10.com" {
	type master;
	file "/etc/bind/wise/wise.c10.com";
};
```

- Buat folder wise di dalam /etc/bind

```
mkdir /etc/bind/wise
```

- Konfigurasi DNS Record pada file /etc/bind/wise/wise.c10.com menggunakan syntax

```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	wise.c10.com. root.wise.c10.com. (
		2022102501		; Serial
		    604800		; Refresh
		     86400		; Retry
		   2419200		; Expire
		    604800)		; Negative
;
@	IN	NS	wise.c10.com.
@	IN	A	192.184.2.2	; IP WISE
www	IN	CNAME	wise.c10.com.
@	IN	AAAA	::1'
```

#### Testing

- Lakukan ping ke `wise.c10.com`, `www.wise.c10.com`, dan cek CNAME

![2](/screenshot/2.png)

> Dapat kita lihat domain `wise.c10.com` sudah dapat diakses mengarah ke IP WISE begitu pula dengan `www.wise.c10.com` yang merupakan domain alias-nya.

## Soal 3

> Membuat subdomain `eden.wise.yyy.com` dengan alias `www.eden.wise.yyy.com` yang diatur DNS-nya di WISE dan mengarah ke Eden.

#### Konfigurasi pada _WISE_

- Menambahkan konfigurasi DNS Record pada file etc/bind/wise/wise.c10.com menggunakan syntax

```
eden	IN	A	192.184.3.3	; IP Eden
www.eden	IN	CNAME	eden.wise.c10.com.
```

#### Testing

- Lakukan ping ke `eden.wise.c10.com`, `www.eden.wise.c10.com`, dan cek CNAME

![3](/screenshot/3.png)

> Dapat kita lihat domain `eden.wise.c10.com` sudah dapat diakses mengarah ke IP Eden begitu pula dengan `www.eden.wise.c10.com` yang merupakan domain alias-nya.

## Soal 4

> Membuat juga reverse domain untuk domain utama.

#### Konfigurasi pada _WISE_

- Menambahkan konfigurasi berikut ke dalam file named.conf.local. Tambahkan reverse dari 3 byte awal dari IP yang ingin dilakukan Reverse DNS. Karena doma utama menggunakan IP 192.184.2 untuk IP dari records, maka reversenya adalah 2.168.192

```
zone "2.184.192.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/2.184.192.in-addr.arpa";
}
```

- Konfigurasi DNS Record pada file /etc/bind/wise/2.184.192.in-addr.arpa menggunakan syntax

```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	wise.c10.com. root.wise.c10.com. (
		2022102501		; Serial
		    604800		; Refresh
		     86400		; Retry
		   2419200		; Expire
		    604800)		; Negative
;
2.184.192.in-addr.arpa.	IN	NS	wise.c10.com.
2   IN	PTR	wise.c10.com. ; Byte ke-4 WISE
```

#### Testing

- Lakukan cek menggunakan perintah `host -t PTR 192.184.2.2`

![4](/screenshot/4.png)

> Dapat kita lihat bahwa `2.2.184.192.in-addr.arpa.` mengarah ke `wise.c10.com`.

## Soal 5

> Membuat Berlint sebagai DNS Slave untuk domain utama agar dapat tetap dihubungi jika server WISE bermasalah.

#### Konfigurasi pada _WISE_

- Menambahkan konfigurasi berikut pada zone `wise.c10.com` dalam file /etc/bind/named.conf.local

```
zone "jarkom2022.com" {
    ...
    notify yes;
    also-notify { 192.184.3.2; };
    allow-transfer { 192.184.3.2; };
    ...
};
```

#### Konfigurasi pada _Berlint_

- Konfigurasi pada file /etc/bind/named.conf.local

```
zone "wise.c10.com.com" {
    type slave;
    masters { 192.184.2.2; };
    file "/var/lib/bind/wise.c10.com.com";
};
```

#### Testing

- Pastikan pengaturan nameserver pada client mengarah ke IP _WISE_ dan IP _Berlint_.

![5a](/screenshot/5a.png)

- Pada server _WISE_ matikan service bind9.

![5b](/screenshot/5b.png)

- Lakukan ping ke `wise.c10.com` pada client _SSS_.

![5c](/screenshot/5c.png)

## Soal 6

> Membuat subdomain yang khusus untuk operation yaitu `operation.wise.yyy.com` dengan alias `www.operation.wise.yyy.com` yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation.

#### Konfigurasi pada _WISE_

- Menambahkan konfigurasi DNS Record pada file etc/bind/wise/wise.c10.com menggunakan syntax

```
ns1	IN	A	192.184.3.3	; IP Eden
operation	IN	CNAME	eden.wise.c10.com.
```

- Kemudian comment dnssec-validation auto; dan tambahkan baris berikut pada /etc/bind/named.conf.options

```
options {
	directory "/var/cache/bind";

	allow-query{any;};

	auth-nxdomain no;	# conform to RFC1035
	listen-on-v6 { any; };
};
```

#### Konfigurasi pada _Eden_

- Comment dnssec-validation auto; dan tambahkan baris berikut pada /etc/bind/named.conf.options

```
options {
	directory "/var/cache/bind";

	allow-query{any;};

	auth-nxdomain no;	# conform to RFC1035
	listen-on-v6 { any; };
};
```

- Konfigurasi domain `operation.wise.c10.com` pada file /etc/bind/named.conf.local menggunakan syntax berikut

```
zone "operation.wise.c10.com" {
	type master;
	file "/etc/bind/delegasi/operation.wise.c10.com";
};
```

- Buat folder delegasi di dalam /etc/bind

```
mkdir /etc/bind/delegasi
```

- Konfigurasi DNS Record pada file /etc/bind/delegasi/operation.wise.c10.com menggunakan syntax

```
;
; BIND data file for local loopback interface
;
$TTL	604800
@	IN	SOA	operation.wise.c10.com. root.operation.wise.c10.com. (
		2022102501		; Serial
		    604800		; Refresh
		     86400		; Retry
		   2419200		; Expire
		    604800)		; Negative
;
@	IN	NS	operation.wise.c10.com.
@	IN	A	192.184.3.3	; IP Eden
www	IN	CNAME	wise.c10.com.
```

#### Testing

- Lakukan ping ke `operation.wise.c10.com`, `www.operation.wise.c10.com`, dan cek CNAME

![6](/screenshot/6.png)

> Dapat kita lihat domain `operation.wise.c10.com` sudah dapat diakses mengarah ke IP Eden begitu pula dengan `www.operation.wise.c10.com` yang merupakan domain alias-nya.

## Soal 7

> Membuat subdomain melalui Berlint dengan akses `strix.operation.wise.yyy.com` dengan alias `www.strix.operation.wise.yyy.com` yang mengarah ke Eden.

#### Konfigurasi pada _Eden_

- Menambahkan konfigurasi DNS Record pada file etc/bind/delegasi/operation.wise.c10.com menggunakan syntax

```
strix	IN	A	192.184.3.3	; IP Eden
www.strix	IN	CNAME	strix.operation.wise.c10.com.
```

#### Testing

- Lakukan ping ke `strix.operation.wise.c10.com`, `www.strix.operation.wise.c10.com`, dan cek CNAME

![7](/screenshot/7.png)

> Dapat kita lihat domain `strix.operation.wise.c10.com` sudah dapat diakses mengarah ke IP Eden begitu pula dengan `www.strix.operation.wise.c10.com` yang merupakan domain alias-nya.

## Soal 8

> Dengan `www.wise.yyy.com`, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com.

#### Mengambil resource Wise
Untuk mengambil resource wise, dapat digunakan beberapa cara, antara lain: wget, git. Kelompok kami menggunakan cara untuk membuat sebuah repository github yang nanti dapat diambil dengan git. Repository yang kami pakai adalah pada link [berikut](https://github.com/akmalf06/jarkom_resource_wise). Adapun command yang kami lakukan adalah sebagai berikut:

```
apt-get update
apt-get install git -y

git config --system user.name "jarkom2c10"
git config --system user.email "akmaldavinci06@gmail.com"
git config --system http.sslVerify false

cd /var/www
git clone https://github.com/akmalf06/jarkom_resource_wise.git wise.c10.com
```

Dengan cara tersebut, kami dapat mengambil resource wise dan menempatkan pada /var/www/wise.c10.com. Adapun buktinya adalah sebagai berikut:

![8a](/screenshot/8a.png)

![8b](/screenshot/8b.png)

#### Melakukan setup webserver apache2
Berikut adalah command yang kami gunakan untuk melakukan setup webserver apache2:
```
apt-get update
apt-get install apache2 -y
echo 'Servername 127.0.0.1' >> /etc/apache2/apache2.conf
apt-get install php -y
apt-get install libapache2-mod-php7.0 -y

echo '<VirtualHost *:80>
    <Directory /var/www/wise.c10.com>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
    </Directory>
    ServerName wise.c10.com
    ServerAlias www.wise.c10.com 
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/wise.c10.com
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/wise.c10.com.conf

service apache2 start
a2dissite 000-default
a2ensite wise.c10.com
service apache2 restart
```
Kami menambahkan konfigurasi apache2 untuk port 80 ke dalam file `/etc/apache2/sites-available/wise.c10.com.conf`. Selanjutnya konfigurasi `000-default` kami nonaktifkan dan diganti dengan konfigurasi `wise.c10.com.conf` yang baru saja dibuat.

#### Testing
Kami melakukan testing dengan menggunakan lynx pada node SSS dan Garden. Berikut adalah hasil testing terhadap setup webserver yang telah dilakukan sebelumnya:

![8c](/screenshot/8c.png)

> Terlihat bahwa webserver sudah dapat diakses dari node SSS dan Garden

## Soal 9

> Url `www.wise.yyy.com/index.php/home` dapat menjadi menjadi `www.wise.yyy.com/home`.

Untuk dapat melakukan perintah sesuai dengan soal, kami mengaktifkan modul `rewrite` pada apache2. Selanjutnya kami melakukan konfigurasi apache2 pada `/var/www/wise.c10.com/.htaccess`. Adapun perintah yang kami lakukan adalah sebagai berikut:

```
a2enmod rewrite

echo 'RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule (.*) /index.php/$1 [L]' > /var/www/wise.c10.com/.htaccess

service apache2 restart
```

Adapun makna dari perintah di atas adalah untuk mengarahkan semua request, contohnya `/home` ke `/index.php/home`.

#### Testing

![9a](/screenshot/9a.png)

![9b](/screenshot/9b.png)

## Soal 10

> Pada subdomain `www.eden.wise.yyy.com`, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/eden.wise.yyy.com.

#### Mengambil resource Wise
Untuk mengambil resource wise, dapat digunakan beberapa cara, antara lain: wget, git. Kelompok kami menggunakan cara untuk membuat sebuah repository github yang nanti dapat diambil dengan git. Repository yang kami pakai adalah pada link [berikut](https://github.com/akmalf06/jarkom_resource_wise). Adapun command yang kami lakukan adalah sebagai berikut:

```
apt-get update
apt-get install git -y

git config --system user.name "jarkom2c10"
git config --system user.email "akmaldavinci06@gmail.com"
git config --system http.sslVerify false

cd /var/www
git clone https://github.com/akmalf06/jarkom_resource_eden.git eden.wise.c10.com
```

Dengan cara tersebut, kami dapat mengambil resource wise dan menempatkan pada /var/www/eden.wise.c10.com.

#### Melakukan setup webserver apache2
Berikut adalah command yang kami gunakan untuk melakukan setup webserver apache2:
```
apt-get update
apt-get install apache2 -y
echo 'Servername 127.0.0.1' >> /etc/apache2/apache2.conf
apt-get install php -y
apt-get install libapache2-mod-php7.0 -y

echo '<VirtualHost *:80>
    <Directory /var/www/eden.wise.c10.com>
        Options -Indexes +FollowSymLinks +MultiViews
        AllowOverride All
        Require all granted
    </Directory>
    <Directory /var/www/eden.wise.c10.com/public>
        Options Indexes followSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    ServerName eden.wise.c10.com
    ServerAlias www.eden.wise.c10.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/eden.wise.c10.com
    RedirectMatch 301 ^/$ http://www.wise.c10.com
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>' > /etc/apache2/sites-available/eden.wise.c10.com.conf

service apache2 start
a2dissite 000-default
a2ensite eden.wise.c10.com
service apache2 restart
```
Kami menambahkan konfigurasi apache2 untuk port 80 ke dalam file `/etc/apache2/sites-available/eden.wise.c10.com.conf`. Selanjutnya konfigurasi `000-default` kami nonaktifkan dan diganti dengan konfigurasi `eden.wise.c10.com.conf` yang baru saja dibuat. Adapun beberapa konfigurasi tambahan untuk soal soal berikutnya juga kami tambahkan pada konfigurasi karena merupakan satu kesatuan. 

#### Testing
Kami melakukan testing dengan menggunakan lynx pada node SSS dan Garden. Berikut adalah hasil testing terhadap setup webserver yang telah dilakukan sebelumnya:

![10a](/screenshot/10a.png)

![10b](/screenshot/10b.png)

## Soal 11

> Akan tetapi, pada folder /public, Loid ingin hanya dapat melakukan directory listing saja.

Untuk dapat melakukan perintah sesuai dengan soal, kami menambahkan konfigurasi berikut pada `/etc/apache2/sites-available/eden.wise.c10.com.conf`:

```
<Directory /var/www/eden.wise.c10.com/public>
	Options Indexes followSymLinks
	AllowOverride None
	Require all granted
</Directory>
```

Adapun perintah yang dapat menyebabkan `/public` dapat menjadi directory listing adalah pada `Options Indexes`. 

#### Testing
Adapun bukti dapat dilihat pada bukti di soal nomor 10.

## Soal 12

> Loid juga ingin menyiapkan error file 404.html pada folder /error untuk mengganti error kode pada apache.

Untuk dapat menggunakan custom html pada `eden.wise.c10.com`, kami menambahkan perintah berikut pada `/var/www/eden.wise.c10.com/.htaccess`:

```
ErrorDocument 404 /error/404.html
```

Sehingga semua error 404 akan diarahkan pada /error/404.html

#### Testing

![12a](/screenshot/12a.png)

![12b](/screenshot/12b.png)

## Soal 13

> Loid juga meminta Franky untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset `www.eden.wise.yyy.com/public/js` menjadi `www.eden.wise.yyy.com/js`.

Untuk dapat melakukan perintah sesuai dengan soal, kami mengaktifkan modul `rewrite` pada apache2. Selanjutnya kami melakukan konfigurasi apache2 pada `/var/www/eden.wise.c10.com/.htaccess`. Adapun perintah yang kami lakukan adalah sebagai berikut:

```
a2enmod rewrite

echo 'RewriteEngine On
RewriteRule ^js$ public/js [L]
RewriteRule ^public\/images\/(?!eden\.png).*eden.*$ /public/images/eden.png [L,R=301]' > /var/www/eden.wise.c10.com/.htaccess

service apache2 restart
```

Adapun makna dari perintah di atas adalah untuk mengarahkan request `/js` ke `/public/js`.

#### Testing

![13a](/screenshot/13a.png)

![13b](/screenshot/13b.png)

## Soal 14

> Loid meminta agar `www.strix.operation.wise.yyy.com` hanya bisa diakses dengan port 15000 dan port 15500.

- masukkan port 15000 dan 15500 ke ports.conf di eden

```
echo '
Listen 15000
Listen 15500' >> /etc/apache2/ports.conf
```

- masukkan juga port 15000 dan 15500 ke strix.operation.wise.c10.com.conf

```
<VirtualHost *:15000 *:15500>
```

![14](/screenshot/14.PNG)

## Soal 15

> dengan autentikasi username Twilight dan password opStrix dan file di /var/www/strix.operation.wise.yyy.

- Masukan potongan koding berikut ke server-setup.sh

```
htpasswd -b -c /etc/apache2/.htpasswd Twilight opStrix
```

![15](/screenshot/15.PNG)

## Soal 16

> dan setiap kali mengakses IP Eden akan dialihkan secara otomatis ke 'www.wise.yyy.com'.

- masukan potongan koding berikut ke server-setup.sh pada bagian VirtualHost secara permanen dengan (301)

```
RedirectMatch 301 ^/$ http://www.wise.c10.com
```

![16](/screenshot/16.PNG)

## Soal 17

> Karena website `www.eden.wise.yyy.com` semakin banyak pengunjung dan banyak modifikasi sehingga banyak gambar-gambar yang random, maka Loid ingin mengubah request gambar yang memiliki substring ???eden??? akan diarahkan menuju eden.png. Bantulah Agent Twilight dan Organisasi WISE menjaga perdamaian!

- Masukan file eden.png ke eden
- masukan aturan baru, jika terdapat substring eden maka akan disalurkan ke eden.png

```
RewriteRule ^public\/images\/(?!eden\.png).*eden.*$ /public/images/eden.png
```

![17](/screenshot/17.PNG)
