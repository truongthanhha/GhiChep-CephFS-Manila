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