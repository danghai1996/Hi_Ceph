# Cấu hình HTTPS cho Ceph Object Gateway

Sinh cert self-site cho Ceph:
```
openssl req -x509 -nodes -days 1095 \
 -newkey rsa:4096 -keyout rgw.key -out rgw.pem

cat rgw.key >> rgw.pem

mv rgw.pem /etc/ceph/rgw.pem
```

Copy cert sang node còn lại.


File config `ceph.conf` cần bổ sung các cấu hình tương ứng với từng node được thiết lập Object Gateway
```ini
[client.rgw.ceph01.rgw0] #có thể tên khác theo cấu hình
rgw_frontends = beast port=80 ssl_port=443 ssl_certificate=/etc/ceph/rgw.pem
#rgw_resolve_cname = true
rgw_dns_name = s3.cloudvnpt.com
```

```ini
[client.rgw.ceph02.rgw0] #có thể tên khác theo cấu hình
rgw_frontends = beast port=80 ssl_port=443 ssl_certificate=/etc/ceph/rgw.pem
#rgw_resolve_cname = true
rgw_dns_name = s3.cloudvnpt.com
```

Restart service radosgateway trên từng node:
```
systemctl restart ceph-radosgw@rgw.ceph01.rgw0.service
```
```
systemctl restart ceph-radosgw@rgw.ceph02.rgw0.service
```

## Cấu hình HAproxy
Phiên bản HAProxy trong bài lab: 
```
[root@lb-ceph-s3 ~]# haproxy --version
HA-Proxy version 1.8.1 2017/12/03
Copyright 2000-2017 Willy Tarreau <willy@haproxy.org>
```

Cài đặt certbot:
```
yum install -y http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

yum install -y certbot
```

Sinh cert SSL cho tên miền cấu hình cho dịch vụ Object Storage:
```
certbot certonly --standalone -d s3.cloudvnpt.com --staple-ocsp -m dangdohai1996@gmail.com --agree-tos
```

Tạo thư mục chứa cert cho HAProxy. Tạo file `.pem` từ cert vừa tạo:
```
mkdir -p /etc/haproxy/certs
cat /etc/letsencrypt/live/s3.cloudvnpt.com/fullchain.pem /etc/letsencrypt/live/s3.cloudvnpt.com/privkey.pem > /etc/haproxy/certs/s3.cloudvnpt.com.pem
```

Tại đây, ta dùng HAproxy đóng vai trò làm LB cho các node Ceph Object Gateway. File cấu hình:
```conf
global
    daemon
    group haproxy
    log /dev/log local0
    log /dev/log local1 notice
    log 127.0.0.1 local2
    maxconn 16000
    pidfile /var/run/haproxy.pid
    stats socket /var/lib/haproxy/stats
    tune.bufsize 32768
    tune.maxrewrite 1024
    user haproxy


defaults
    log global
    maxconn 8000
    mode http
    option redispatch
    option http-server-close
    option splice-auto
    retries 3
    timeout http-request 20s
    timeout queue 1m
    timeout connect 10s
    timeout client 1m
    timeout server 1m
    timeout check 10s


listen stats
    bind 192.168.60.84:8080
    mode http
    stats enable
    stats uri /stats
    stats realm HAProxy\ Statistics
    stats refresh 10s
    stats show-node
    stats show-legends

frontend s3_http
  bind :80
  reqadd X-Forwarded-Proto:\ http
  default_backend rgw

frontend s3_https
  bind :443 ssl crt /etc/haproxy/certs/s3.cloudvnpt.com.pem
  reqadd X-Forwarded-Proto:\ https
#  default_backend rgw-https   #Nếu để https sẽ gặp lỗi timeout
  default_backend rgw

backend rgw
  mode http
  balance roundrobin
  server ceph01 192.168.60.74:80 check inter 5s
  server ceph02 192.168.60.75:80 check inter 5s

backend rgw-https
  mode http
  balance roundrobin
  server ceph01 192.168.60.74:443 check inter 5s
  server ceph02 192.168.60.75:443 check inter 5s
```

Restart HAProxy:
```
systemctl restart haproxy
```

# Tham khảo
- https://ksingh7.medium.com/https-medium-com-karansingh010-https-ization-of-ceph-object-storage-public-endpoint-82f7a1d18df4
- https://docs.ceph.com/en/pacific/radosgw/frontends/
- https://documentation.suse.com/ses/6/html/ses-all/cha-ceph-configuration.html#config-ogw-http-frontends
- https://documentation.suse.com/sle-ha/15-SP1/html/SLE-HA-all/cha-ha-lb.html#sec-ha-lb-haproxy
- https://documentation.suse.com/ses/6/html/ses-all/cha-ceph-gw.html#ogw-haproxy