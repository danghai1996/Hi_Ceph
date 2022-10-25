# Deploy Ceph Object Gateway với `cephadm`

## Cách 1. Khởi tạo RGW
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

## Cách 2. Khởi tạo RGW sử dụng file service specification
Khởi tạo file `rgw.yaml`:
```yaml
service_type: rgw
service_id: vnpt.hanoi
placement:
  hosts:
  - cephadm-01
  - cephadm-02
  - cephadm-03
  count_per_host: 1
spec:
  rgw_realm: vnpt
  rgw_zone: hanoi
  rgw_frontend_port: 80
  rgw_frontend_type: beast
networks:
  - 192.168.60.0/24
```
**Trong đó:**
- `service_id`: tên service của rgw. Ghép theo cấu trúc `<rgw_realm>.<rgw.zone>`
- `rgw_frontend_port`: Port sử dụng cho RGW
- `rgw_frontend_ssl_certificate`: Danh sách các cert SSL sử dụng
- `rgw_frontend_type`: loại frontend sử dụng. Có thể là `civetweb` hoặc `beast`. Mặc định là `beast`.
- `rgw_realm`: RGW realm được sử dụng cho RGW. Cần khởi tạo thủ công.
- `rgw_zone`: Tên zone cho RGW. Cũng giống Realm là cần khởi tạo thủ công.
- `ssl: true` : Bật SSL
- `count_per_host` : Số lượng Ceph Object Gateway được triển khai trên mỗi node được khai báo. Để có hiệu quả tốt nhất về chi phí, xét giá trị là **2**. [Tham khảo](https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/5/html/operations_guide/management-of-ceph-object-gateway-services-using-the-ceph-orchestrator#deploying-the-ceph-object-gateway-using-the-service-specification_ops)
- `networks`: dải network sử dụng cho các object gateway.

Khởi tạo realm, groupzone, zone cho dịch vụ RGW:
```
radosgw-admin realm create --rgw-realm=vnpt
radosgw-admin zonegroup create --rgw-zonegroup=vnpt-s3 --default
radosgw-admin zone create --rgw-zonegroup=vnpt-s3 --rgw-zone=hanoi --default
radosgw-admin period update --rgw-realm=vnpt --commit
```

Xóa các zonegroup, zone default:
```
radosgw-admin zone delete --rgw-zone=default
radosgw-admin period update --commit
radosgw-admin zonegroup delete --rgw-zonegroup=default
radosgw-admin period update --commit
```


Triển khai Ceph Object Gateway sử dụng service specification:
```
ceph orch apply -i rgw.yaml 
```

Kiểm tra:
```
ceph orch ps | grep rgw

rgw.vnpt.hanoi.cephadm-01.yatase  cephadm-01  192.168.60.71:80  running (2m)     2m ago   2m    16.3M        -  16.2.10  0d668911f040  ee4ffc4bb090
rgw.vnpt.hanoi.cephadm-02.eojccp  cephadm-02  192.168.60.72:80  running (2m)     2m ago   2m    16.5M        -  16.2.10  0d668911f040  6218ea65e2d8
rgw.vnpt.hanoi.cephadm-03.zemvbi  cephadm-03  192.168.60.73:80  running (2m)     2m ago   2m    15.9M        -  16.2.10  0d668911f040  9209701f9cd0
```

Kiểm tra các pools:
```
--- RAW STORAGE ---
CLASS     SIZE    AVAIL    USED  RAW USED  %RAW USED
hdd    180 GiB  120 GiB  60 GiB    60 GiB      33.34
TOTAL  180 GiB  120 GiB  60 GiB    60 GiB      33.34

--- POOLS ---
POOL                     ID  PGS   STORED  OBJECTS     USED  %USED  MAX AVAIL
device_health_metrics     1    1      0 B        0      0 B      0     37 GiB
.rgw.root                 2   32  5.4 KiB       18  204 KiB      0     37 GiB
default.rgw.log           3   32    182 B        2   24 KiB      0     37 GiB
default.rgw.control       4   32      0 B        8      0 B      0     37 GiB
default.rgw.meta          5    8      0 B        0      0 B      0     37 GiB
hanoi.rgw.log             6   32  3.7 KiB      179  420 KiB      0     37 GiB
hanoi.rgw.control         7   32      0 B        8      0 B      0     37 GiB
hanoi.rgw.meta            8    8  1.3 KiB        7   72 KiB      0     37 GiB
hanoi.rgw.buckets.index   9   14      0 B       11      0 B      0     37 GiB
hanoi.rgw.buckets.data   10   32  2.1 KiB        1   12 KiB      0     37 GiB
```

Ta thực hiện xóa các pool default:
```
ceph osd pool rm default.rgw.log default.rgw.log --yes-i-really-really-mean-it
ceph osd pool rm default.rgw.control default.rgw.control --yes-i-really-really-mean-it
ceph osd pool rm default.rgw.meta default.rgw.meta --yes-i-really-really-mean-it
```
Nếu gặp lỗi như dưới đây thì cần phải cấu hình cho phép xóa pools:
```
root@cephadm-01:~# ceph osd pool rm default.rgw.log default.rgw.log --yes-i-really-really-mean-it
Error EPERM: pool deletion is disabled; you must first set the mon_allow_pool_delete config option to true before you can destroy a pool
```
Cấu hình cho phép xóa pools:
```
ceph config set mon mon_allow_pool_delete true
```

Sau đó thực hiện lại các lệnh xóa. Có thể disable sau khi đã thực hiện xóa xong.

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
