# Bucket Managerment

## Các lệnh thao tác với Bucket
### 1. List tất cả bucket
```
radosgw-admin bucket list
```

Ví dụ:
```json
root@cephadm01:/# radosgw-admin bucket list
[
    "bucket-test",
    "bk-smartcloud"
]
```

### 2. List bucket của 1 user
```
radosgw-admin bucket list --uid=<user_id>
```

Ví dụ:
```json
root@cephadm01:/# radosgw-admin bucket list --uid="haidd"
[
    "bucket-haidd"
]
```



# Tham khảo

- https://docs.ceph.com/en/latest/man/8/radosgw-admin/