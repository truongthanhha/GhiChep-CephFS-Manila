# Cài đặt và cấu hình trên node Controller

## Điều kiện kiên quyết
### 1. Thực hiện khởi tạo database, làm các bước sau

<span style=“color:red;”> text </span>

- Kết nối database với user root
```
$ mysql -u root -p
```

- Khởi tạo database *manila*:
```
CREATE DATABASE manila;
```
- Gán quyền truy cập manila database:
```
GRANT ALL PRIVILEGES ON manila.* TO 'manila'@'localhost' \
  IDENTIFIED BY 'MANILA_DBPASS';
GRANT ALL PRIVILEGES ON manila.* TO 'manila'@'%' \
  IDENTIFIED BY 'MANILA_DBPASS';
```
Thay **MANILA_DBPASS** bằng mật khẩu phù hợp
- Thoát khỏi database client

### 2. Export các thông tin xác thực 
```
$ source admin-openrc.sh
```

### 3. Khởi tạo service
- Khởi tạo **manila**  user:
```
$ openstack user create --domain default --password-prompt manila
User Password:
Repeat User Password:
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | e0353a670a9e496da891347c589539e9 |
| enabled   | True                             |
| id        | 83a3990fc2144100ba0e2e23886d8acc |
| name      | manila                           |
+-----------+----------------------------------+
```
- Thêm **admin** role cho **manila** user
```
$ openstack role add --project service --user manila admin
```
>**Chú ý**: Lệnh trên không có output
	
- Khởi tạo **manila** và **manilav2** service 
```
$ openstack service create --name manila \
  --description "OpenStack Shared File Systems" share
  +-------------+----------------------------------+
  | Field       | Value                            |
  +-------------+----------------------------------+
  | description | OpenStack Shared File Systems    |
  | enabled     | True                             |
  | id          | 82378b5a16b340aa9cc790cdd46a03ba |
  | name        | manila                           |
  | type        | share                            |
  +-------------+----------------------------------+
```

```
$ openstack service create --name manilav2 \
  --description "OpenStack Shared File Systems" sharev2
  +-------------+----------------------------------+
  | Field       | Value                            |
  +-------------+----------------------------------+
  | description | OpenStack Shared File Systems    |
  | enabled     | True                             |
  | id          | 30d92a97a81a4e5d8fd97a32bafd7b88 |
  | name        | manilav2                         |
  | type        | sharev2                          |
  +-------------+----------------------------------+
  
```

### 4. Khởi tạo Shared File Systems API endpoints
```
$ openstack endpoint create --region RegionOne \
  share public http://controller:8786/v1/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | 0bd2bbf8d28b433aaea56a254c69f69d        |
  | interface    | public                                  |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | 82378b5a16b340aa9cc790cdd46a03ba        |
  | service_name | manila                                  |
  | service_type | share                                   |
  | url          | http://controller:8786/v1/%(tenant_id)s |
  +--------------+-----------------------------------------+

$ openstack endpoint create --region RegionOne \
  share internal http://controller:8786/v1/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | a2859b5732cc48b5b083dd36dafb6fd9        |
  | interface    | internal                                |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | 82378b5a16b340aa9cc790cdd46a03ba        |
  | service_name | manila                                  |
  | service_type | share                                   |
  | url          | http://controller:8786/v1/%(tenant_id)s |
  +--------------+-----------------------------------------+

$ openstack endpoint create --region RegionOne \
  share admin http://controller:8786/v1/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | f7f46df93a374cc49c0121bef41da03c        |
  | interface    | admin                                   |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | 82378b5a16b340aa9cc790cdd46a03ba        |
  | service_name | manila                                  |
  | service_type | share                                   |
  | url          | http://controller:8786/v1/%(tenant_id)s |
  +--------------+-----------------------------------------+
  
  ```
  
  ```
  $ openstack endpoint create --region RegionOne \
  sharev2 public http://controller:8786/v2/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | d63cc0d358da4ea680178657291eddc1        |
  | interface    | public                                  |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | 30d92a97a81a4e5d8fd97a32bafd7b88        |
  | service_name | manilav2                                |
  | service_type | sharev2                                 |
  | url          | http://controller:8786/v2/%(tenant_id)s |
  +--------------+-----------------------------------------+

$ openstack endpoint create --region RegionOne \
  sharev2 internal http://controller:8786/v2/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | afc86e5f50804008add349dba605da54        |
  | interface    | internal                                |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | 30d92a97a81a4e5d8fd97a32bafd7b88        |
  | service_name | manilav2                                |
  | service_type | sharev2                                 |
  | url          | http://controller:8786/v2/%(tenant_id)s |
  +--------------+-----------------------------------------+

$ openstack endpoint create --region RegionOne \
  sharev2 admin http://controller:8786/v2/%\(tenant_id\)s
  +--------------+-----------------------------------------+
  | Field        | Value                                   |
  +--------------+-----------------------------------------+
  | enabled      | True                                    |
  | id           | e814a0cec40546e98cf0c25a82498483        |
  | interface    | admin                                   |
  | region       | RegionOne                               |
  | region_id    | RegionOne                               |
  | service_id   | 30d92a97a81a4e5d8fd97a32bafd7b88        |
  | service_name | manilav2                                |
  | service_type | sharev2                                 |
  | url          | http://controller:8786/v2/%(tenant_id)s |
  +--------------+-----------------------------------------+
  ```
  
## Cài đặt và cấu hình các thành phần
### 1.  Cài đặt các packages:
```
apt-get install manila-api manila-scheduler python-manilaclient
```

### 2. Chỉnh sửa file /etc/manila/manila.conf thêm các thông tin sau

  - Cấu hình truy cập database trong phần **[database]** :
  
  ```
  [database]
  connection = mysql+pymysql://manila:MANILA_DBPASS@controller/manila
  ```
 Thay **MANILA_DBPASS** bằng mật khẩu của bạn
 
### 3. Hoàn tất cấu hình file **manila.conf**

  - Trong thẻ [DEFAULT], cấu hình truy cập vào RabbitMQ message queue
 
   ```
   [DEFAULT]
   transport_url = rabbit://openstack:RABBIT_PASS@controller
   ```
   Thay **RABBIT_PASS** bằng mật khẩu của bạn
   
   - Trong thẻ [DEFAULT], thêm các tham số cấu hình 
   
   ```
   [DEFAULT]
   default_share_type = default_share_type
   share_name_template = share-%s
   rootwrap_config = /etc/manila/rootwrap.conf
   api_paste_config = /etc/manila/api-paste.ini
   ```
   - Trong thẻ [DEFAULT] và [keystone_authtoken],[oslo_concurrency] thêm nội dung:
   ```
   [DEFAULT]
    auth_strategy = keystone
	my_ip = 172.16.68.90

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

   [oslo_concurrency]
   lock_path = /var/lock/manila

### 4. Thực hiện đồng bộ database 
```
$ su -s /bin/sh -c "manila-manage db sync" manila
```
## Hoàn tất việc cài đặt
### 1. Khởi động lại dịch vụ

```
$ service manila-scheduler restart
$ service manila-api restart
```
