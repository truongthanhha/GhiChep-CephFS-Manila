# Cấu hình CephFS native share backend trong file manila.conf

## 1. Enable protocol 
Thêm tham số **enabled_share_protocols ** trong file /etc/manila/manila.conf ở server cài manila-api service
```
enabled_share_protocols = NFS,CIFS,CEPHFS
```
Sau đó restart lại manila-api

## 2. Enable backend 

```
nabled_share_backends = cephfsnative1
```

## 3. Định nghĩa backend **cephfsnative1** đã khai báo ở trên:
```
[cephfsnative1]
driver_handles_share_servers = False
share_backend_name = CEPHFSNATIVE1
share_driver = manila.share.drivers.cephfs.driver.CephFSDriver
cephfs_conf_path = /etc/ceph/ceph.conf
cephfs_protocol_helper_type = CEPHFS
cephfs_auth_id = manila
cephfs_cluster_name = ceph
cephfs_enable_snapshots = false
```
Tham số **driver-handles-share-servers** được thiết lập giá trị **False** nếu không có **share-servers**

sử dụng tính năng snapshot, thiết lập giá trị **cephfs_enable_snapshots** là **True**

Sử dụng native Ceph protocol, thiết lập tham số **cephfs_protocol_helper_type** giá trị **CEPHFS**

# Tạo shares
- Giá trị share type có thể có thiết lập driver_handles_share_servers là True. Cấu hình share type phù hợp với CephFS native share
```
manila type-create cephfsnativetype false
manila type-key cephfsnativetype set vendor_name=Ceph storage_protocol=CEPHFS
```
- Khởi tạo share:
```
manila create --share-type cephfsnativetype --name cephnativeshare1 cephfs 1
```
- Lấy thông tin share:
```
manila share-export-location-list cephnativeshare1
```
Thông tin sẽ nhận được **{mon ip addr:port}[,{mon ip addr:port}]:{path to be mounted}**


# Cho phép truy cập share 

- Cho phép Ceph ID alice có quyền truy cập share sử dụng cephx

```
manila access-allow cephnativeshare1 cephx alice
```

- Lấy về assess/secret key của alice

```
manila access-list cephnativeshare1
+--------------------------------------+-------------+-----------+--------------+--------+------------------------------------------+----------------------------+----------------------------+
| id                                   | access_type | access_to | access_level | state  | access_key                               | created_at                 | updated_at                 |
+--------------------------------------+-------------+-----------+--------------+--------+------------------------------------------+----------------------------+----------------------------+
| 010dc080-ce5b-4277-bbbb-36317ff32fa9 | cephx       | alice     | rw           | active | AQDShntbXw5uCxAAqdhLz1kjAltSFSDxOlFEoQ== | 2018-08-30T06:57:54.000000 | 2018-08-30T06:57:55.000000 |
+--------------------------------------+-------------+-----------+--------------+--------+------------------------------------------+----------------------------+----------------------------+
```
# Mount vào máy ảo

## Khởi tạo keyring alice.keyring với ID alice xác thực với hệ thống Ceph Cluster

```
[client.alice]
        key = AQA8+ANW/4ZWNRAAOtWJMFPEihBA1unFImJczA==
```
## Khởi tạo ceph.conf chứa thông tin IP của MON 

```
[client]
        client quota = true
        mon host = 172.16.68.94
```

## Sử dụng ceph-fuse để mount

- Cài đặt ceph-fuse trên CentOS7
```
 sudo yum -y install epel-release
 rpm -Uhv http://download.ceph.com/rpm-luminous/el7/noarch/ceph-release-1-1.el7.noarch.rpm
 yum install ceph-fuse
```
Thực hiện mount

```
sudo ceph-fuse /mnt \
--id=alice \
--conf=/etc/ceph/ceph.conf \
--keyring=/etc/ceph/alice.keyring \
--client-mountpoint=/volumes/_nogroup/84f67aff-a1ef-49f6-80fe-346a1f73056c
```
Kết quả

```
[root@vm-centos7 ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        16G  1.8G   14G  12% /
devtmpfs        492M     0  492M   0% /dev
tmpfs           497M     0  497M   0% /dev/shm
tmpfs           497M   19M  479M   4% /run
tmpfs           497M     0  497M   0% /sys/fs/cgroup
tmpfs           100M     0  100M   0% /run/user/0
ceph-fuse       1.0G     0  1.0G   0% /mnt
[root@vm-centos7 ~]# echo thanhha >> /mnt/thanhha
[root@vm-centos7 ~]# cat /mnt/thanhha
thanhha
[root@vm-centos7 ~]#
```