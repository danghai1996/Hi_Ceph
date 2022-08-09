# Cấu hình Ceph Object Gateway làm Primary Backend lưu trữ dữ liệu cho NextCloud

## Trên node Ceph Mon
Tạo user S3 cho NextCloud:
```
radosgw-admin user create --uid="nextcloud" --display-name="Nextcloud S3 User"
radosgw-admin quota set --quota-scope=user --uid="nextcloud" --max-objects=-1 --max-size=20G
```

Output:
```json
{
    "user_id": "nextcloud",
    "display_name": "Nextcloud S3 User",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "subusers": [],
    "keys": [
        {
            "user": "nextcloud",
            "access_key": "VEFXEAA3B9OY131CKNKJ",
            "secret_key": "T5LTlWtNqgGD4PF6UGCevm00L1oytuP1pSsray21"
        }
    ],
    "swift_keys": [],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "default_storage_class": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "temp_url_keys": [],
    "type": "rgw",
    "mfa_ids": []
}
```

Khởi tạo 1 bucket cho NextCloud. Ở đây, ta đã tạo sẵn 1 bucket có tên `nextcloud`:
```
root@cephadm01:/# radosgw-admin bucket list
[
    "nextcloud",
    "bucket-haidd",
    "bucket-test",
    "bk-smartcloud"
]
```

# Trên node NextCloud
Chỉnh sửa file `/var/www/nextcloud/config/config.php`. Thực hiện thêm phần cấu hình như dưới đây:

```php
  'objectstore' =>
  array (
    'class' => 'OC\\Files\\ObjectStore\\S3',
    'arguments' =>
    array (
      'bucket' => 'nextcloud',
      'autocreate' => true,
      'key' => 'VEFXEAA3B9OY131CKNKJ',
      'secret' => 'T5LTlWtNqgGD4PF6UGCevm00L1oytuP1pSsray21',
      'hostname' => '192.168.60.51',
      'port' => 80,
      'use_ssl' => false,
      'region' => 'optional',
      'use_path_style' => true,
    ),
  ),
```

- `'bucket' => 'nextcloud',` : Tên bucket lưu trữ dữ liệu
- `'key' => 'VEFXEAA3B9OY131CKNKJ',` : access key user tạo ở trên
- `'secret' => 'T5LTlWtNqgGD4PF6UGCevm00L1oytuP1pSsray21',` : Secreate key tạo ở trên
- `'hostname' => '192.168.60.51',` : đường dẫn tới ObjectGW
- `'port' => 80,` : Port của service Object GW

File sau khi cấu hình sẽ tương tự như sau:
```php
<?php
$CONFIG = array (
  'instanceid' => 'oc84yytgr9vg',
  'passwordsalt' => 'xkU4ZfS/+BXix585Ngs036dwCMGyIO',
  'secret' => 'XbZpUrHcZ/wWdGgMCPAR91GljbfpRCQDT3J5sdEQQdMzt5iI',
  'trusted_domains' =>
  array (
    0 => '192.168.10.55',
  ),
  'objectstore' =>
  array (
  'class' => 'OC\\Files\\ObjectStore\\S3',
  'arguments' =>
  array (
      'bucket' => 'nextcloud',
      'autocreate' => true,
      'key' => 'VEFXEAA3B9OY131CKNKJ',
      'secret' => 'T5LTlWtNqgGD4PF6UGCevm00L1oytuP1pSsray21',
      'hostname' => '192.168.60.51',
      'port' => 80,
      'use_ssl' => false,
      'region' => 'optional',
      'use_path_style' => true,
  ),
  ),
  'datadirectory' => '/var/www/nextcloud/data',
  'dbtype' => 'mysql',
  'version' => '24.0.3.2',
  'overwrite.cli.url' => 'http://192.168.10.55',
  'dbname' => 'nextcloud',
  'dbhost' => 'localhost',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'nc_user',
  'dbpassword' => 'Vnpt@!2022',
  'installed' => true,
);
```

Restart service apache2:
```
systemctl restart apache2
```

# Tham khảo:
- https://docs.nextcloud.com/server/latest/admin_manual/configuration_files/primary_storage.html#simple-storage-service-s3
- https://github.com/lacoski/khoa-luan/blob/master/Lab/ceph-nextcloud-s3.md