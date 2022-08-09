# Multi-Site Ceph Object Gateway

Ceph hỗ trợ triển khai, cấu hình Multi-site cho Ceph Object Gateway

- **Multi-zone**: Cấu hình bao gồm 1 zonegroup và nhiều zone trong đó, mỗi zone sẽ có 1 hoặc nhiều cá thể `ceph-radosgw` (`ceph-radosgw` instances). Mỗi zone sẽ có một cụm Ceph Storage Cluster đứng sau của riêng nó. Nhiều zone trong zonegroup cung cấp khả năng khắc phục thảm hỏa (DR-disaster recovery) cho zonegroup nếu 1 trong các zone gặp sự cố nghiêm trọng. Mỗi zone đang hoạt động và có thể nhận các thao tác write. Ngoài việc khắc phục thảm họa, multi active zone có thể đóng vai trò làm nền tảng cho các mạng phân phối nội dung (content delivery network)
- **Multi-zone-group**: Ceph Object Gateway hỗ trợ nhiêu zonegroup, mỗi zonegroup sẽ có 1 hoặc nhiêu zone. Các object lưu trữ trong 1 zonegroup 