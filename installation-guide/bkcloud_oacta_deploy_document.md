# Triển khai hệ thống OpenStack Ocata trên hạ tầng máy chủ bkcloud

## 1. Thiết lập hạ tầng và mô hình triển khai hệ thống

### Hạ tầng mạng

Hệ thống bkcloud OpenStack Ocata được triển khai trên 4 máy chủ bkcloud với hệ thống mạng kết nối gồm 3 mạng: 

- Mạng 10 Gbps 192.168.80.0/24 được sử dụng để làm mạng block storage - volume data network cho máy ảo, đồng thời sử dụng để triển khai các mạng private VLAN cho các project. 
- Mạng 1 Gbps 192.168.60.0/24 được sử dụng để triển khai mạng Flat provider network cấp truy cập Internet cho các máy ảo. 
- Mạng 100 Mbps 192.168.50.0/24 được sử dụng để làm mạng management network quản lý các máy tính vật lý cũng như được sử dụng để các service trên hệ thống OpenStack kết nối với nhau.

### Sơ đồ hạ tầng vật lý và sơ đồ triển khai các service của hệ thống OpenStack lên các máy chủ vật lý

Mô hình hạ tầng vật lý và sơ đồ triển khai các service của hệ thống OpenStack lên các máy chủ vật lý như sau: [bkcloud_infra_diagram.png](./images/bkcloud_infra_diagram.png)

![bkcloud_infra_diagram](./images/bkcloud_infra_diagram.png)

Trên mô hình này, controller node là máy chủ bkcloud 11.

Dải VLAN được sử dụng làm private network là 500-700. Do trên hệ thống sử dụng switch 10 Gbps Nexus 3548p để triển khai các mạng private VLAN này và triển khai mạng 192.168.80.0/24 để làm mạng volume data network cho máy ảo, nên chúng ta cần cấu hình switch để switch có thể vận chuyển các gói tin VLAN 500-700 và các gói tin của mạng 192.168.80.0/24  giữa các máy chủ vật lý trên hệ thống. Cách cấu hình switch 10 Gbps nằm trong tài liệu đi kèm tài liệu này.

### Địa chỉ ip và cấu hình mạng của các máy chủ vật lý

Địa chỉ ip và cấu hình mạng của các máy chủ vật lý trên hệ thống như sau:

- Máy chủ bkcloud11 - file ```/etc/network/interfaces```

```conf

source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback

auto eno2
iface eno2 inet static
address 192.168.60.11
netmask 255.255.255.0

auto eno3
iface eno3 inet static
address 192.168.50.11
netmask 255.255.255.0
gateway 192.168.50.1
dns-nameservers 8.8.8.8

auto eno4
iface eno4 inet static
address 192.168.60.10
netmask 255.255.255.0

auto enp27s0f1
iface enp27s0f1 inet static
address 192.168.80.11
netmask 255.255.255.0

```

- Máy chủ bkcloud11 - file ```/etc/hosts```

```conf

root@bkcloud11:~# cat /etc/hosts  | egrep -v "(^#.*|^$)"
127.0.0.1       localhost
127.0.1.1       bkcloud11
192.168.50.11 bkcloud11 controller
192.168.50.12 bkcloud12
192.168.50.13 bkcloud13
192.168.50.14 bkcloud14
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```

- Máy chủ bkcloud12 - file ```/etc/network/interfaces```

```conf

source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback

auto eno2
iface eno2 inet static
address 192.168.60.12
netmask 255.255.255.0

auto eno3
iface eno3 inet static
address 192.168.50.12
netmask 255.255.255.0
gateway 192.168.50.1
dns-nameservers 8.8.8.8

auto enp27s0f1
iface enp27s0f1 inet static
address 192.168.80.12
netmask 255.255.255.0

```

- Máy chủ bkcloud12 - file ```/etc/hosts```

```conf

root@bkcloud12:~# cat /etc/hosts  | egrep -v "(^#.*|^$)"
127.0.0.1       localhost
127.0.1.1       bkcloud12
192.168.50.11 bkcloud11 controller
192.168.50.12 bkcloud12
192.168.50.13 bkcloud13
192.168.50.14 bkcloud14
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```

- Máy chủ bkcloud13 - file ```/etc/network/interfaces```

```conf

source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback

auto enp11s0f0
iface enp11s0f0 inet static
address 192.168.60.13
netmask 255.255.255.0

auto enp11s0f1
iface enp11s0f1 inet static
address 192.168.50.13
netmask 255.255.255.0
gateway 192.168.50.1
dns-nameservers 8.8.8.8

auto ens2f0
iface ens2f0 inet static
address 192.168.80.13
netmask 255.255.255.0

```

- Máy chủ bkcloud13 - file ```/etc/hosts```

```conf

root@bkcloud13:~# cat /etc/hosts  | egrep -v "(^#.*|^$)"
127.0.0.1       localhost
127.0.1.1       bkcloud13
192.168.50.11 bkcloud11 controller
192.168.50.12 bkcloud12
192.168.50.13 bkcloud13
192.168.50.14 bkcloud14
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```

- Máy chủ bkcloud14 - file ```/etc/network/interfaces```

```conf

source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback

auto enp11s0f0
iface enp11s0f0 inet static
address 192.168.60.14
netmask 255.255.255.0

auto enp11s0f1
iface enp11s0f1 inet static
address 192.168.50.14
netmask 255.255.255.0
gateway 192.168.50.1
dns-nameservers 8.8.8.8

auto ens1f0
iface ens1f0 inet static
address 192.168.80.14
netmask 255.255.255.0

```

- Máy chủ bkcloud14 - file ```/etc/hosts```

```conf

root@bkcloud14:~# cat /etc/hosts  | egrep -v "(^#.*|^$)"
127.0.0.1       localhost
127.0.1.1       bkcloud14
192.168.50.11 bkcloud11 controller
192.168.50.12 bkcloud12
192.168.50.13 bkcloud13
192.168.50.14 bkcloud14
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```

### Cấu hình phân vùng ổ đĩa lưu trữ của các máy chủ vật lý

Tất cả các máy chủ vật lý đều được triển khai lvm. Cấu hình phân vùng bộ nhớ ngoài trên các máy chủ vật lý như sau:

- Máy chủ bkcloud11

```terminal

root@bkcloud11:~# fdisk -l
Disk /dev/sda: 1.8 TiB, 1999307276288 bytes, 3904897024 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: E7BABC10-F6FB-4B33-8C64-88569E11E3F3

Device          Start        End    Sectors   Size Type
/dev/sda1        2048    1050623    1048576   512M EFI System
/dev/sda2     1050624    2050047     999424   488M Linux filesystem
/dev/sda3     2050048  295018495  292968448 139.7G Linux LVM
/dev/sda4   295018496 1133879295  838860800   400G Linux filesystem
/dev/sda5  1133879296 3231031295 2097152000  1000G Linux filesystem
/dev/sda6  3231031296 3904896990  673865695 321.3G Linux filesystem

root@bkcloud11:~# pvs
  PV         VG                       Fmt  Attr PSize    PFree  
  /dev/sda3  ubuntu-vg                lvm2 a--   139.70g   9.31g
  /dev/sda4  ubuntu-vg                lvm2 a--   400.00g      0 
  /dev/sda5  cinder-volumes-bkcloud11 lvm2 a--  1000.00g 760.00g
  
root@bkcloud11:~# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda5
  VG Name               cinder-volumes-bkcloud11
  PV Size               1000.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              255999
  Free PE               194559
  Allocated PE          61440
  PV UUID               ZAUxRm-9VNu-RrlO-sNgx-zs0a-UgMe-TDPjtB
   
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               139.70 GiB / not usable 2.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              35762
  Free PE               2383
  Allocated PE          33379
  PV UUID               tJ0Vnu-73Gf-ZOcQ-WOQw-CMSE-usCG-rDJTZJ
   
  --- Physical volume ---
  PV Name               /dev/sda4
  VG Name               ubuntu-vg
  PV Size               400.00 GiB / not usable 4.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              102399
  Free PE               0
  Allocated PE          102399
  PV UUID               5c2hLv-c6u1-eKHI-aAkt-kpIb-irT6-1hrKws

root@bkcloud11:~# vgs
  VG                       #PV #LV #SN Attr   VSize    VFree  
  cinder-volumes-bkcloud11   1   3   0 wz--n- 1000.00g 760.00g
  ubuntu-vg                  2   3   0 wz--n-  539.69g   9.31g

root@bkcloud11:~# vgdisplay
  --- Volume group ---
  VG Name               cinder-volumes-bkcloud11
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  28
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               1000.00 GiB
  PE Size               4.00 MiB
  Total PE              255999
  Alloc PE / Size       61440 / 240.00 GiB
  Free  PE / Size       194559 / 760.00 GiB
  VG UUID               ExO4hv-KC4W-eYus-S1Xw-fU4V-gXBT-KR4LFF
   
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  18
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               539.69 GiB
  PE Size               4.00 MiB
  Total PE              138161
  Alloc PE / Size       135778 / 530.38 GiB
  Free  PE / Size       2383 / 9.31 GiB
  VG UUID               lAw8JU-UgTn-RZFA-jUb5-H6fh-899y-VlETYb

root@bkcloud11:~# lvs
  LV                                          VG                       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert                                                
  bkcloud11-glance-images                     ubuntu-vg                -wi-ao---- 400.00g                                                    
  bkcloud11-root                              ubuntu-vg                -wi-ao---- 111.76g                                                    
  bkcloud11-swap                              ubuntu-vg                -wi-ao----  18.62g                                                    
root@bkcloud11:~# lvdisplay
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/bkcloud11-root
  LV Name                bkcloud11-root
  VG Name                ubuntu-vg
  LV UUID                1x2Wpd-8cMx-7yaf-BPxO-Rc0A-Ph7l-8g0fL6
  LV Write Access        read/write
  LV Creation host, time bkcloud11, 2017-08-25 18:29:53 +0700
  LV Status              available
  # open                 1
  LV Size                111.76 GiB
  Current LE             28610
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/bkcloud11-swap
  LV Name                bkcloud11-swap
  VG Name                ubuntu-vg
  LV UUID                cRN36D-uooi-bxl3-MOr5-1dMZ-PgSn-U0iOjm
  LV Write Access        read/write
  LV Creation host, time bkcloud11, 2017-08-25 18:30:13 +0700
  LV Status              available
  # open                 2
  LV Size                18.62 GiB
  Current LE             4768
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1
   
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/bkcloud11-glance-images
  LV Name                bkcloud11-glance-images
  VG Name                ubuntu-vg
  LV UUID                PCFkkf-tvBr-oipK-qJI4-W9D8-0p3n-6HQ36A
  LV Write Access        read/write
  LV Creation host, time bkcloud11, 2017-08-25 20:46:05 +0700
  LV Status              available
  # open                 1
  LV Size                400.00 GiB
  Current LE             102400
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:2

```

- Máy chủ bkcloud12

```terminal

root@bkcloud12:~# fdisk -l
Disk /dev/sdb: 465.3 GiB, 499558383616 bytes, 975699968 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 4D43940E-A2DE-4FD0-A8D0-33C4CD8BEA5B

Device     Start       End   Sectors   Size Type
/dev/sdb1   2048 975699934 975697887 465.3G Linux LVM

Disk /dev/sda: 930.5 GiB, 999116767232 bytes, 1951399936 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 580442CB-8DF8-4F16-97F8-D867DD121533

Device         Start        End    Sectors   Size Type
/dev/sda1       2048    1050623    1048576   512M EFI System
/dev/sda2    1050624    2050047     999424   488M Linux filesystem
/dev/sda3    2050048  295018495  292968448 139.7G Linux LVM
/dev/sda4  295018496 1951399902 1656381407 789.8G Linux LVM
root@bkcloud12:~# 
root@bkcloud12:~# pvs
  PV         VG                       Fmt  Attr PSize   PFree  
  /dev/sda3  ubuntu-vg                lvm2 a--  139.70g   9.31g
  /dev/sda4  cinder-volumes-bkcloud12 lvm2 a--  789.82g 709.82g
root@bkcloud12:~# 
root@bkcloud12:~# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               139.70 GiB / not usable 2.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              35762
  Free PE               2384
  Allocated PE          33378
  PV UUID               eWOiod-Ppf4-3xeo-y8zc-l29j-sjsP-u7c6ZP
   
  --- Physical volume ---
  PV Name               /dev/sda4
  VG Name               cinder-volumes-bkcloud12
  PV Size               789.82 GiB / not usable 3.98 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              202194
  Free PE               181714
  Allocated PE          20480
  PV UUID               atGJhF-WroY-z5GW-tpop-Bpe3-11Hz-CPuOLS
root@bkcloud12:~# 
root@bkcloud12:~# vgs
  VG                       #PV #LV #SN Attr   VSize   VFree  
  cinder-volumes-bkcloud12   1   1   0 wz--n- 789.82g 709.82g
  ubuntu-vg                  1   2   0 wz--n- 139.70g   9.31g
root@bkcloud12:~# 
root@bkcloud12:~# vgdisplay 
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               139.70 GiB
  PE Size               4.00 MiB
  Total PE              35762
  Alloc PE / Size       33378 / 130.38 GiB
  Free  PE / Size       2384 / 9.31 GiB
  VG UUID               vmYL61-VKVR-NtBr-JVKb-00fw-kGec-xGFRkh
   
  --- Volume group ---
  VG Name               cinder-volumes-bkcloud12
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  14
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               789.82 GiB
  PE Size               4.00 MiB
  Total PE              202194
  Alloc PE / Size       20480 / 80.00 GiB
  Free  PE / Size       181714 / 709.82 GiB
  VG UUID               oWPOpa-EiC4-SQCr-6INz-nFqb-5T7A-2gfSmF
root@bkcloud12:~# 
root@bkcloud12:~# lvs
  LV                                          VG                       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert

  root                                        ubuntu-vg                -wi-ao---- 111.76g                                                    
  swap                                        ubuntu-vg                -wi-ao----  18.62g               
root@bkcloud12:~#                                    
root@bkcloud12:~# lvdisplay 
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/root
  LV Name                root
  VG Name                ubuntu-vg
  LV UUID                3XgPKY-sXbv-ysbG-Nkev-lWlo-LCHD-0aNivJ
  LV Write Access        read/write
  LV Creation host, time bkcloud12, 2017-08-26 18:30:48 +0700
  LV Status              available
  # open                 1
  LV Size                111.76 GiB
  Current LE             28610
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0
   
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/swap
  LV Name                swap
  VG Name                ubuntu-vg
  LV UUID                7KMW8s-12yT-3vWO-CBn3-MmXL-0euZ-0ctcY2
  LV Write Access        read/write
  LV Creation host, time bkcloud12, 2017-08-26 18:31:07 +0700
  LV Status              available
  # open                 2
  LV Size                18.62 GiB
  Current LE             4768
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1
   
```

- Máy chủ bkcloud 13 và máy chủ bkcloud 14: Chưa triển khai.

## 2. Triển khai, thiết lập hệ thống và các câu lệnh được sử dụng để triển khai OpenStack Ocata trên hạ tầng vật lý

Phần này sẽ trình bày cấu hình của các service trên các máy chủ vật lý ở sở đồ triển khai và một số câu lệnh cấu hình đã được sử dụng

Hướng dẫn cài đặt được sử dụng để thiết lập hệ thống:

[https://docs.openstack.org/ocata/install-guide-ubuntu/environment-sql-database.html](https://docs.openstack.org/ocata/install-guide-ubuntu/environment-sql-database.html)

Quy ước chung password cho mọi service là: xxxx

### Cấu hình máy chủ bkcloud11

#### Cấu hình chrony - NTP service bkcloud11

- Cài đặt các package: ```# apt install chrony```
- File ```/etc/chrony/chrony.conf```:

```conf

root@bkcloud11:~# cat /etc/chrony/chrony.conf | egrep -v "(^#.*|^$)"
server vn.pool.ntp.org iburst
keyfile /etc/chrony/chrony.keys
commandkey 1
driftfile /var/lib/chrony/chrony.drift
log tracking measurements statistics
logdir /var/log/chrony
maxupdateskew 100.0
dumponexit
dumpdir /var/lib/chrony
allow 192.168.50.0/24
logchange 0.5
hwclockfile /etc/adjtime
rtcsync

```

#### Enable the OpenStack repository & install OpenStack Client bkcloud11

```conf
# apt install software-properties-common
# add-apt-repository cloud-archive:ocata
# apt update && apt dist-upgrade
# apt install python-openstackclient
```

#### Cấu hình database - mariadb service bkcloud11

- Cài đặt các package: ```# apt install mariadb-server python-pymysql```
- Password config - database password: xxxx
- File ```/etc/mysql/mariadb.conf.d/99-openstack.cnf```

```conf
root@bkcloud11:~# cat /etc/mysql/mariadb.conf.d/99-openstack.cnf | egrep -v "(^#.*|^$)"
[mysqld]
bind-address = 192.168.50.11
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

#### Cấu hình message queue - rabbitmq-server service bkcloud11

- Cài đặt các package: ```# apt install rabbitmq-server```
- Thêm user vào rabbitmq-server service và cấp quyền cho user openstack:

```conf
# rabbitmqctl add_user openstack xxxx
# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

#### Cấu hình memcached service bkcloud11

- Cài đặt các package: ```# apt install memcached python-memcache```
- File ```/etc/memcached.conf```:

```conf
root@bkcloud11:~# cat /etc/memcached.conf | egrep -v "(^#.*|^$)"
-d
logfile /var/log/memcached.log
-m 64
-p 11211
-u memcache
-l 192.168.50.11
```

#### Cấu hình Identity service - Keystone bkcloud11

- Cấu hình database

```conf

# mysql

CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY 'xxxx';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'xxxx';

```

- Cài đặt các package: ```# apt install keystone```

- File ```/etc/keystone/keystone.conf```

```conf

[DEFAULT]
[assignment]
[auth]
[cache]
[catalog]
[cors]
[cors.subdomain]
[credential]
[database]
connection = mysql+pymysql://keystone:xxxx@controller/keystone
[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[extra_headers]
[federation]
[fernet_tokens]
[healthcheck]
[identity]
[identity_mapping]
[kvs]
[ldap]
[matchmaker_redis]
[memcache]

```

- Populate the Identity service database:

```bash

# su -s /bin/sh -c "keystone-manage db_sync" keystone

```

- Initialize Fernet key repositories:

```bash

# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

```

- Bootstrap the Identity service:

```bash

 #keystone-manage bootstrap --bootstrap-password xxxx \
  --bootstrap-admin-url http://controller:35357/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne


```

- File /etc/apache2/apache2.conf

```conf

root@bkcloud11:~# cat /etc/apache2/apache2.conf | egrep -v "(^#.*|^$)"
ServerName controller
Mutex file:${APACHE_LOCK_DIR} default
PidFile ${APACHE_PID_FILE}
Timeout 300
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 5
User ${APACHE_RUN_USER}
Group ${APACHE_RUN_GROUP}
HostnameLookups Off
ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn
IncludeOptional mods-enabled/*.load
IncludeOptional mods-enabled/*.conf
Include ports.conf
<Directory />
        Options FollowSymLinks
        AllowOverride None
        Require all denied
</Directory>
<Directory /usr/share>
        AllowOverride None
        Require all granted
</Directory>
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>
AccessFileName .htaccess
<FilesMatch "^\.ht">
        Require all denied
</FilesMatch>
LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %O" common
LogFormat "%{Referer}i -> %U" referer
LogFormat "%{User-agent}i" agent
IncludeOptional conf-enabled/*.conf
IncludeOptional sites-enabled/*.conf

```

- Restart the Apache service: 

```bash

# service apache2 restart
# rm -f /var/lib/keystone/keystone.db

```

- File```admin.sh```

```bash

root@bkcloud11:~# cat /home/bkcloud11/admin.sh 
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=xxxx
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2

```

- Tạo các a domain, projects, users, and roles:

```bash

$. admin.sh
$ openstack project create --domain default \
  --description "Service Project" service
$ openstack project create --domain default \
  --description "Demo Project" demo
$ openstack user create --domain default \
  --password-prompt iot
$ openstack role create user
$ openstack role add --project demo --user demo user

```

#### Cấu hình Image service - Glance bkcloud11

- Cấu hình dataabase:

```conf

# mysql

CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY 'xxxx';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY 'xxxx';

```

- Cấu hình endpoint, user, role, service:

```conf

openstack user create --domain default --password xxxx glance

openstack role add --project service --user glance admin

openstack service create --name glance \
  --description "OpenStack Image" image

openstack endpoint create --region RegionOne \
  image public http://controller:9292

openstack endpoint create --region RegionOne \
  image internal http://controller:9292

openstack endpoint create --region RegionOne \
  image admin http://controller:9292

```

- Cài đặt các package: ```# apt install glance```
- File ```/etc/glance/glance-api.conf```

```conf

root@bkcloud11:~# cat /etc/glance/glance-api.conf | egrep -v "(^#.*|^$)"
[DEFAULT]
[cors]
[DEFAULT]
[cors]
[cors.subdomain]
[database]
connection = mysql+pymysql://glance:xxxx@controller/glance
backend = sqlalchemy
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /mnt/glance
[image_format]
disk_formats = ami,ari,aki,vhd,vhdx,vmdk,raw,qcow2,vdi,iso,ploop.root-tar
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = xxxx
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
[store_type_location_strategy]
[task]
[taskflow_executor]
```

- File ```/etc/glance/glance-registry.conf```:

```conf

root@bkcloud11:~# cat /etc/glance/glance-registry.conf | egrep -v "(^#.*|^$)"
[DEFAULT]
[database]
connection = mysql+pymysql://glance:xxxx@controller/glance
backend = sqlalchemy
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = xxxx
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]

```

- Populate the Image service database:

```bash

# su -s /bin/sh -c "glance-manage db_sync" glance

```

- Restart the Image services:

```bash

# service glance-registry restart
# service glance-api restart

```

#### Cấu hình Compute service - Nova bkcloud11

- Cấu hình dataabase:

```conf

# mysql

CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;

# Grant proper access to the databases:

GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'xxxx';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY 'xxxx';

GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'xxxx';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY 'xxxx';

GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'xxxx';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
  IDENTIFIED BY 'xxxx';

```

- Cấu hình endpoint, user, role, service:

```conf

openstack user create --domain default --password xxxx nova

openstack role add --project service --user nova admin

openstack service create --name nova \
  --description "OpenStack Compute" compute


openstack endpoint create --region RegionOne \
  compute public http://controller:8774/v2.1


openstack endpoint create --region RegionOne \
  compute internal http://controller:8774/v2.1


openstack endpoint create --region RegionOne \
  compute admin http://controller:8774/v2.1


openstack user create --domain default --password xxxx placement
openstack role add --project service --user placement admin

openstack service create --name placement --description "Placement API" placement

openstack endpoint create --region RegionOne placement public http://controller:8778

openstack endpoint create --region RegionOne placement internal http://controller:8778

openstack endpoint create --region RegionOne placement admin http://controller:8778

```

- Cài đặt các package: ```# apt install nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler nova-placement-api nova-compute```

- File ```/etc/nova/nova.conf```:

```conf

root@bkcloud11:~# cat /etc/nova/nova.conf | egrep -v "(^#.*|^$)"
[DEFAULT]
transport_url = rabbit://openstack:xxxx@controller
my_ip = 192.168.50.11
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
force_dhcp_release=true
state_path=/var/lib/nova
enabled_apis=osapi_compute,metadata
log_dir=/var/log/nova
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:xxxx@controller/nova_api
[barbican]
[cache]
[cells]
enable=False
[cinder]
os_region_name = RegionOne
[cloudpipe]
[conductor]
[console]
[consoleauth]
[cors]
[cors.subdomain]
[crypto]
[database]
connection = mysql+pymysql://nova:xxxx@controller/nova
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://controller:9292
[guestfs]
[healthcheck]
[hyperv]
[image_file_url]
[ironic]
[key_manager]
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = xxxx
[libvirt]
[matchmaker_redis]
[metrics]
[mks]
[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = xxxx
service_metadata_proxy = true
metadata_proxy_shared_secret = xxxx
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
lock_path=/var/lock/nova
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:35357/v3
username = placement
password = xxxx
[quota]
[rdp]
[remote_debug]
[scheduler]
[serial_console]
[service_user]
[spice]
[ssl]
[trusted_computing]
[upgrade_levels]
[vendordata_dynamic_auth]
[vmware]
[vnc]
enabled = true
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://192.168.60.10:6080/vnc_auto.html
[workarounds]
[wsgi]
api_paste_config=/etc/nova/api-paste.ini
[xenserver]
[xvp]

```

- Populate the nova-api database:

```bash

# su -s /bin/sh -c "nova-manage api_db sync" nova

```

- Register the cell0 database:

```bash

# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova

```

- Create the cell1 cell:

```bash

# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova

```

- Populate the nova database:

```bash

# su -s /bin/sh -c "nova-manage db sync" nova

```

- Verify nova cell0 and cell1 are registered correctly:

```bash

# nova-manage cell_v2 list_cells

```

- Restart the Compute services:

```bash

service nova-api restart
service nova-consoleauth restart
service nova-scheduler restart
service nova-conductor restart
service nova-novncproxy restart
service nova-compute restart

```

- Run commands on the controller node - discover compute hosts:

```bash

# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

```

#### Cấu hình Networking service - Neutron bkcloud11

- Cấu hình dataabase:

```conf
# mysql

CREATE DATABASE neutron;

GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
  IDENTIFIED BY 'xxxx';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
  IDENTIFIED BY 'xxxx';


```

- Cấu hình endpoint, user, role, service:

```conf

openstack user create --domain default --password xxxx neutron
openstack role add --project service --user neutron admin

openstack service create --name neutron \
  --description "OpenStack Networking" network

openstack endpoint create --region RegionOne \
  network public http://controller:9696

openstack endpoint create --region RegionOne \
  network internal http://controller:9696

openstack endpoint create --region RegionOne \
  network admin http://controller:9696


```

- Cài đặt các package: ```# apt install neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent```

- File ```/etc/neutron/neutron.conf```:

```conf

root@bkcloud11:~# cat /etc/neutron/neutron.conf | egrep -v "(^#.*|^$)"
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:xxxx@controller
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
core_plugin = ml2
[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
[cors]
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
core_plugin = ml2
[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
[cors]
[cors.subdomain]
[database]
connection = mysql+pymysql://neutron:xxxx@controller/neutron
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = xxxx

[matchmaker_redis]
[nova]
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = xxxx
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[qos]
[quotas]
[ssl]

```

- File ```/etc/neutron/plugins/ml2/ml2_conf.ini```:

```conf

root@bkcloud11:~# cat /etc/neutron/plugins/ml2/ml2_conf.ini | egrep -v "(^#.*|^$)"                                                                           
[DEFAULT]
[ml2]
type_drivers = flat,vlan
tenant_network_types = vlan
mechanism_drivers = linuxbridge
extension_drivers = port_security
[ml2_type_flat]
flat_networks = provider
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
network_vlan_ranges = private_vlan:500:700
[ml2_type_vxlan]
[securitygroup]
enable_ipset = true

```

- File ```/etc/neutron/plugins/ml2/linuxbridge_agent.ini```:

```conf

root@bkcloud11:~# cat /etc/neutron/plugins/ml2/linuxbridge_agent.ini | egrep -v "(^#.*|^$)"
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:eno2, private_vlan:enp27s0f1
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = false

```

- File ```/etc/neutron/l3_agent.ini```:

```conf

root@bkcloud11:~# cat /etc/neutron/l3_agent.ini| egrep -v "(^#.*|^$)"
[DEFAULT]
interface_driver = linuxbridge
[agent]
[ovs]


```

- File ```/etc/neutron/dhcp_agent.ini```:

```conf

root@bkcloud11:~# cat /etc/neutron/dhcp_agent.ini | egrep -v "(^#.*|^$)"
[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True
[agent]
[ovs]

```

- File ```/etc/neutron/metadata_agent.ini```:

```conf

root@bkcloud11:~# cat /etc/neutron/metadata_agent.ini | egrep -v "(^#.*|^$)"
[DEFAULT]
nova_metadata_ip = controller
metadata_proxy_shared_secret = xxxx
[agent]
[cache]

```

- Remember: You must check Configure the Compute service to use the Networking service in file ```/etc/nova/nova.conf```

```conf

[neutron]
# ...
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = xxxx
service_metadata_proxy = true
metadata_proxy_shared_secret = xxxx

```

- Populate the database:

```conf

# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron

```

- Restart services:

```conf

service nova-api restart
service nova-compute restart

service neutron-server restart
service neutron-linuxbridge-agent restart
service neutron-dhcp-agent restart
service neutron-metadata-agent restart
service neutron-l3-agent restart

```

#### Cấu hình Dashboard service - Horizon bkcloud11

<https://docs.openstack.org/horizon/latest/contributor/quickstart.html>

- Cài đặt các gói cần thiết 

```bash

# sudo apt-get install git python-pip
git clone https://git.openstack.org/openstack/horizon
cd horizon
cp openstack_dashboard/local/local_settings.py.example openstack_dashboard/local/local_settings.py
pip install tox
. .tox/runserver/bin/activate
pip install python-memcached
deactivate
```

- File ```openstack_dashboard/local/local_settings.py```:

```py

import os
from django.utils.translation import ugettext_lazy as _
from horizon.utils import secret_key
from openstack_dashboard.settings import HORIZON_CONFIG
DEBUG = True
WEBROOT = '/'
ALLOWED_HOSTS = ['*', ]
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211',
    }
}
OPENSTACK_HOST = "controller"
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
TIME_ZONE = "Asia/Ho_Chi_Minh"
LOCAL_PATH = os.path.dirname(os.path.abspath(__file__))
SECRET_KEY = secret_key.generate_or_read_from_file(
    os.path.join(LOCAL_PATH, '.secret_key_store'))
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
OPENSTACK_KEYSTONE_BACKEND = {
    'name': 'native',
    'can_edit_user': True,
    'can_edit_group': True,
    'can_edit_project': True,
    'can_edit_domain': True,
    'can_edit_role': True,
}
OPENSTACK_HYPERVISOR_FEATURES = {
    'can_set_mount_point': False,
    'can_set_password': False,
    'requires_keypair': False,
    'enable_quotas': True
}
OPENSTACK_CINDER_FEATURES = {
    'enable_backup': False,
}
OPENSTACK_NEUTRON_NETWORK = {
    'enable_router': True,
    'enable_quotas': True,
    'enable_ipv6': True,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_lb': True,
    'enable_firewall': True,
    'enable_vpn': True,
    'enable_fip_topology_check': True,
    # Default dns servers you would like to use when a subnet is
    # created.  This is only a default, users can still choose a different
    # list of dns servers when creating a new subnet.
    # The entries below are examples only, and are not appropriate for
    # real deployments
    # 'default_dns_nameservers': ["8.8.8.8", "8.8.4.4", "208.67.222.222"],
    # The profile_support option is used to detect if an external router can be
    # configured via the dashboard. When using specific plugins the
    # profile_support can be turned on if needed.
    'profile_support': None,
    #'profile_support': 'cisco',
    # Set which provider network types are supported. Only the network types
    # in this list will be available to choose from when creating a network.
    # Network types include local, flat, vlan, gre, vxlan and geneve.
    # 'supported_provider_types': ['*'],
    # You can configure available segmentation ID range per network type
    # in your deployment.
    # 'segmentation_id_range': {
    #     'vlan': [1024, 2048],
    #     'vxlan': [4094, 65536],
    # },
    # You can define additional provider network types here.
    # 'extra_provider_types': {
    #     'awesome_type': {
    #         'display_name': 'Awesome New Type',
    #         'require_physical_network': False,
    #         'require_segmentation_id': True,
    #     }
    # },
    # Set which VNIC types are supported for port binding. Only the VNIC
    # types in this list will be available to choose from when creating a
    # port.
    # VNIC types include 'normal', 'macvtap' and 'direct'.
    # Set to empty list or None to disable VNIC type selection.
    'supported_vnic_types': ['*'],
}
OPENSTACK_HEAT_STACK = {
    'enable_user_pass': True,
}
IMAGE_CUSTOM_PROPERTY_TITLES = {
    "architecture": _("Architecture"),
    "kernel_id": _("Kernel ID"),
    "ramdisk_id": _("Ramdisk ID"),
    "image_state": _("Euca2ools state"),
    "project_id": _("Project ID"),
    "image_type": _("Image Type"),
}
IMAGE_RESERVED_CUSTOM_PROPERTIES = []
API_RESULT_LIMIT = 1000
API_RESULT_PAGE_SIZE = 20
SWIFT_FILE_TRANSFER_CHUNK_SIZE = 512 * 1024
INSTANCE_LOG_LENGTH = 35
DROPDOWN_MAX_ITEMS = 30
LOGGING = {
    'version': 1,
    # When set to True this will disable all logging except
    # for loggers specified in this configuration dictionary. Note that
    # if nothing is specified here and disable_existing_loggers is True,
    # django.db.backends will still log unless it is disabled explicitly.
    'disable_existing_loggers': False,
    'formatters': {
        'operation': {
            # The format of "%(message)s" is defined by
            # OPERATION_LOG_OPTIONS['format']
            'format': '%(asctime)s %(message)s'
        },
    },
    'handlers': {
        'null': {
            'level': 'DEBUG',
            'class': 'logging.NullHandler',
        },
        'console': {
            # Set the level to "DEBUG" for verbose output logging.
            'level': 'INFO',
            'class': 'logging.StreamHandler',
        },
        'operation': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'operation',
        },
    },
    'loggers': {
        # Logging from django.db.backends is VERY verbose, send to null
        # by default.
        'django.db.backends': {
            'handlers': ['null'],
            'propagate': False,
        },
        'requests': {
            'handlers': ['null'],
            'propagate': False,
        },
        'horizon': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'horizon.operation_log': {
            'handlers': ['operation'],
            'level': 'INFO',
            'propagate': False,
        },
        'openstack_dashboard': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'novaclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'cinderclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'keystoneclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'glanceclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'neutronclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'heatclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'swiftclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'openstack_auth': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'nose.plugins.manager': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'django': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'iso8601': {
            'handlers': ['null'],
            'propagate': False,
        },
        'scss': {
            'handlers': ['null'],
            'propagate': False,
        },
    },
}
SECURITY_GROUP_RULES = {
    'all_tcp': {
        'name': _('All TCP'),
        'ip_protocol': 'tcp',
        'from_port': '1',
        'to_port': '65535',
    },
    'all_udp': {
        'name': _('All UDP'),
        'ip_protocol': 'udp',
        'from_port': '1',
        'to_port': '65535',
    },
    'all_icmp': {
        'name': _('All ICMP'),
        'ip_protocol': 'icmp',
        'from_port': '-1',
        'to_port': '-1',
    },
    'ssh': {
        'name': 'SSH',
        'ip_protocol': 'tcp',
        'from_port': '22',
        'to_port': '22',
    },
    'smtp': {
        'name': 'SMTP',
        'ip_protocol': 'tcp',
        'from_port': '25',
        'to_port': '25',
    },
    'dns': {
        'name': 'DNS',
        'ip_protocol': 'tcp',
        'from_port': '53',
        'to_port': '53',
    },
    'http': {
        'name': 'HTTP',
        'ip_protocol': 'tcp',
        'from_port': '80',
        'to_port': '80',
    },
    'pop3': {
        'name': 'POP3',
        'ip_protocol': 'tcp',
        'from_port': '110',
        'to_port': '110',
    },
    'imap': {
        'name': 'IMAP',
        'ip_protocol': 'tcp',
        'from_port': '143',
        'to_port': '143',
    },
    'ldap': {
        'name': 'LDAP',
        'ip_protocol': 'tcp',
        'from_port': '389',
        'to_port': '389',
    },
    'https': {
        'name': 'HTTPS',
        'ip_protocol': 'tcp',
        'from_port': '443',
        'to_port': '443',
    },
    'smtps': {
        'name': 'SMTPS',
        'ip_protocol': 'tcp',
        'from_port': '465',
        'to_port': '465',
    },
    'imaps': {
        'name': 'IMAPS',
        'ip_protocol': 'tcp',
        'from_port': '993',
        'to_port': '993',
    },
    'pop3s': {
        'name': 'POP3S',
        'ip_protocol': 'tcp',
        'from_port': '995',
        'to_port': '995',
    },
    'ms_sql': {
        'name': 'MS SQL',
        'ip_protocol': 'tcp',
        'from_port': '1433',
        'to_port': '1433',
    },
    'mysql': {
        'name': 'MYSQL',
        'ip_protocol': 'tcp',
        'from_port': '3306',
        'to_port': '3306',
    },
    'rdp': {
        'name': 'RDP',
        'ip_protocol': 'tcp',
        'from_port': '3389',
        'to_port': '3389',
    },
}
REST_API_REQUIRED_SETTINGS = ['OPENSTACK_HYPERVISOR_FEATURES',
                              'LAUNCH_INSTANCE_DEFAULTS',
                              'OPENSTACK_IMAGE_FORMATS',
                              'OPENSTACK_KEYSTONE_DEFAULT_DOMAIN']
ALLOWED_PRIVATE_SUBNET_CIDR = {'ipv4': [], 'ipv6': []}

```

- To start the Horizon development server use the command below:

```bash

tox -e runserver 0.0.0.0:8000

```

#### Cấu hình Block Storage service - Cinder bkcloud11

- Cấu hình database

```conf

# mysql

CREATE DATABASE cinder;

GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' \
  IDENTIFIED BY 'xxxx';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
  IDENTIFIED BY 'xxxx';

```

- Cấu hình endpoint, user, role, service:

```conf

openstack user create --domain default --password xxxx cinder
openstack role add --project service --user cinder admin

openstack service create --name cinderv2 \
      --description "OpenStack Block Storage" volumev2
openstack service create --name cinderv3 \
      --description "OpenStack Block Storage" volumev3
openstack endpoint create --region RegionOne \
  volumev2 public http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev2 internal http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev2 admin http://controller:8776/v2/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev3 public http://controller:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev3 internal http://controller:8776/v3/%\(project_id\)s
openstack endpoint create --region RegionOne \
  volumev3 admin http://controller:8776/v3/%\(project_id\)s
  
```

- Cài đặt các package: ```# apt install cinder-api cinder-scheduler cinder-volume```
- Kiểm tra thiết lập phân vùng vật lý và thiết lập lvm của máy chủ vật lý:

```conf

root@bkcloud11:~# fdisk -l
Disk /dev/sda: 1.8 TiB, 1999307276288 bytes, 3904897024 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: E7BABC10-F6FB-4B33-8C64-88569E11E3F3

Device          Start        End    Sectors   Size Type
/dev/sda1        2048    1050623    1048576   512M EFI System
/dev/sda2     1050624    2050047     999424   488M Linux filesystem
/dev/sda3     2050048  295018495  292968448 139.7G Linux LVM
/dev/sda4   295018496 1133879295  838860800   400G Linux filesystem
/dev/sda5  1133879296 3231031295 2097152000  1000G Linux filesystem
/dev/sda6  3231031296 3904896990  673865695 321.3G Linux filesystem

root@bkcloud11:~# pvs
  PV         VG                       Fmt  Attr PSize    PFree  
  /dev/sda3  ubuntu-vg                lvm2 a--   139.70g   9.31g
  /dev/sda4  ubuntu-vg                lvm2 a--   400.00g      0 
  /dev/sda5  cinder-volumes-bkcloud11 lvm2 a--  1000.00g 760.00g
  
root@bkcloud11:~# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda5
  VG Name               cinder-volumes-bkcloud11
  PV Size               1000.00 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              255999
  Free PE               194559
  Allocated PE          61440
  PV UUID               ZAUxRm-9VNu-RrlO-sNgx-zs0a-UgMe-TDPjtB
   
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               139.70 GiB / not usable 2.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              35762
  Free PE               2383
  Allocated PE          33379
  PV UUID               tJ0Vnu-73Gf-ZOcQ-WOQw-CMSE-usCG-rDJTZJ
   
  --- Physical volume ---
  PV Name               /dev/sda4
  VG Name               ubuntu-vg
  PV Size               400.00 GiB / not usable 4.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              102399
  Free PE               0
  Allocated PE          102399
  PV UUID               5c2hLv-c6u1-eKHI-aAkt-kpIb-irT6-1hrKws

root@bkcloud11:~# vgs
  VG                       #PV #LV #SN Attr   VSize    VFree  
  cinder-volumes-bkcloud11   1   3   0 wz--n- 1000.00g 760.00g
  ubuntu-vg                  2   3   0 wz--n-  539.69g   9.31g

root@bkcloud11:~# vgdisplay
  --- Volume group ---
  VG Name               cinder-volumes-bkcloud11
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  28
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               1000.00 GiB
  PE Size               4.00 MiB
  Total PE              255999
  Alloc PE / Size       61440 / 240.00 GiB
  Free  PE / Size       194559 / 760.00 GiB
  VG UUID               ExO4hv-KC4W-eYus-S1Xw-fU4V-gXBT-KR4LFF
   
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  18
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               539.69 GiB
  PE Size               4.00 MiB
  Total PE              138161
  Alloc PE / Size       135778 / 530.38 GiB
  Free  PE / Size       2383 / 9.31 GiB
  VG UUID               lAw8JU-UgTn-RZFA-jUb5-H6fh-899y-VlETYb

root@bkcloud11:~# lvs
  LV                                          VG                       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert                                                
  bkcloud11-glance-images                     ubuntu-vg                -wi-ao---- 400.00g                                                    
  bkcloud11-root                              ubuntu-vg                -wi-ao---- 111.76g                                                    
  bkcloud11-swap                              ubuntu-vg                -wi-ao----  18.62g                                                    
root@bkcloud11:~# lvdisplay
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/bkcloud11-root
  LV Name                bkcloud11-root
  VG Name                ubuntu-vg
  LV UUID                1x2Wpd-8cMx-7yaf-BPxO-Rc0A-Ph7l-8g0fL6
  LV Write Access        read/write
  LV Creation host, time bkcloud11, 2017-08-25 18:29:53 +0700
  LV Status              available
  # open                 1
  LV Size                111.76 GiB
  Current LE             28610
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/bkcloud11-swap
  LV Name                bkcloud11-swap
  VG Name                ubuntu-vg
  LV UUID                cRN36D-uooi-bxl3-MOr5-1dMZ-PgSn-U0iOjm
  LV Write Access        read/write
  LV Creation host, time bkcloud11, 2017-08-25 18:30:13 +0700
  LV Status              available
  # open                 2
  LV Size                18.62 GiB
  Current LE             4768
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1
   
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/bkcloud11-glance-images
  LV Name                bkcloud11-glance-images
  VG Name                ubuntu-vg
  LV UUID                PCFkkf-tvBr-oipK-qJI4-W9D8-0p3n-6HQ36A
  LV Write Access        read/write
  LV Creation host, time bkcloud11, 2017-08-25 20:46:05 +0700
  LV Status              available
  # open                 1
  LV Size                400.00 GiB
  Current LE             102400
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:2

```

- Thiết lập lvm filter để ngăn LVM của máy chủ vật lý scan và add cache các volume của các máy ảo. LVM filter này sẽ chỉ cho phép LVM của máy chủ vật lý scan các phân vùng vật lý cần thiết:
- File ```/etc/lvm/lvm.conf```

```conf

devices {
...
        filter = [  "a/sda1/", "a/sda2/", "a/sda3/","a/sda4/","a/sda5/", "r/.*/"]
...
}

```

- File ```/etc/cinder/cinder.conf```

```conf

root@bkcloud11:~# cat /etc/cinder/cinder.conf | egrep -v "(^#.*|^$)"
[DEFAULT]
rootwrap_config = /etc/cinder/rootwrap.conf
api_paste_confg = /etc/cinder/api-paste.ini
iscsi_helper = tgtadm
volume_name_template = volume-%s
verbose = True
state_path = /var/lib/cinder
lock_path = /var/lock/cinder
volumes_dir = /var/lib/cinder/volumes
transport_url = rabbit://openstack:xxxx@controller
auth_strategy = keystone
my_ip = 192.168.50.11
enabled_backends = lvm-bkcloud11
glance_api_servers = http://controller:9292
[database]
connection = mysql+pymysql://cinder:xxxx@controller/cinder
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = xxxx
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
[lvm-bkcloud11]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes-bkcloud11
iscsi_protocol = iscsi
iscsi_helper = tgtadm
iscsi_ip_address = 192.168.80.11

```

- Populate the Block Storage database:

```bash
su -s /bin/sh -c "cinder-manage db sync" cinder
```

- Configure Compute to use Block Storage - chỉnh sửa file ```/etc/nova/nova.conf``` trên controller:

```conf
[cinder]
os_region_name = RegionOne
```

- Restart the Compute API service and Block Storage Services:

```bash
service nova-api restart
service cinder-scheduler restart
service apache2 restart
service tgt restart
service cinder-volume restart
```

### Cấu hình máy chủ bkcloud12

#### Cấu hình chrony - NTP service bkcloud12

- Cài đặt các package: ```# apt install chrony```
- File ```/etc/chrony/chrony.conf```:

```conf

root@bkcloud12:~# cat /etc/chrony/chrony.conf | egrep -v "(^#.*|^$)"
server controller iburst
keyfile /etc/chrony/chrony.keys
commandkey 1
driftfile /var/lib/chrony/chrony.drift
log tracking measurements statistics
logdir /var/log/chrony
maxupdateskew 100.0
dumponexit
dumpdir /var/lib/chrony
logchange 0.5
hwclockfile /etc/adjtime
rtcsync

```

#### Enable the OpenStack repository & install OpenStack Client bkcloud12

```conf
# apt install software-properties-common
# add-apt-repository cloud-archive:ocata
# apt update && apt dist-upgrade
# apt install python-openstackclient
```

#### Cấu hình Compute service - Nova bkcloud12

- Cài đặt các package: ```# apt install nova-compute```

- File ```/etc/nova/nova.conf```:

```conf

root@bkcloud12:~# cat /etc/nova/nova.conf | egrep -v "(^#.*|^$)"
[DEFAULT]
transport_url = rabbit://openstack:xxxx@controller
my_ip = 192.168.50.12
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
force_dhcp_release=true
state_path=/var/lib/nova
enabled_apis=osapi_compute,metadata
log_dir=/var/log/nova
[api]
auth_strategy = keystone
[api_database]
connection = mysql+pymysql://nova:xxxx@controller/nova_api
[barbican]
[cache]
[cells]
enable=False
[cinder]
os_region_name = RegionOne
[cloudpipe]
[conductor]
[console]
[consoleauth]
[cors]
[cors.subdomain]
[crypto]
[database]
[ephemeral_storage_encryption]
[filter_scheduler]
[glance]
api_servers = http://controller:9292
[guestfs]
[healthcheck]
[hyperv]
[image_file_url]
[ironic]
[key_manager]
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = xxxx
[libvirt]
[matchmaker_redis]
[metrics]
[mks]
[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = xxxx
[notifications]
[osapi_v21]
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[pci]
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:35357/v3
username = placement
password = xxxx
[quota]
[rdp]
[remote_debug]
[scheduler]
[serial_console]
[service_user]
[spice]
[ssl]
[trusted_computing]
[upgrade_levels]
[vendordata_dynamic_auth]
[vmware]
[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://192.168.60.11:6080/vnc_auto.html
[workarounds]
[wsgi]
api_paste_config=/etc/nova/api-paste.ini
[xenserver]
[xvp]


```

- Restart the Compute service:

```bash

service nova-compute restart

```

- Run commands on the controller node - discover compute hosts:

```bash

# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

```

#### Cấu hình Networking service - Neutron bkcloud12

- Cài đặt các package: ```# apt install neutron-linuxbridge-agent```

- File ```/etc/neutron/neutron.conf```:

```conf

root@bkcloud12:~# cat /etc/neutron/neutron.conf | egrep -v "(^#.*|^$)"
[DEFAULT]
transport_url = rabbit://openstack:xxxx@controller
auth_strategy = keystone
[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
[cors]
[cors.subdomain]
[database]
connection = sqlite:////var/lib/neutron/neutron.sqlite
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = xxxx
[matchmaker_redis]
[nova]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_kafka]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[qos]
[quotas]
[ssl]

```

- File ```/etc/neutron/plugins/ml2/linuxbridge_agent.ini```:

```conf

root@bkcloud12:~# cat /etc/neutron/plugins/ml2/linuxbridge_agent.ini | egrep -v "(^#.*|^$)"
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:eno2, private_vlan:enp27s0f1
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = false

```

- Remember: You must check Configure the Compute service to use the Networking service in file ```/etc/nova/nova.conf```

```conf

[neutron]
# ...
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = xxxx

```

- Restart services:

```conf

service nova-compute restart
service neutron-linuxbridge-agent restart

```

#### Cấu hình Dashboard service - Horizon bkcloud12

<https://docs.openstack.org/horizon/latest/contributor/quickstart.html>

- Cài đặt các gói cần thiết 

```bash

# sudo apt-get install git python-pip
git clone https://git.openstack.org/openstack/horizon
cd horizon
cp openstack_dashboard/local/local_settings.py.example openstack_dashboard/local/local_settings.py
pip install tox
. .tox/runserver/bin/activate
pip install python-memcached
deactivate
```

- File ```openstack_dashboard/local/local_settings.py```:

```py

import os
from django.utils.translation import ugettext_lazy as _
from horizon.utils import secret_key
from openstack_dashboard.settings import HORIZON_CONFIG
DEBUG = True
WEBROOT = '/'
ALLOWED_HOSTS = ['*', ]
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'Default'
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211',
    }
}
OPENSTACK_HOST = "controller"
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
TIME_ZONE = "Asia/Ho_Chi_Minh"
LOCAL_PATH = os.path.dirname(os.path.abspath(__file__))
SECRET_KEY = secret_key.generate_or_read_from_file(
    os.path.join(LOCAL_PATH, '.secret_key_store'))
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'
OPENSTACK_KEYSTONE_BACKEND = {
    'name': 'native',
    'can_edit_user': True,
    'can_edit_group': True,
    'can_edit_project': True,
    'can_edit_domain': True,
    'can_edit_role': True,
}
OPENSTACK_HYPERVISOR_FEATURES = {
    'can_set_mount_point': False,
    'can_set_password': False,
    'requires_keypair': False,
    'enable_quotas': True
}
OPENSTACK_CINDER_FEATURES = {
    'enable_backup': False,
}
OPENSTACK_NEUTRON_NETWORK = {
    'enable_router': True,
    'enable_quotas': True,
    'enable_ipv6': True,
    'enable_distributed_router': False,
    'enable_ha_router': False,
    'enable_lb': True,
    'enable_firewall': True,
    'enable_vpn': True,
    'enable_fip_topology_check': True,
    # Default dns servers you would like to use when a subnet is
    # created.  This is only a default, users can still choose a different
    # list of dns servers when creating a new subnet.
    # The entries below are examples only, and are not appropriate for
    # real deployments
    # 'default_dns_nameservers': ["8.8.8.8", "8.8.4.4", "208.67.222.222"],
    # The profile_support option is used to detect if an external router can be
    # configured via the dashboard. When using specific plugins the
    # profile_support can be turned on if needed.
    'profile_support': None,
    #'profile_support': 'cisco',
    # Set which provider network types are supported. Only the network types
    # in this list will be available to choose from when creating a network.
    # Network types include local, flat, vlan, gre, vxlan and geneve.
    # 'supported_provider_types': ['*'],
    # You can configure available segmentation ID range per network type
    # in your deployment.
    # 'segmentation_id_range': {
    #     'vlan': [1024, 2048],
    #     'vxlan': [4094, 65536],
    # },
    # You can define additional provider network types here.
    # 'extra_provider_types': {
    #     'awesome_type': {
    #         'display_name': 'Awesome New Type',
    #         'require_physical_network': False,
    #         'require_segmentation_id': True,
    #     }
    # },
    # Set which VNIC types are supported for port binding. Only the VNIC
    # types in this list will be available to choose from when creating a
    # port.
    # VNIC types include 'normal', 'macvtap' and 'direct'.
    # Set to empty list or None to disable VNIC type selection.
    'supported_vnic_types': ['*'],
}
OPENSTACK_HEAT_STACK = {
    'enable_user_pass': True,
}
IMAGE_CUSTOM_PROPERTY_TITLES = {
    "architecture": _("Architecture"),
    "kernel_id": _("Kernel ID"),
    "ramdisk_id": _("Ramdisk ID"),
    "image_state": _("Euca2ools state"),
    "project_id": _("Project ID"),
    "image_type": _("Image Type"),
}
IMAGE_RESERVED_CUSTOM_PROPERTIES = []
API_RESULT_LIMIT = 1000
API_RESULT_PAGE_SIZE = 20
SWIFT_FILE_TRANSFER_CHUNK_SIZE = 512 * 1024
INSTANCE_LOG_LENGTH = 35
DROPDOWN_MAX_ITEMS = 30
LOGGING = {
    'version': 1,
    # When set to True this will disable all logging except
    # for loggers specified in this configuration dictionary. Note that
    # if nothing is specified here and disable_existing_loggers is True,
    # django.db.backends will still log unless it is disabled explicitly.
    'disable_existing_loggers': False,
    'formatters': {
        'operation': {
            # The format of "%(message)s" is defined by
            # OPERATION_LOG_OPTIONS['format']
            'format': '%(asctime)s %(message)s'
        },
    },
    'handlers': {
        'null': {
            'level': 'DEBUG',
            'class': 'logging.NullHandler',
        },
        'console': {
            # Set the level to "DEBUG" for verbose output logging.
            'level': 'INFO',
            'class': 'logging.StreamHandler',
        },
        'operation': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'operation',
        },
    },
    'loggers': {
        # Logging from django.db.backends is VERY verbose, send to null
        # by default.
        'django.db.backends': {
            'handlers': ['null'],
            'propagate': False,
        },
        'requests': {
            'handlers': ['null'],
            'propagate': False,
        },
        'horizon': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'horizon.operation_log': {
            'handlers': ['operation'],
            'level': 'INFO',
            'propagate': False,
        },
        'openstack_dashboard': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'novaclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'cinderclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'keystoneclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'glanceclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'neutronclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'heatclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'swiftclient': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'openstack_auth': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'nose.plugins.manager': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'django': {
            'handlers': ['console'],
            'level': 'DEBUG',
            'propagate': False,
        },
        'iso8601': {
            'handlers': ['null'],
            'propagate': False,
        },
        'scss': {
            'handlers': ['null'],
            'propagate': False,
        },
    },
}
SECURITY_GROUP_RULES = {
    'all_tcp': {
        'name': _('All TCP'),
        'ip_protocol': 'tcp',
        'from_port': '1',
        'to_port': '65535',
    },
    'all_udp': {
        'name': _('All UDP'),
        'ip_protocol': 'udp',
        'from_port': '1',
        'to_port': '65535',
    },
    'all_icmp': {
        'name': _('All ICMP'),
        'ip_protocol': 'icmp',
        'from_port': '-1',
        'to_port': '-1',
    },
    'ssh': {
        'name': 'SSH',
        'ip_protocol': 'tcp',
        'from_port': '22',
        'to_port': '22',
    },
    'smtp': {
        'name': 'SMTP',
        'ip_protocol': 'tcp',
        'from_port': '25',
        'to_port': '25',
    },
    'dns': {
        'name': 'DNS',
        'ip_protocol': 'tcp',
        'from_port': '53',
        'to_port': '53',
    },
    'http': {
        'name': 'HTTP',
        'ip_protocol': 'tcp',
        'from_port': '80',
        'to_port': '80',
    },
    'pop3': {
        'name': 'POP3',
        'ip_protocol': 'tcp',
        'from_port': '110',
        'to_port': '110',
    },
    'imap': {
        'name': 'IMAP',
        'ip_protocol': 'tcp',
        'from_port': '143',
        'to_port': '143',
    },
    'ldap': {
        'name': 'LDAP',
        'ip_protocol': 'tcp',
        'from_port': '389',
        'to_port': '389',
    },
    'https': {
        'name': 'HTTPS',
        'ip_protocol': 'tcp',
        'from_port': '443',
        'to_port': '443',
    },
    'smtps': {
        'name': 'SMTPS',
        'ip_protocol': 'tcp',
        'from_port': '465',
        'to_port': '465',
    },
    'imaps': {
        'name': 'IMAPS',
        'ip_protocol': 'tcp',
        'from_port': '993',
        'to_port': '993',
    },
    'pop3s': {
        'name': 'POP3S',
        'ip_protocol': 'tcp',
        'from_port': '995',
        'to_port': '995',
    },
    'ms_sql': {
        'name': 'MS SQL',
        'ip_protocol': 'tcp',
        'from_port': '1433',
        'to_port': '1433',
    },
    'mysql': {
        'name': 'MYSQL',
        'ip_protocol': 'tcp',
        'from_port': '3306',
        'to_port': '3306',
    },
    'rdp': {
        'name': 'RDP',
        'ip_protocol': 'tcp',
        'from_port': '3389',
        'to_port': '3389',
    },
}
REST_API_REQUIRED_SETTINGS = ['OPENSTACK_HYPERVISOR_FEATURES',
                              'LAUNCH_INSTANCE_DEFAULTS',
                              'OPENSTACK_IMAGE_FORMATS',
                              'OPENSTACK_KEYSTONE_DEFAULT_DOMAIN']
ALLOWED_PRIVATE_SUBNET_CIDR = {'ipv4': [], 'ipv6': []}

```

- To start the Horizon development server use the command below:

```bash

tox -e runserver 0.0.0.0:8000

```

#### Cấu hình Block Storage service - Cinder bkcloud12

- Cài đặt các package: ```# apt install cinder-volume```
- Kiểm tra thiết lập phân vùng vật lý và thiết lập lvm của máy chủ vật lý:

```conf

root@bkcloud12:~# fdisk -l
Disk /dev/sdb: 465.3 GiB, 499558383616 bytes, 975699968 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 4D43940E-A2DE-4FD0-A8D0-33C4CD8BEA5B

Device     Start       End   Sectors   Size Type
/dev/sdb1   2048 975699934 975697887 465.3G Linux LVM

Disk /dev/sda: 930.5 GiB, 999116767232 bytes, 1951399936 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 580442CB-8DF8-4F16-97F8-D867DD121533

Device         Start        End    Sectors   Size Type
/dev/sda1       2048    1050623    1048576   512M EFI System
/dev/sda2    1050624    2050047     999424   488M Linux filesystem
/dev/sda3    2050048  295018495  292968448 139.7G Linux LVM
/dev/sda4  295018496 1951399902 1656381407 789.8G Linux LVM
root@bkcloud12:~# 
root@bkcloud12:~# pvs
  PV         VG                       Fmt  Attr PSize   PFree  
  /dev/sda3  ubuntu-vg                lvm2 a--  139.70g   9.31g
  /dev/sda4  cinder-volumes-bkcloud12 lvm2 a--  789.82g 709.82g
root@bkcloud12:~# 
root@bkcloud12:~# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               139.70 GiB / not usable 2.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              35762
  Free PE               2384
  Allocated PE          33378
  PV UUID               eWOiod-Ppf4-3xeo-y8zc-l29j-sjsP-u7c6ZP
   
  --- Physical volume ---
  PV Name               /dev/sda4
  VG Name               cinder-volumes-bkcloud12
  PV Size               789.82 GiB / not usable 3.98 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              202194
  Free PE               181714
  Allocated PE          20480
  PV UUID               atGJhF-WroY-z5GW-tpop-Bpe3-11Hz-CPuOLS
root@bkcloud12:~# 
root@bkcloud12:~# vgs
  VG                       #PV #LV #SN Attr   VSize   VFree  
  cinder-volumes-bkcloud12   1   1   0 wz--n- 789.82g 709.82g
  ubuntu-vg                  1   2   0 wz--n- 139.70g   9.31g
root@bkcloud12:~# 
root@bkcloud12:~# vgdisplay 
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               139.70 GiB
  PE Size               4.00 MiB
  Total PE              35762
  Alloc PE / Size       33378 / 130.38 GiB
  Free  PE / Size       2384 / 9.31 GiB
  VG UUID               vmYL61-VKVR-NtBr-JVKb-00fw-kGec-xGFRkh
   
  --- Volume group ---
  VG Name               cinder-volumes-bkcloud12
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  14
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               789.82 GiB
  PE Size               4.00 MiB
  Total PE              202194
  Alloc PE / Size       20480 / 80.00 GiB
  Free  PE / Size       181714 / 709.82 GiB
  VG UUID               oWPOpa-EiC4-SQCr-6INz-nFqb-5T7A-2gfSmF
root@bkcloud12:~# 
root@bkcloud12:~# lvs
  LV                                          VG                       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert

  root                                        ubuntu-vg                -wi-ao---- 111.76g                                                    
  swap                                        ubuntu-vg                -wi-ao----  18.62g               
root@bkcloud12:~#                                    
root@bkcloud12:~# lvdisplay 
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/root
  LV Name                root
  VG Name                ubuntu-vg
  LV UUID                3XgPKY-sXbv-ysbG-Nkev-lWlo-LCHD-0aNivJ
  LV Write Access        read/write
  LV Creation host, time bkcloud12, 2017-08-26 18:30:48 +0700
  LV Status              available
  # open                 1
  LV Size                111.76 GiB
  Current LE             28610
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0
   
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/swap
  LV Name                swap
  VG Name                ubuntu-vg
  LV UUID                7KMW8s-12yT-3vWO-CBn3-MmXL-0euZ-0ctcY2
  LV Write Access        read/write
  LV Creation host, time bkcloud12, 2017-08-26 18:31:07 +0700
  LV Status              available
  # open                 2
  LV Size                18.62 GiB
  Current LE             4768
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1

```

- Thiết lập lvm filter để ngăn LVM của máy chủ vật lý scan và add cache các volume của các máy ảo. LVM filter này sẽ chỉ cho phép LVM của máy chủ vật lý scan các phân vùng vật lý cần thiết:
- File ```/etc/lvm/lvm.conf```

```conf

devices {
...
        filter = [  "a/sda1/", "a/sda2/", "a/sda3/","a/sda4/", "r/.*/"]
...
}

```

- File ```/etc/cinder/cinder.conf```

```conf

root@bkcloud12:~# cat /etc/cinder/cinder.conf | egrep -v "(^#.*|^$)"
[DEFAULT]
rootwrap_config = /etc/cinder/rootwrap.conf
api_paste_confg = /etc/cinder/api-paste.ini
iscsi_helper = tgtadm
volume_name_template = volume-%s
verbose = True
auth_strategy = keystone
state_path = /var/lib/cinder
lock_path = /var/lock/cinder
volumes_dir = /var/lib/cinder/volumes
transport_url = rabbit://openstack:xxxx@controller
auth_strategy = keystone
my_ip = 192.168.50.12
enabled_backends = lvm-bkcloud12
glance_api_servers = http://controller:9292
[database]
connection = mysql+pymysql://cinder:xxxx@controller/cinder
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = xxxx
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
[lvm-bkcloud12]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes-bkcloud12
iscsi_protocol = iscsi
iscsi_helper = tgtadm
iscsi_ip_address = 192.168.80.12


```

- Restart the Compute API service and Block Storage Services:

```bash
service tgt restart
service cinder-volume restart
```