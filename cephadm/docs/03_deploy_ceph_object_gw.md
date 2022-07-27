# Deploy Ceph Object Gateway với `cephadm`

## 1. Khởi tạo RGW
Khởi tạo 2 daemon RGW với id là `s3cloud`
```
ceph orch apply rgw s3cloud
```

Kiểm tra:
```
ceph orch ls
```
KQ:
```
NAME                       PORTS        RUNNING  REFRESHED  AGE  PLACEMENT
alertmanager               ?:9093,9094      1/1  8m ago     2d   count:1
crash                                       3/3  8m ago     2d   *
grafana                    ?:3000           1/1  8m ago     2d   count:1
mgr                                         2/2  8m ago     2d   count:2
mon                                         3/5  8m ago     2d   count:5
node-exporter              ?:9100           3/3  8m ago     2d   *
osd.all-available-devices                     6  8m ago     2d   *
prometheus                 ?:9095           1/1  8m ago     2d   count:1
rgw.s3cloud                ?:80             2/2  8m ago     30m  count:2
```

## 2. Tạo user
Tạo user:
```
radosgw-admin user create --uid=cloud --display-name="Cloud VNPT" --email=cloud@vnpt.vn
```

```json
{
    "user_id": "cloud",
    "display_name": "Cloud VNPT",
    "email": "cloud@vnpt.vn",
    "suspended": 0,
    "max_buckets": 1000,
    "subusers": [],
    "keys": [
        {
            "user": "cloud",
            "access_key": "91F6X1UX4UHM1CFEM6RZ",
            "secret_key": "BR8Zc6MMMrQIiq23nqYZQCJoG1LZfkSyzwknBILr"
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

Liệt kê các user:
```
radosgw-admin  user list
```
Output:
```json
[
    "cloud",
    "dashboard",
    "haidd"
]
```

Tạo subuser:
```
radosgw-admin subuser create --uid=cloud --subuser=cloud:haidd --access=full
```
```json
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
