# Pool của Ceph Object Gateway

### 1. `.rgw.root`
- Các bản ghi về region, zone, globla information không xác định.
- Mỗi bản ghi cho một object

### 2. `<zone>.rgw.control`
- Pool điều khiển 

### 3. `<zone>.rgw.meta`
Pool chứa các namespace với các metadata.
- namespace: `root`
    - `<bucket> .bucket.meta.<bucket>:<marker>`
    - tenant được sử dụng để phân biệt các bucket, nhưng không phải là các bucket.

    Ví dụ:
    ```
    .bucket.meta.prodtx:test%25star:default.84099.6
    .bucket.meta.testcont:default.4126.1
    .bucket.meta.prodtx:testcont:default.84099.4
    prodtx/testcont
    prodtx/test%25star
    testcont
    ```
    
- namespace: `users.uid`: pool user ID chứa bản đồ các ID của user.
- namespace: `users.email`: chứa thông tin email của user.
- namespace: `users.keys`: chứa access key và secret key của user
- namespace: `users.swift`: chứa thông tin Swift subuser của 1 user.

# Tham khảo
- https://docs.ceph.com/en/latest/radosgw/layout/
- https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/3/html/object_gateway_guide_for_red_hat_enterprise_linux/rgw-configuration-reference-rgw#about-pools-rgw