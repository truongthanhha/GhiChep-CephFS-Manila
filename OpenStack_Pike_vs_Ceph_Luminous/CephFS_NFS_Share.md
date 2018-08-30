# Cấu hình CephFS native share backend trong file manila.conf

## 1. Thêm tham số **enabled_share_protocols ** trong file /etc/manila/manila.conf ở server cài manila-api service
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





