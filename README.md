# Parallel Programming dengan MPI dan Python
Tutorial menggunakan message passing interface (MPI) dengan Python. Tutorial ini menggunakan Linux Ubuntu 22.04

## 0. Sebelum Bekerja
- Siapkan beberapa komputer. Tentukan satu komputer untuk master dan beberapa komputer untuk worker.
- Pastikan seluruh komputer terhubung dalam satu jaringan (misalnya: terhubung dengan wifi yang sama).
- Update dan upgrade package `sudo apt update && sudo apt upgrade`.
- Install beberapa package sebagai utilitas, yaitu `net-tools` dan `vim` (opsional).
```sh
sudo apt install net-tools vim
```
- Untuk memeriksa IP komputer, gunakan perintah `ifconfig`.
## 1. Konfigurasi File `/etc/hosts`
### 1.1 Untuk Master
Buka file `/etc/hosts` melalui vim (atau vi, nano, yang lainnya). Tambahkan beberapa IP dan aliasnya dari masing-masing komputer.
```sh
192.168.1.100 master
192.168.1.200 worker1
192.168.1.300 worker2
```
### 1.2 Untuk Worker
Sama seperti pada master, tambahkan IP dan aliasnya, namun hanya IP master dan IP komputer itu sendiri.
```sh
192.168.1.100 master
192.168.1.200 worker1
```
## 2. Buat User Baru
### 2.1 Buat User
> *Lakukan di **Master** dan **Worker***

Buat user baru dengan nama yang sama di masing-masing komputer. Saya menamakannya `mpiuser`.
```sh
sudo adduser mpiuser
```
### 2.2 Beri Akses Root
> *Lakukan di **Master** dan **Worker***
```sh
sudo usermod -aG mpiuser
```
### 2.3 Masuk ke User yang Telah Dibuat
> *Lakukan di **Master** dan **Worker***
```sh
su - mpiuser
```
## 3. Konfigurasi SSH
Lakukan konfigurasi SSH setelah masuk ke user yang telah dibuat sebelumnya.
### 3.1 Install SSH
> *Lakukan di **Master** dan **Worker***
```sh
sudo apt install openssh-server
```
Anda dapat menghubungkan komputer dengan SSH.
```sh
ssh <nama user>@<host>

# Misal: ssh mpiuser@worker1
```
### 3.2 Buat Key
> *Lakukan di **Master***

Ketik perintah di bawah.
```sh
ssh-keygen -t rsa
```
Anda akan diminta memasukkan beberapa input, lewati semuanya.

SSH key akan disimpan di direktori `~/.ssh` dalam bentuk `id_rsa` (privasi) dan `id_rsa.pub` (publik).
### 3.3 Salin Key Publik ke Worker
> *Lakukan di **Master***

Key publik perlu disalin ke masing-masing worker dan disimpan ke dalam file `auhorized_keys`. Hal ini dapat dilakukan di master dengan bantuan SSH.
```sh
cd ~/.ssh
cat id_rsa.pub | ssh <nama user>@<host> "mkdir .ssh; cat >> .ssh/authorized_keys"
```
Ganti `<nama user>` dengan nama user dan `<host>` dengan worker. Lakukan perintah ini ke seluruh worker.

File `authorized_keys` akan muncul di direktori `~/.ssh`.
## 4. Konfigurasi NFS
### 4.1 Buat Shared Folder
> *Lakukan di **Master** dan **Worker***

Buat folder dengan nama bebas, misalnya cloud. Disarankan diletakkan di direktori `~/` atau `~/Desktop`.
```sh
mkdir ~/cloud
```
### 4.2 Install NFS Server
> *Lakukan di **Master***
```sh
sudo apt install nfs-kernel-server
```
### 4.3 Konfigurasi File `/etc/exports`
> *Lakukan di **Master***

Buka file `/etc/exports` dengan vim `sudo vim /etc/exports` (atau dengan text editor yang lainnya).

Tambahkan baris berkut.
```sh
<shared folder> *(rw,sync,no_root_squash,no_subtree_check)

# Misal: /home/mpiuser/cloud *(rw,sync,no_root_squash,no_subtree_check)
```
Selanjutnya jalankan perintah berikut.
```sh
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```
### 4.4 Install NFS Client
> *Lakukan di **Worker***
```sh
sudo apt install nfs-common
```
### 4.5 Mount Folder
> *Lakukan di **Worker***
```sh
sudo mount <server host>:<shared folder di server> <shared folder di client>

# Misal: sudo mount master:/home/mpiuser/cloud /home/mpiuser/cloud
```
Lakukan percobaan pembuatan file/folder di shared folder di salah satu komputer. File/folder tersebut seharusnya akan muncul di masing-masing komputer di folder yang sama.
## 5. MPI
### 5.1 Install MPI
> *Lakukan di **Master** dan **Worker***
```sh
sudo apt install openmpi-bin libopenmpi-dev
```
### 5.2 Install Library `mpi4py`
> *Lakukan di **Master** dan **Worker***

Install `mpi4py` dengan `pip`. Install package `python3-pip` jika belum memiliki `pip`.
```sh
sudo apt install python3-pip
pip install mpi4py
```
### 5.3 Testing MPI
> *Lakukan di **Master***

Buat dan edit file `test.py` di shared folder (`cloud`).
```sh
vim ~/cloud/test.py
```
Tabahkan baris berikut.
```py
from mpi4py import MPI

comm = MPI.COMM_WORLD
size = comm.Get_size()
worker = comm.Get_rank()

print("Hello world from worker", str(worker), "of", str(size))
```
Jalankan file `test.py` dengan MPI.
```sh
mpirun -np <jumlah prosesor> -host <daftar host> python3 test.py

# Misal: mpirun -np 3 -host master,worker1,worker2 python3 test.py
```
Output:
```sh
Hello world from worker 1 of 4
Hello world from worker 2 of 4
Hello world from worker 0 of 4
```
## Referensi
Tutorial untuk `mpi4py` bisa akses https://mpi4py.readthedocs.io/en/stable/tutorial.html

Referensi lebih lanjut:
- https://towardsdatascience.com/parallel-programming-in-python-with-message-passing-interface-mpi4py-551e3f198053
- https://www.geeksforgeeks.org/creating-an-mpi-cluster/








