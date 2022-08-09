# Encryption

Ceph Object Gateway hỗ trợ việc mã hóa dữ liệu các object được upload lên. Với 3 option cho để quản lý en-cryption keys. 

Dữ liệu có thể gửi thông qua HTTP không được mã hóa, và Ceph Object Gateway sẽ thực hiện lưu trữ dữ liệu đó trong cụm Ceph Cluster dưới dạng mã hóa.

**Lưu ý:** Ecryption key phải dài 256bits và được mã hóa base64.



# Tham khảo:
- https://docs.ceph.com/en/pacific/radosgw/encryption/