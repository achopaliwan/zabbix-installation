# Zabbix Problem and Solving

<p>
<h2> Data tidak update</h2>
Data pada web monitoring tidak terupdate secara otomatis. Ditandai dengan adanya notifikasi: <em>"zabbix server is not running the information may not be current"</em>. Cara memperbaiki:
</p>

<h2>1. Ceck dan tambah kapasitas CacheSize</h2>

Yang pertama dilakukan adalah mengecek file Log dari Zabbix Server yang berlokasi di path `/var/log/zabbix/zabbix_server.log` . Jika kita menemukan log seperti berikut, <br>

>10582:20190729:003449.780 [file:dbconfig.c,line:94] __zbx_mem_realloc(): out of memory (requested 512 bytes)<br>
10582:20190729:003449.780 [file:dbconfig.c,line:94] __zbx_mem_realloc(): please increase CacheSize configuration parameter

<p>
Artinya kapasitas CacheSize tidak cukup dan harus ditambahkan. CacheSize merupakan shared memory yang digunakan server zabbix untuk storing/menyimpan data host, item dan trigger dan berbeda dengan Cache yang terdapat pada server/komputer itu sendiri.
</p>

<p>
Cara menambah CacheSize adalah dengan mengubah nilai CacheSize pada file <br></p>

```shel
/etc/zabbix/zabbix_server.conf
```
<p>
dan ubah nilai default CacheSize<br></p>

>CacheSize=64M<br>

Kemudian restart zabbix server. Setelah direstart zabbix sudah normal kembali, namun jika belum juga maka bisa dilanjutkan dengan cara di bawah. 


<h2> 2. Mengganti password database zabbix_server.conf dan zabbix.conf.php </h2>

<p>ganti password pada DBPassword=</p>

```shel
nano /etc/zabbix/zabbix_server.conf
```

<p>ganti password pada $DB['PASSWORD']=</p>

```shel
nano /etc/zabbix/web/zabbix.conf.php
```
<br>
Update mySql PASSWORD<br>

```shel
mysql -u root -p
mysql> SET PASSWORD FOR zabbix@localhost = PASSWORD('password');
mysql> flush privileges;
mysql> exit;
```
restart Apache2 dan zabbix-server.

<h2> 3. Buat Database Baru </h2>
<p>
Dapat dilakukan dengan cara mengahapus dan membuat kembali database untuk zabbix server. Caranya, sbb :
</p>

```shel
mysql -u root -p
mysql> show databases;
mysql> drop database zabbix;
mysql> CREATE DATABASE zabbix;
mysql> GRANT ALL on zabbix.* to zabbix@localhost IDENTIFIED BY 'password';
mysql> flush privileges;
mysql> exit;
```

Memuat zabbix database schema ke database yang telah dibuat diatas:

```shel
cd /usr/share/doc/zabbix-server-mysql
zcat create.sql.gz | mysql -u root -p zabbix
```

<p>
Edit Zabbix Configuration File: <br></p>

```shel
nano /etc/zabbix/zabbix_server.conf
```

>DBHost=localhost <br>
DBName=zabbix <br>
DBUser=zabbix <br>
DBPassword=password <br>


<p>
langkah terakhir adalah restart Apache2 dan zabbix-server.
</p>

-Note tambahan, untuk setup firewall :

Perintah paling dasar pada firewall ufw adalah `ufw allow <port>/<ptotocol>`. 
Contoh: `ufw allow 22/tcp` untuk mengijinkan koneksi SSH dari komputer lain.

Untuk Zabbix server kita berikan ijin akses port 10050 dan 10051 sperti berikut

```shel
sudo ufw allow proto tcp from any to any port 10050,10051
```

atau satu per satu:

```shel
$ sudo ufw allow proto tcp from 172.16.2.xxx port 10050
$ sudo ufw allow proto tcp from 172.16.2.xxx port 10050
```

Cara cek koneksi pada IP dan port tertentu:
```shel
$ nc -v -z 172.16.2.79 10050
Connection to 172.16.2.79 10050 port [tcp/zabbix-agent] succeeded!
```
