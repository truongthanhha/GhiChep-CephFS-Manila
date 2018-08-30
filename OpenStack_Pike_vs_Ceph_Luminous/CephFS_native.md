# Cấu hình CephFS native share backend trong file manila.conf

## 1. Thêm tham số **enabled_share_protocols **
```
enabled_share_protocols = NFS,CIFS,CEPHFS
```
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

sử dụng tính năng snapshot, thiết lập giá trị **cephfs_enable_snapshots ** là **True**

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


