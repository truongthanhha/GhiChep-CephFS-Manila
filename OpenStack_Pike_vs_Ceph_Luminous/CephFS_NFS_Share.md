# Cấu hình CephFS NFS share backend trong file manila.conf

## 1. Enable protocol 
Thêm tham số **enabled_share_protocols** trong file /etc/manila/manila.conf ở server cài manila-api service
```
enabled_share_protocols = NFS,CIFS,CEPHFS
```
Sau đó restart lại manila-api
## 2. Enable backend 

```
nabled_share_backends = cephfsnfs1
```

## 3. Định nghĩa backend **cephfsnfs1** đã khai báo ở trên:
```
[cephfsnfs1]
driver_handles_share_servers = False
share_backend_name = CEPHFSNFS1
share_driver = manila.share.drivers.cephfs.driver.CephFSDriver
cephfs_protocol_helper_type = NFS
cephfs_conf_path = /etc/ceph/ceph.conf
cephfs_auth_id = manila1
cephfs_cluster_name = ceph
cephfs_enable_snapshots = False
cephfs_ganesha_server_is_remote= False
cephfs_ganesha_server_ip = 172.16.68.94
```
- Tham số **driver-handles-share-servers** được thiết lập giá trị **False** nếu không có **share-servers**

- thiết lập tham số **cephfs_protocol_helper_type** giá trị **NFS** cho phép truy cập CephFS thông qua giao thức NFS 
- Tham số **ceph_auth_id** là user xác thực với Ceph Cluster
- Tham số **cephfs_ganesha_server_is_remote** thiết lập là False nếu NFS-ganesha server được cài đặt cùng server với **manila-share**
service. Nếu Ganesha chạy trên server khác, thiết lập là True, cùng với các thiết lập **cephfs_ganesha_server_ip**, **cephfs_ganesha_server_username**,
**cephfs_ganesha_server_password **

# Tạo shares

- Khởi tạo share type sử dụng NFS protocol
```
manila type-create cephfsnfstype false
manila type-key cephfsnfstype set vendor_name=Ceph storage_protocol=NFS
```

- Khởi tạo share

```
manila create --share-type cephfsnfstype --name cephnfsshare3 nfs 3
```
- Xuất thông tin share
```
manila share-export-location-list cephnfsshare3
```
Thông tin sẽ nhận được **{NFS-Ganesha server address}:{path to be mounted}**

# Cho phép truy cập share 

- Cho phép Ceph ID alice có quyền truy cập share sử dụng NFS share

```
manila access-allow cephnfsshare3 ip 192.168.20.74
+--------------------------------------+---------------------------------------------------------------------+-----------+
| ID                                   | Path                                                                | Preferred |
+--------------------------------------+---------------------------------------------------------------------+-----------+
| d04086a9-7b30-4b5f-9b6d-8c755cdb3da4 | 172.16.68.94:/volumes/_nogroup/19e871c6-bbc2-45b4-8c7c-714279bf52b1 | False     |
+--------------------------------------+---------------------------------------------------------------------+-----------+

```
- Cài đặt gói nfs client trên máy ảo
```
yum install nfs-utils
```
- Thực hiện mount trên client
```
sudo mount -o rw,noatime -t nfs 172.16.68.94:/volumes/_nogroup/19e871c6-bbc2-45b4-8c7c-714279bf52b1 /opt/cephfsNFS
```

Kết quả:
```
[root@vm-centos7 /]# sudo mount -t nfs 172.16.68.94:/volumes/_nogroup/19e871c6-bbc2-45b4-8c7c-714279bf52b1 /mnt/
[root@vm-centos7 /]# df -h
Filesystem                                                           Size  Used Avail Use% Mounted on
/dev/vda1                                                             16G  1.7G   14G  12% /
devtmpfs                                                             485M     0  485M   0% /dev
tmpfs                                                                496M     0  496M   0% /dev/shm
tmpfs                                                                496M  6.6M  490M   2% /run
tmpfs                                                                496M     0  496M   0% /sys/fs/cgroup
tmpfs                                                                100M     0  100M   0% /run/user/0
172.16.68.94:/volumes/_nogroup/19e871c6-bbc2-45b4-8c7c-714279bf52b1  3.0G     0  3.0G   0% /mnt
[root@vm-centos7 /]# cd /mnt/
[root@vm-centos7 mnt]# touch thanhha
[root@vm-centos7 mnt]# echo "thanhha" > thanhha
[root@vm-centos7 mnt]# cat thanhha
thanhha
[root@vm-centos7 mnt]#
```