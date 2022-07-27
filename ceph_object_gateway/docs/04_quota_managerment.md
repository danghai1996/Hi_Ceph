# Quota Managerment

Ceph Object Gateway cho phép bạn đặt quota cho user và bucket. Quota bao gồm số lượng object trong bucket và giới hạn dung lượng của bucket có thể lưu trữ.

- **Bucket**: option `--bucket` cho phép bạn chỉ định quota cho nhóm người dùng sở hữu bucket.
- **Maximum Objects**: `--max-objects` cho phép bạn chỉ định số lượng object tối đa. Set giá trị âm sẽ disable giá trị này.
- **Maximum Size**: The `--max-size` cho phép bạn chỉ định quota kcihs thước trong B/K/M/G/T. Trong đó, B là giá trị mặc định. Giá trị âm sẽ vô hiệu hóa giá trị này.
- **Quota Scope**: The `--quota-scope` đặt scope cho quota. Nó sẽ quyết định quota được set cho `bucket` hay `user`. Quota cho bucket áp dụng cho bucket mà user sở hữu. User quota sẽ áp dụng cho user.

## 1. Set user quota
Trước khi enable quota, ta cần phải set các thông số cho quota (quota parameters)
```
radosgw-admin quota set --quota-scope=user --uid=<uid> [--max-objects=<num objects>] [--max-size=<max size>]
```

Ví dụ:
```
radosgw-admin quota set --quota-scope=user --uid=cloud --max-objects=1024 --max-size=1024B
```

## 2. Enable/Disable user quota
Enable:
```
radosgw-admin quota enable --quota-scope=user --uid=<uid>
```

Disable:
```
radosgw-admin quota disable --quota-scope=user --uid=<uid>
```

Ví dụ:
```
radosgw-admin quota enable --quota-scope=user --uid=cloud
```

## 3. Set bucket quota
```
radosgw-admin quota set --uid=<uid> --quota-scope=bucket [--max-objects=<num objects>] [--max-size=<max size]
```
- `uid`: id user sở hữu bucket.

## 4. Enable/Disable bucket quota
Enable:
```
radosgw-admin quota enable --quota-scope=bucket --uid=<uid>
```

Disable:
```
radosgw-admin quota disable --quota-scope=bucket --uid=<uid>
```

## 5. Get quota setting
Ta có thể xem thông tin thiết lập quota thông qua info của user:
```
radosgw-admin user info --uid=<uid>
```

Ví dụ:
```json
root@cephadm01:/# radosgw-admin user info --uid=cloud
{
    "user_id": "cloud",
    "display_name": "Cloud VNPT",
    "email": "cloud@vnpt.vn",
    "suspended": 0,
    "max_buckets": 1000,
    "subusers": [
        {
            "id": "cloud:haidd",
            "permissions": "full-control"
        }
    ],
    "keys": [
        {
            "user": "cloud",
            "access_key": "91F6X1UX4UHM1CFEM6RZ",
            "secret_key": "BR8Zc6MMMrQIiq23nqYZQCJoG1LZfkSyzwknBILr"
        }
    ],
    "swift_keys": [
        {
            "user": "cloud:haidd",
            "secret_key": "UmdgbEl166Acr4TJwTnrAcUdiOx1FGCT5UcahGn0"
        }
    ],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "default_storage_class": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": true,
        "check_on_raw": false,
        "max_size": 10240,
        "max_size_kb": 10,
        "max_objects": 1024
    },
    "user_quota": {
        "enabled": true,
        "check_on_raw": false,
        "max_size": 1024,
        "max_size_kb": 1,
        "max_objects": 1024
    },
    "temp_url_keys": [],
    "type": "rgw",
    "mfa_ids": []
}
```

## 6. Update quota stats
Quota stats không đồng bộ, ta có thể thực hiện update quota stats cho user và bucket thủ công bằng lệnh dưới đây:
```
radosgw-admin user stats --uid=<uid> --sync-stats
```

## 7. Get user usage stats
Xem mức độ sử dụng quota của user
```
radosgw-admin user stats --uid=<uid>
```
Để lấy dữ liệu mới nhất, thêm option `--sync-stats`

Ví dụ:
```json
radosgw-admin user stats --uid=cloud

{
    "stats": {
        "size": 0,
        "size_actual": 0,
        "size_kb": 0,
        "size_kb_actual": 0,
        "num_objects": 0
    },
    "last_stats_sync": "2022-07-02T08:06:45.359380Z",
    "last_stats_update": "2022-07-05T08:31:20.982058Z"
}
```

## 8. Default quotas
Default quota sẽ được sử dụng cho những user được tạo mới và không ảnh hưởng tới các user đã tồn tại.

Xem cấu hình `rgw bucket default quota max objects`, `rgw bucket default quota max size`, `rgw user default quota max objects`, và `rgw user default quota max size` trong phần [file cấu hình của ceph cho rados gateway](https://docs.ceph.com/en/quincy/radosgw/config-ref/).

## 9. Reading/Writing Global quotas
Ta có thể xem hoặc thêm cài đặt global quota (Default quota). Xem global quota setting:
```
radosgw-admin global quota get
```

Ví dụ:
```json
root@cephadm01:/# radosgw-admin global quota get
{
    "bucket quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "user quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    }
}
```

Thiết lập global quota setting được thực hiện bằng lệnh tương tự với các lệnh `quota set`, `quota enable` và `quota disable`

Ví dụ:
```json
root@cephadm01:/# radosgw-admin global quota set --quota-scope bucket --max-objects 2048

Global quota changes saved. They will take effect as the gateways are restarted.
{
    "bucket quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": 2048
    }
}

root@cephadm01:/# radosgw-admin global quota enable --quota-scope bucket
Global quota changes saved. They will take effect as the gateways are restarted.
{
    "bucket quota": {
        "enabled": true,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": 2048
    }
}
```


# Tham khảo
- https://docs.ceph.com/en/quincy/radosgw/admin/?#quota-management