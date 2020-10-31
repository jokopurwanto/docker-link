# Pengenalan Docker Link
![Link](https://user-images.githubusercontent.com/32213421/97776029-c4c21f00-1b97-11eb-8432-5b2b8f5a8268.JPG)

Docker mempunyai dua cara untuk menghubungkan antar container yang pertama melalui `link` dan yang kedua melalui `network`, pada pembahasan kali ini kita akan membahas mengenai `link` sedangkan untuk `network` akan dibuatkan pembahasannya secara terpisah.

**Docker link** adalah fitur koneksi dari docker yang digunakan untuk menghubungkan antar container. `Link` merupakan fitur lama yang saat ini mulai ditingalkan semenjak diperkenalkannya fitur `network` pada docker version 1.9.

![LegacyLink](https://user-images.githubusercontent.com/32213421/97776233-f8ea0f80-1b98-11eb-9722-bc3397cb7cb8.PNG)

Docker sendiri menyarankan untuk mulai beralih menggunakan `network` karena ke depan fitur `link` akan di hapus dari docker, tetapi tidak ada salahnya kita belajar `link` untuk menambah pemahaman kita mengenai docker.

Jika kalian ingin mengetahui lebih lanjut mengenai `link` kalian bisa mengunjungi dokumentasinya [di sini](https://docs.docker.com/network/links/). 
## Cara Kerja Link
![DockerLink](https://user-images.githubusercontent.com/32213421/97776032-c68be280-1b97-11eb-8428-e94f2b75608e.JPG)

Ketika kita menggunakan `link` untuk menghubungkan container satu dengan container lain maka docker akan melakukan **dua hal**:
1. **Pertama** docker akan melakukan set `environment variables` pada target container berdasarkan parameter `--link`. Tujuannya untuk mengekspos setiap `environment variables` yang berasal dari **source container** sebagai `environment variables` di **target container**. Contoh set `environment variables` dari redis sebagai berikut:
    - REDISDB_PORT=tcp://172.17.0.2:6379
      
      Docker membuat `environment variables` **alias_PORT**, variabel ini berisi URL port yang terekspos dari **source container**.
    - REDISDB_ENV_REDIS_VERSION=6.0.9 
      
      Docker membuat `environment variables` **alias_ENV_name**, variabel ini berisi ver redis yang terekspos dari **source containe**r.

2. **Kedua** docker akan melakukan update **file hosts** pada `target container`. **Tujuannya** untuk memetakan **hostname ke IP address** dengan memasukan tiga hostname yaitu **original**, **alias** dan **hash_id**. Contoh format update file hosts (**IP_address alias hash_id original**) sebagai berikut: **172.17.0.2  redisDB  78c26ae78e09  redis-server**

## Contoh Skenario Docker Link
![SkenarioDockerLink](https://user-images.githubusercontent.com/32213421/97776033-c7bd0f80-1b97-11eb-8a38-0d44e45d929f.JPG)

Pada skenario ini kita akan menggunakan base image redis, dari base image redis kita akan buat **dua container**. Satu container sebagai **redis server** dan satu container sebagai **redis client**. Redis client akan bertugas untuk memasukan data ke redis server sedangkan redis server bertugas untuk menampung data yang di masukan oleh redis client. Untuk koneksi antar container kita gunakan `link` sebagai penghubung.
## Membuat Docker Container
Pertama buat container **redis-server** dengan menggunakan base image `redis:6.0.9`.
```
$ docker run -d --name redis-server redis:6.0.9
```
![redis-server](https://user-images.githubusercontent.com/32213421/97776068-f6d38100-1b97-11eb-8e63-de22376d034a.PNG)
- **-d** digunakan untuk menjalankan container di mode `detached` (background).
- **--name** digunakan untuk memberikan nama container redis-server.
- **redis:6.0.9** merupakan base image yang digunakan untuk membuat container.

Kemudian buat container **redis-client** dengan menggunakan base image `redis:6.0.9`.
```
$ docker run -d --name redis-client --link redis-server:redisDB redis:6.0.9
```
![redis-client](https://user-images.githubusercontent.com/32213421/97776070-f935db00-1b97-11eb-9d2f-0ddca4baeffa.PNG)
- **-d** digunakan untuk menjalankan container di mode `detached` (background).
- **--name** digunakan untuk memberikan nama container **redis-client**.
- **--link** digunakan untuk `linking` ke **redis-server** dengan memberikannya alias **redisDB**.
- **redis:6.0.9** merupakan base image yang digunakan untuk membuat container.

Pastikan semua container dalam kondisi aktif, untuk mengechecknya gunakan perintah berikut:
```
$ docker ps
```
![docker-ps](https://user-images.githubusercontent.com/32213421/97776072-fb983500-1b97-11eb-9ea1-8bd6e9b5867e.PNG)
Semua container dalam status `up` yang artinya semua dalam kondisi aktif.
## Test Linking Pada Redis Client
Masuk ke dalam container **redis-client**
```
$ docker exec -it redis-client /bin/bash
```
![docker-exec](https://user-images.githubusercontent.com/32213421/97776073-fe932580-1b97-11eb-97ba-21d58ea57513.PNG)
- **-it** digunakan untuk masuk ke `interactive mode`.
- **redis-client** merupakan nama container yang dijalankan.
- **/bin/bash** digunakan untuk membuat bash session pada container **redis-client**.

Setelah menjalankan perintah tersebut kalian akan masuk dalam **bash session** dengan tampilan prompt sebagai berikut:
```
root@eb5c13c92285:/data#
```
![docker-bash](https://user-images.githubusercontent.com/32213421/97776075-005ce900-1b98-11eb-96e4-8865413a1995.PNG)

Catatan:
- Tanda **$** menunjukan prompt dari host machine milik kita.
- Tanda **#** menunjukan prompt dari container yang kita jalankan (**redis-client**).

Sesuai dengan cara kerja `link`, docker akan melakukan dua hal pertama akan melakukan set `environment variables` dan kedua akan melakukan update pada `file hosts`.
## Periksa Environment Variables
Lakukan pengecekan `environment variables` pada container (**redis-client**) dengan menjalankan perintah berikut:
```
# env
```
![docker-bash-env](https://user-images.githubusercontent.com/32213421/97776077-02bf4300-1b98-11eb-8782-ccfeb1285c29.PNG)

Maka akan ditampilkan semua `environment variables`, terdapat environment variables yang di ekspos dari **source container** (redis-server) hasil dari proses `linking`.
## Periksa File Hosts
Kemudian lakukan pengecekan **file hosts** dengan menjalankan perintah berikut:
```
# cat /etc/hosts
```
![docker-bash-hosts](https://user-images.githubusercontent.com/32213421/97776082-0652ca00-1b98-11eb-9a32-db3fa9a19afc.PNG)

Maka akan ditampilkan pemetaan tiga hostname (**original**, **alias** dan **hash_id**) ke **IP address**. 
## Insert Data ke Container Redis Server
Pertama kita masuk ke redis server, melalui perintah berikut:
```bash
# redis-cli -h redisDB
```
![docker-redis-cli](https://user-images.githubusercontent.com/32213421/97776083-094dba80-1b98-11eb-89d4-51086d07cf5e.PNG)
- **-h** digunakan untuk merujuk ke hostname.
- **redisDB** merupakan hostname yang tuju. Kita bisa menggunakan salah satu hostname karena hasil dari `linking` menghasilkan tiga hostname (**original**, **alias** dan **hash_id**).

Catatan:
- Tanda **$** menunjukan prompt dari host machine milik kita.
- Tanda **#** menunjukan prompt dari container yang kita jalankan (**redis-client**).
- Tanda **>** menunjukan prompt dari redis.

Kemudian lakukan test **PING** yang akan mendapatkan balasan **PONG** artinya redis berhasil dipasang dengan benar.
```bash
redisDB:6379> PING
PONG
```
![docker-redis-cli-ping](https://user-images.githubusercontent.com/32213421/97776085-0bb01480-1b98-11eb-98e9-a621bf7de927.PNG)

Lakukan operasi menyimpan data dengan `key` name untuk menyimpan informasi data nama, lalu data tersebut di print untuk ditampilkan.
```bash
redisDB:6379> set name "Joko Purwanto"
redisDB:6379> get name 
"Joko Purwanto"
```
![docker-redis-cli-set](https://user-images.githubusercontent.com/32213421/97776086-0e126e80-1b98-11eb-8b8a-cf3cebec19ab.PNG)

Sampai di sini kita telah berhasil menggunakan `link` untuk menghubungkan **source container** sebagai redis server dengan **target container** sebagai redis client.
