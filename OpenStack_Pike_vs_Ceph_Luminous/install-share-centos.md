# Cài đặt và cấu hình các thành phần
## 1. Cài đặt các package
```
$ yum install openstack-manila-share python2-PyMySQL
```

## 2. Chỉnh sửa file **/etc/manila/manila.conf** :

- Trong thẻ [database], cấu hình truy cập database

```
[database]
connection = mysql://manila:MANILA_DBPASS@controller/manila
```
Thay **MANILA_DBPASS** bằng mật khẩu mà bạn đã chọn khi phần quyền truy cập database **manila**

## 4. Hoàn thành việc cấu hình file **manila.conf**

- Trong thẻ **[DEFAULT]** thêm các trường thông tin sau:

```
[DEFAULT]
transport_url = rabbit://openstack:RABBIT_PASS@controller
default_share_type = default_share_type
rootwrap_config = /etc/manila/rootwrap.conf
my_ip = 172.16.68.94
```
172.16.68.94 là địa chỉ IP của Ceph1 node

- Trong thẻ **[DEFAULT]** và **[keystone_authtoken]** cấu hình các thông tin xác thực. 
```
[DEFAULT]
auth_strategy = keystone

[keystone_authtoken]
memcached_servers = controller:11211
auth_uri = http://controller:5000
auth_url = http://controller:35357
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = manila
password = MANILA_PASS
```
- Trong thẻ **[oslo_concurrency]** thêm lock_path
```
[oslo_concurrency]
lock_path = /var/lib/manila/tmp
```
Thay **MANILA_DBPASS** bằng mật khẩu bạn chọn khi tạo user manila


# CephFS Driver
Có 2 tùy chọn sử dụng CephFS Native shares và sử dụng CephFS NFS shares

- Nếu máy ảo sử dụng CephFS sử dụng giao thức của Ceph, truy cập được quản lý bởi cơ chế xác thực cephx của Ceph. 
Nếu User yêu cầu truy cập, Ceph sẽ khởi tạo Ceph auth ID và secret key tương ứng
. Để biết thêm về phương thức Client truy cập share khởi tạo bởi driver tham khảo thêm
trong tại liệu của Ceph http://docs.ceph.com/docs/master/cephfs/.

>**Chú ý** phía máy ảo nếu sử dụng mount của kernel thay vì sử dụng FUSE, thiết lập size limits có thể
không được áp dụng

- Nếu máy ảo truy cập CephFS thông qua NFS, NFS-Ganesha server sẽ làm trung gian truy cập
vào CephFS. 

## Các tính năng

- Khởi tạo/xóa share
- Đối với CephFS native
  - CephFS native chỉ hỗ trợ truy cập thông qua cơ chế xác thực của **cephx**
  - **read-only** chỉ hỗ trợ bản từ bản manila Newton trở đi
  - **read-write** hỗ trợ từ bản Mitaka trở đi
  
- Đối với truy cập share thông qua NFS
  - Truy cập thông qua giao thức NFS chỉ hỗ trợ bởi qua **ip**
  - **read-only** và **read-write** chỉ hỗ trợ từ bản Pike trở đi
 
- Mở rộng hoặc thu nhỏ kích thước của share
- Khởi tạo/xóa snapshot

>**Warning** Tính năng CephFS snapshot chỉ đang thử nghiệm chưa hỗ trợ trong product

## Các điều kiện kiên quyết

### 1. Đối với CephFS native share
- Mitaka hoặc bản mới nhất của manila
- Jewel hoặc bản mới nhất của Ceph
- Ceph cluster đã được cấu hình và thiết lập file system
- Gói **ceph-common** được cài đặt trên server chạy manila-share service 
- Đối với Ceph client trên máy ảo,sử dụng **ceph-fuse** tốt hơn việc sử dụng mount native của Kernel 
- Kết nối giữa Ceph cluster public network với server chạy **manila-share** service
- Kết nối giữa Ceph cluster public network với máy ảo

### Đối với CephFS NFS shares
- Pike hoặc bản mới nhất của manila
- Kraken hoặc bản mới nhất của Ceph
- 2.5 hoặc bản mới nhất của NFS-Ganesha
- Ceph cluster đã được cấu hình và thiết lập file system
- Gói **ceph-common** được cài đặt trên server chạy manila-share service
- NFS client được cài đặt trên VM
- Kết nối giữa Ceph cluster public network với server chạy **manila-share** service
- Kết nối giữa Ceph cluster public network với NFS-Ganesha server 
- Kết nối giữa NFS-Ganesha server với máy ảo

## Thêm user xác thực khi driver giao tiếp với hệ thống Ceph

Thực hiện trên client 

```
read -d '' MON_CAPS << EOF
allow r,
allow command "auth del",
allow command "auth caps",
allow command "auth get",
allow command "auth get-or-create"
EOF

ceph auth get-or-create client.manila -o manila.keyring \
mds 'allow *' \
osd 'allow rw' \
mon "$MON_CAPS"
```

manila.keyring với ceph.conf cần được đặt tại server chạy manila-share service 
Phân quyền sở hữu manila.keyring cho tiến trình manila-share
```
chown manila:manila /etc/ceph/manila.keyring
```

Thêm thẻ ***[client.manila]*** vào ceph.conf
```
[client.manila]
client mount uid = 0
client mount gid = 0
log file = /var/log/ceph/ceph-client.manila.log
admin socket = /opt/stack/status/stack/ceph-$name.$pid.asok
keyring = /etc/ceph/manila.keyring
```

## Thiết lập CephFS

Có 2 tùy chọn

### 1. sử dụng CephFS native
### 2. Sử dụng CephFS NFS share
