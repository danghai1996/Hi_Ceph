# Deploy CEPH Cluster sử dụng `cephadm` trên CentOS-8

# Mô hình triển khai
```
                                         |
            +----------------------------+----------------------------+
            |                            |                            |
            |192.168.60.65               |192.168.60.66               |192.168.60.67 
+-----------+-----------+    +-----------+-----------+    +-----------+-----------+
|      [cephadm01]      |    |      [cephadm02]      |    |      [cephadm03]      |
|                       +----+                       +----+                       |
|       CentOS 8        |    |       CentOS 8        |    |        CentOS 8       |
|                       |    |                       |    |                       |
+-----------------------+    +-----------------------+    +-----------------------+
```

# I. Cấu hình ban đầu
## 0. Update
> ### Thực hiện trên tất cả các node
```sh
dnf update -y
```

**Lưu ý:** Nếu gặp lỗi `Error: Failed to download metadata for repo 'appstream'` như dưới đây:
```sh
[root@cephadm01 ~]# dnf update -y
CentOS Linux 8 - AppStream                                                                                            94  B/s |  38  B     00:00
Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: No URLs in mirrorlist
```

Thực hiện chỉnh sửa cấu hỉnh Repo và thực hiện update lại:
```sh
cd /etc/yum.repos.d/
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*
```

## 1. Cấu hình hostname, file host trên các node
> ### Thực hiện đặt hostname trên từng node tương ứng
Node 1
```sh
hostnamectl set-hostname cephadm01
bash
```
Node 2
```sh
hostnamectl set-hostname cephadm02
bash
```
Node 1
```sh
hostnamectl set-hostname cephadm03
bash
```

> ### Thực hiện trên cả 3 node
```sh
echo "192.168.60.65 cephadm01" >> /etc/hosts
echo "192.168.60.66 cephadm02" >> /etc/hosts
echo "192.168.60.67 cephadm03" >> /etc/hosts
```

## 2. Disable selinux, firewalld
> ### Thực hiện trên tất cả các node
```sh
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

systemctl stop firewalld && systemctl disable firewalld

init 6
```

## 3. Cài đặt Docker-ce
> ### Thực hiện trên cả 3 node
Thực hiện cài đặt Docker-ce trên cả 3 node
```sh
yum install -y yum-utils

yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

yum install -y docker-ce docker-ce-cli containerd.io
```

Khởi động service docker
```sh
systemctl start docker
systemctl enable docker
systemctl status docker
```

# II. Cài đặt, cấu hình `cephadm`
## 1. Cài đặt `cephadm` trên một node
Thực hiện cài đặt `cephadm` trên 1 trong 3 node. Ở đây, thực hiện trên node 1.
> ### Thực hiện trên node 1
```sh
dnf -y install centos-release-ceph-octopus epel-release
dnf -y install cephadm
```

## 2. Bootstrap cluster
Khởi tạo một Ceph cluster mới:
```sh
mkdir -p /etc/ceph

cephadm bootstrap --mon-ip 192.168.60.65
```

```
Verifying podman|docker is present...
Verifying lvm2 is present...
Verifying time synchronization is in place...
Unit chronyd.service is enabled and running
Repeating the final host check...
podman|docker (/usr/bin/docker) is present
systemctl is present
lvcreate is present
Unit chronyd.service is enabled and running
Host looks OK
Cluster fsid: 8301d8d6-8a5d-11ec-933b-fa163e71af70
Verifying IP 192.168.60.65 port 3300 ...
Verifying IP 192.168.60.65 port 6789 ...
Mon IP 192.168.60.65 is in CIDR network 192.168.60.0/24
Pulling container image docker.io/ceph/ceph:v15...
Extracting ceph user uid/gid from container image...
Creating initial keys...
Creating initial monmap...
Creating mon...
Waiting for mon to start...
Waiting for mon...
mon is available
Assimilating anything we can from ceph.conf...
Generating new minimal ceph.conf...
Restarting the monitor...
Setting mon public_network...
Creating mgr...
Verifying port 9283 ...
Wrote keyring to /etc/ceph/ceph.client.admin.keyring
Wrote config to /etc/ceph/ceph.conf
Waiting for mgr to start...
Waiting for mgr...
mgr not available, waiting (1/10)...
mgr not available, waiting (2/10)...
mgr not available, waiting (3/10)...
mgr not available, waiting (4/10)...
mgr not available, waiting (5/10)...
mgr is available
Enabling cephadm module...
Waiting for the mgr to restart...
Waiting for Mgr epoch 5...
Mgr epoch 5 is available
Setting orchestrator backend to cephadm...
Generating ssh key...
Wrote public SSH key to to /etc/ceph/ceph.pub
Adding key to root@localhost's authorized_keys...
Adding host cephadm01...
Deploying mon service with default placement...
Deploying mgr service with default placement...
Deploying crash service with default placement...
Enabling mgr prometheus module...
Deploying prometheus service with default placement...
Deploying grafana service with default placement...
Deploying node-exporter service with default placement...
Deploying alertmanager service with default placement...
Enabling the dashboard module...
Waiting for the mgr to restart...
Waiting for Mgr epoch 13...
Mgr epoch 13 is available
Generating a dashboard self-signed certificate...
Creating initial admin user...
Fetching dashboard port number...
Ceph Dashboard is now available at:

             URL: https://cephadm01:8443/
            User: admin
        Password: fpfow3lj5r

You can access the Ceph CLI with:

        sudo /usr/sbin/cephadm shell --fsid 8301d8d6-8a5d-11ec-933b-fa163e71af70 -c /etc/ceph/ceph.conf -k /etc/ceph/ceph.client.admin.keyring

Please consider enabling telemetry to help improve Ceph:

        ceph telemetry on

For more information see:

        https://docs.ceph.com/docs/master/mgr/telemetry/

Bootstrap complete.
```


# Tham khảo
- https://docs.ceph.com/en/latest/cephadm/install/
- https://programmerall.com/article/74842092559/
- https://www.server-world.info/en/note?os=CentOS_8&p=ceph15&f=9
- https://techglimpse.com/failed-metadata-repo-appstream-centos-8/