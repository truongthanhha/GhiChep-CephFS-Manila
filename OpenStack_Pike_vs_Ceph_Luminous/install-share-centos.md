# Cài đặt và cấu hình các thành phần
## 1. Cài đặt các pack
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