-- MySQL安装步骤 
fdisk -l
df -h
mkfs.ext4 /dev/vdb 
mkdir /database
mount /dev/vdb /database/
vim /etc/rc.local  -- 添加 mount /dev/vdb /database
cat /etc/my_aucs.cnf 
mv /etc/my_aucs.cnf /etc/my_monitor.cnf
sed -i 's/aucs/monitor/g' /etc/my_monitor.cnf 
cat /etc/my_monitor.cnf
mkdir -p /database/monitor/{data,binlog,relaylog,backup}
id mysql
chown -R mysql.mysql /database/
cd /usr/local/
tar xzvf mysql-5.6.29-linux-glibc2.5-x86_64.tar.gz 
mv mysql-5.6.29-linux-glibc2.5-x86_64 mysql
chown -R mysql.mysql mysql
cd mysql
./scripts/mysql_install_db --defaults-file=/etc/my_monitor.cnf --user=mysql &
./bin/mysqld_safe --defaults-file=/etc/my_monitor.cnf --user=mysql &
ln -s /usr/local/mysql/bin/mysql* /usr/bin/
mysql -V
mysql -uroot -p -S /tmp/mysql_monitor.sock 
mysql -uzhangjl -p -S /tmp/mysql_monitor.sock 



-- pmm-server 安装步骤
#uname -vr
#yum grouplist
#yum grouplist |more 
yum remove docker
yum -y install  docker-io
service docker start   -- 或者/etc/init.d/docker start


---------------------------------------------遇到问题后解决过程，无效。。。。---------------------------------------------------------------------------------------------------------------
-- 保证可联网，以下报错表示不能上网。
[root@DB55 yum.repos.d]# docker create \
>    -v /opt/prometheus/data \
>    -v /opt/consul-data \
>    -v /var/lib/mysql \
>    -v /var/lib/grafana \
>    --name pmm-data \
>    percona/pmm-server:1.2.0 /bin/true
Unable to find image 'percona/pmm-server:1.2.0' locally
Pulling repository percona/pmm-server
Get https://index.docker.io/v1/repositories/percona/pmm-server/images: dial tcp 34.200.194.233:443: connection timed out

[root@DB55 mysql]# cat /etc/resolv.conf 
options timeout:1 attempts:1 rotate single-request-reopen
nameserver 100.100.2.136
nameserver 100.100.2.138

[root@DB55 mysql]# echo "nameserver 8.8.8.8" > /etc/resolv.conf


[root@DB55 mysql]# nslookup index.docker.io
Server:         8.8.8.8
Address:        8.8.8.8#53

Non-authoritative answer:
index.docker.io canonical name = elb-io.us-east-1.aws.dckr.io.
elb-io.us-east-1.aws.dckr.io    canonical name = us-east-1-elbio-rm5bon1qaeo4-623296237.us-east-1.elb.amazonaws.com.
Name:   us-east-1-elbio-rm5bon1qaeo4-623296237.us-east-1.elb.amazonaws.com
Address: 52.200.132.201
Name:   us-east-1-elbio-rm5bon1qaeo4-623296237.us-east-1.elb.amazonaws.com
Address: 34.200.194.233
Name:   us-east-1-elbio-rm5bon1qaeo4-623296237.us-east-1.elb.amazonaws.com
Address: 52.87.47.61

echo '52.87.47.61 index.docker.io' >> /etc/hosts

------------------------------------------------------------------------------------------------------------------------------------------------------------

-- 拉取服务端pmm-server镜像 。在Centos6遇到问题，一直无法正常拉取，会出现中断情况。后来系统版本换成Centos7拉取成功。默认本地不存pmm-server，会从官网下载但是比较慢。

docker pull percona/pmm-server

#git clone https://github.com/percona/pmm-server.git


#docker create -v /opt/prometheus/data  -v /opt/consul-data  -v /usr/local/mysql  -v /var/lib/grafana --name pmm-data percona/pmm-server:1.2.0 /bin/true
docker create -v /opt/prometheus/data  -v /opt/consul-data  -v /var/lib/mysql  -v /var/lib/grafana  --name pmm-data    percona/pmm-server /bin/true

docker run -d -p 8080:80  --volumes-from pmm-data --name pmm-server --restart always percona/pmm-server


[root@izbp117baq7y62i7k8manfz lib]# docker images
REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
<none>                         <none>              05e908874a22        18 hours ago        375.6 MB
docker.io/percona/pmm-server   latest              eb82a0e154c8        2 weeks ago         1.266 GB
docker.io/centos               latest              36540f359ca3        3 weeks ago         192.5 MB


[root@izbp117baq7y62i7k8manfz ~]# docker inspect pmm-data
[
    {
        "Id": "b8ff1fec96afe64431a17582949c2a4564f40567e7eb863fb0f177a7c7f1c459",
        "Created": "2017-08-02T00:35:50.69964356Z",
        "Path": "/bin/true",
        "Args": [],
        "State": {
            "Status": "created",
            "Running": false,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 0,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "0001-01-01T00:00:00Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:eb82a0e154c810e42bd8f1506696f88e10719b6c5c87ad0aa21d15096c6b6b74",
        "ResolvConfPath": "",
        "HostnamePath": "",
        "HostsPath": "",
        "LogPath": "",
        "Name": "/pmm-data",
        "RestartCount": 0,
        "Driver": "devicemapper",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "docker-runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": -1,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0
        },
        "GraphDriver": {
            "Name": "devicemapper",
            "Data": {
                "DeviceId": "36",
                "DeviceName": "docker-253:1-132971-fcb29d4645d296034f08c235ac50ad21fb4f43720fb0ee124bfff8ca45690528",
                "DeviceSize": "10737418240"
            }
        },
        "Mounts": [
            {
                "Name": "877bdbf30ba40472e7cdebad0d2b78188085b4779d81ae4e7b49975d2baf7b64",
                "Source": "/var/lib/docker/volumes/877bdbf30ba40472e7cdebad0d2b78188085b4779d81ae4e7b49975d2baf7b64/_data",
                "Destination": "/var/lib/grafana",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Name": "b3a9eac5e5edb57d8639383a54e01201a5d2e62bfe14272a60b47c20d15ed125",
                "Source": "/var/lib/docker/volumes/b3a9eac5e5edb57d8639383a54e01201a5d2e62bfe14272a60b47c20d15ed125/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Name": "7eab7c950737c5648735fcda7e67b37ebce03cbf2a12c71f114238479cb79a36",
                "Source": "/var/lib/docker/volumes/7eab7c950737c5648735fcda7e67b37ebce03cbf2a12c71f114238479cb79a36/_data",
                "Destination": "/opt/consul-data",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Name": "3a78d38867b329028504509fb1a3727cb3a1c9e88cb84e179354ea9c1738fb14",
                "Source": "/var/lib/docker/volumes/3a78d38867b329028504509fb1a3727cb3a1c9e88cb84e179354ea9c1738fb14/_data",
                "Destination": "/opt/prometheus/data",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "b8ff1fec96af",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": true,
            "AttachStderr": true,
            "ExposedPorts": {
                "443/tcp": {},
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/true"
            ],
            "Image": "percona/pmm-server",
            "Volumes": {
                "/opt/consul-data": {},
                "/opt/prometheus/data": {},
                "/var/lib/grafana": {},
                "/var/lib/mysql": {}
            },
            "WorkingDir": "/opt",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "build-date": "20161214",
                "license": "GPLv2",
                "name": "CentOS Base Image",
                "vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": null,
            "SandboxKey": "",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "",
                    "EndpointID": "",
                    "Gateway": "",
                    "IPAddress": "",
                    "IPPrefixLen": 0,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": ""
                }
            }
        }
    }
]





[root@izbp117baq7y62i7k8manfz ~]# docker inspect pmm-server
[
    {
        "Id": "2b830a5aa4e7081ba22b4bf68e73a115ff470a562c06dc6ccf66eed8b63d7d2a",
        "Created": "2017-08-02T00:36:14.210890527Z",
        "Path": "/opt/entrypoint.sh",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 17114,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2017-08-02T00:36:14.938575229Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:eb82a0e154c810e42bd8f1506696f88e10719b6c5c87ad0aa21d15096c6b6b74",
        "ResolvConfPath": "/var/lib/docker/containers/2b830a5aa4e7081ba22b4bf68e73a115ff470a562c06dc6ccf66eed8b63d7d2a/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/2b830a5aa4e7081ba22b4bf68e73a115ff470a562c06dc6ccf66eed8b63d7d2a/hostname",
        "HostsPath": "/var/lib/docker/containers/2b830a5aa4e7081ba22b4bf68e73a115ff470a562c06dc6ccf66eed8b63d7d2a/hosts",
        "LogPath": "/var/lib/docker/containers/2b830a5aa4e7081ba22b4bf68e73a115ff470a562c06dc6ccf66eed8b63d7d2a/2b830a5aa4e7081ba22b4bf68e73a115ff470a562c06dc6ccf66eed8b63d7d2a-json.log",
        "Name": "/pmm-server",
        "RestartCount": 0,
        "Driver": "devicemapper",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "80/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "8080"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "always",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": [
                "pmm-data"
            ],
            "CapAdd": null,
            "CapDrop": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "docker-runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": null,
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DiskQuota": 0,
            "KernelMemory": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": -1,
            "OomKillDisable": false,
            "PidsLimit": 0,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0
        },
        "GraphDriver": {
            "Name": "devicemapper",
            "Data": {
                "DeviceId": "38",
                "DeviceName": "docker-253:1-132971-55537da00b6a089229ee3066b4e7e48b8f484f306be6266f33532883f5515a47",
                "DeviceSize": "10737418240"
            }
        },
        "Mounts": [
            {
                "Name": "b3a9eac5e5edb57d8639383a54e01201a5d2e62bfe14272a60b47c20d15ed125",
                "Source": "/var/lib/docker/volumes/b3a9eac5e5edb57d8639383a54e01201a5d2e62bfe14272a60b47c20d15ed125/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Name": "7eab7c950737c5648735fcda7e67b37ebce03cbf2a12c71f114238479cb79a36",
                "Source": "/var/lib/docker/volumes/7eab7c950737c5648735fcda7e67b37ebce03cbf2a12c71f114238479cb79a36/_data",
                "Destination": "/opt/consul-data",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Name": "3a78d38867b329028504509fb1a3727cb3a1c9e88cb84e179354ea9c1738fb14",
                "Source": "/var/lib/docker/volumes/3a78d38867b329028504509fb1a3727cb3a1c9e88cb84e179354ea9c1738fb14/_data",
                "Destination": "/opt/prometheus/data",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Name": "877bdbf30ba40472e7cdebad0d2b78188085b4779d81ae4e7b49975d2baf7b64",
                "Source": "/var/lib/docker/volumes/877bdbf30ba40472e7cdebad0d2b78188085b4779d81ae4e7b49975d2baf7b64/_data",
                "Destination": "/var/lib/grafana",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "2b830a5aa4e7",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "443/tcp": {},
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/opt/entrypoint.sh"
            ],
            "Image": "percona/pmm-server",
            "Volumes": null,
            "WorkingDir": "/opt",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "build-date": "20161214",
                "license": "GPLv2",
                "name": "CentOS Base Image",
                "vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "c2c1b4ccfe08ef1ef81a019d55bab77fd09deb20466e1741ff9747608a27111e",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "443/tcp": null,
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "8080"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/c2c1b4ccfe08",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "56926ee9be575097dbb26453eccfba8c159b2569bcf73b960a277dd4dffb5680",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "9e7a665d12c50c21e0357dbe210157cb34ee7ac6875b96c3fc8a5f27d8ccc7ad",
                    "EndpointID": "56926ee9be575097dbb26453eccfba8c159b2569bcf73b960a277dd4dffb5680",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02"
                }
            }
        }
    }
]



[root@izbp117baq7y62i7k8manfz ~]# docker exec -i -t pmm-server /bin/bash


[root@2b830a5aa4e7 opt]# ll /etc/grafana/grafana.ini 
-rw-r--r-- 1 grafana grafana 11911 Jul 13 08:20 /etc/grafana/grafana.ini
[root@2b830a5aa4e7 opt]# vi /etc/grafana/grafana.ini    -- 修改默认type 类型，由sqlite3 >>> mysql 

# Either "mysql", "postgres" or "sqlite3", it's your choice
;type = mysql
;host = 127.0.0.1:3306
;name = grafana
;user = root
# If the password contains # or ; you have to wrap it with trippel quotes. Ex """#password;"""
;password =



[root@izbp117baq7y62i7k8manfz lib]# netstat -ntpl 
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 10.100.15.60:42000      0.0.0.0:*               LISTEN      19975/node_exporter 
tcp        0      0 10.100.15.60:42002      0.0.0.0:*               LISTEN      20039/mysqld_export 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      2210/sshd           
tcp6       0      0 :::8080                 :::*                    LISTEN      25745/docker-proxy- 
tcp6       0      0 :::33061                :::*                    LISTEN      19924/mysqld        



[root@2b830a5aa4e7 opt]# mysql -uroot -p -S /var/lib/mysql/mysql.sock 
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 766
Server version: 5.5.55-38.8 Percona Server (GPL), Release 38.8, Revision 11f5bbd

Copyright (c) 2009-2017 Percona LLC and/or its affiliates
Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 


-- 重启  pmm-server 服务
[root@izbp117baq7y62i7k8manfz docker]#  docker restart pmm-server  
pmm-server
[root@izbp117baq7y62i7k8manfz docker]# ps -ef|grep docker
root     17844     1  0 09:22 ?        00:00:00 /usr/bin/dockerd-current --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current --default-runtime=docker-runc --exec-opt native.cgroupdriver=systemd --userland-proxy-path=/usr/libexec/docker/docker-proxy-current
root     17852 17844  0 09:22 ?        00:00:00 /usr/bin/docker-containerd-current -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --shim docker-containerd-shim --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --runtime docker-runc --runtime-args --systemd-cgroup=true
root     18357 17844  0 09:22 ?        00:00:00 /usr/libexec/docker/docker-proxy-current -proto tcp -host-ip 0.0.0.0 -host-port 8080 -container-ip 172.17.0.2 -container-port 80
root     18383 17852  0 09:22 ?        00:00:00 /usr/bin/docker-containerd-shim-current 2b830a5aa4e7081ba22b4bf68e73a115ff470a562c06dc6ccf66eed8b63d7d2a /var/run/docker/libcontainerd/2b830a5aa4e7081ba22b4bf68e73a115ff470a562c06dc6ccf66eed8b63d7d2a /usr/libexec/docker/docker-runc-current
dockerr+ 18508 18424  1 09:22 ?        00:00:00 /usr/sbin/grafana-server --homepath=/usr/share/grafana --config=/etc/grafana/grafana.ini cfg:default.paths.data=/var/lib/grafana cfg:default.paths.logs=/var/log/grafana cfg:default.paths.plugins=/var/lib/grafana/plugins cfg:default.server.root_url=%(protocol)s://%(domain)s:%(http_port)s/graph ENV_AUTH_BASIC
root     18608  2271  0 09:23 pts/0    00:00:00 grep --color=auto docker




-- 客户端配置

-- 10.100.15.60 client 配置

wget https://www.percona.com/downloads/pmm-client/pmm-client-1.2.0/binary/redhat/6/x86_64/pmm-client-1.2.0-1.x86_64.rpm 
rpm -ivh pmm-client-1.2.0-1.x86_64.rpm 


[root@izbp117baq7y62i7k8manfz lib]# pmm-admin config --server 10.100.15.60:8080
OK, PMM server is alive.

PMM Server      | 10.100.15.60:8080 
Client Name     | izbp117baq7y62i7k8manfz
Client Address  | 10.100.15.60 



[root@izbp117baq7y62i7k8manfz ~]# pmm-admin list
pmm-admin 1.2.0

PMM Server      | 10.100.15.60:8080 
Client Name     | izbp117baq7y62i7k8manfz
Client Address  | 10.100.15.60 
Service Manager | linux-systemd

-------------- ------------------------ ----------- -------- ------------ --------
SERVICE TYPE   NAME                     LOCAL PORT  RUNNING  DATA SOURCE  OPTIONS 
-------------- ------------------------ ----------- -------- ------------ --------
linux:metrics  izbp117baq7y62i7k8manfz  42000       YES      -                    





[root@izbp117baq7y62i7k8manfz docker]# pmm-admin add mysql --help
This command adds the given MySQL instance to system, metrics and queries monitoring.

When adding a MySQL instance, this tool tries to auto-detect the DSN and credentials.
If you want to create a new user to be used for metrics collecting, provide --create-user option. pmm-admin will create
a new user 'pmm@' automatically using the given (auto-detected) MySQL credentials for granting purpose.

Table statistics is automatically disabled when there are more than 10000 tables on MySQL.

[name] is an optional argument, by default it is set to the client name of this PMM client.

Usage:
  pmm-admin add mysql [name] [flags]

Examples:
  pmm-admin add mysql --password abc123
  pmm-admin add mysql --password abc123 --create-user
  pmm-admin add mysql --password abc123 --port 3307 instance3307

Flags:
      --create-user                       create a new MySQL user
      --create-user-maxconn uint16        max user connections for a new user (default 10)
      --create-user-password string       optional password for a new MySQL user
      --defaults-file string              path to my.cnf
      --disable-binlogstats               disable binlog statistics
      --disable-processlist               disable process state metrics
      --disable-queryexamples             disable collection of query examples
      --disable-tablestats                disable table statistics
      --disable-tablestats-limit uint16   number of tables after which table stats are disabled automatically (default 1000)
      --disable-userstats                 disable user statistics
      --force                             force to create/update MySQL user
  -h, --help                              help for mysql
      --host string                       MySQL host
      --password string                   MySQL password
      --port string                       MySQL port
      --query-source string               source of SQL queries: auto, slowlog, perfschema (default "auto")
      --socket string                     MySQL socket
      --user string                       MySQL username

Global Flags:
  -c, --config-file string   PMM config file (default "/usr/local/percona/pmm-client/pmm.yml")
      --dev-enable           enable experimental features
      --service-port int     service port
      --verbose              verbose output
[root@izbp117baq7y62i7k8manfz docker]# pmm-admin add mysql --query-source perfschema  --user zhangjl --password zhangjl@che2017 --socket=/tmp/mysql_monitor.sock 
[linux:metrics] OK, already monitoring this system.
[mysql:metrics] OK, already monitoring MySQL metrics.
[mysql:queries] OK, already monitoring MySQL queries.
[root@izbp117baq7y62i7k8manfz docker]# 


[root@iZbp17vlqunx6iduomayheZ local]# ps -ef|grep pmm
root     14162     1  1 11:21 pts/2    00:03:56 /usr/local/percona/pmm-client/node_exporter -collectors.enabled=diskstats,filefd,filesystem,loadavg,meminfo,netdev,netstat,stat,time,uname,vmstat -web.listen-address=10.100.12.30:42000 -web.auth-file=/usr/local/percona/pmm-client/pmm.yml -web.ssl-cert-file=/usr/local/percona/pmm-client/server.crt -web.ssl-key-file=/usr/local/percona/pmm-client/server.key
root     14183     1 13 11:21 pts/2    00:27:07 /usr/local/percona/pmm-client/mysqld_exporter -collect.auto_increment.columns=true -collect.binlog_size=true -collect.global_status=true -collect.global_variables=true -collect.info_schema.innodb_metrics=true -collect.info_schema.processlist=true -collect.info_schema.query_response_time=true -collect.info_schema.tables=true -collect.info_schema.tablestats=true -collect.info_schema.userstats=true -collect.perf_schema.eventswaits=true -collect.perf_schema.file_events=true -collect.perf_schema.indexiowaits=true -collect.perf_schema.tableiowaits=true -collect.perf_schema.tablelocks=true -collect.slave_status=true -web.listen-address=10.100.12.30:42002 -web.auth-file=/usr/local/percona/pmm-client/pmm.yml -web.ssl-cert-file=/usr/local/percona/pmm-client/server.crt -web.ssl-key-file=/usr/local/percona/pmm-client/server.key




-- 添加 10.100.12.71 client monitor 配置
[root@DB71 local]# rpm -ivh pmm-client-1.2.0-1.x86_64.rpm 
warning: pmm-client-1.2.0-1.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:pmm-client             ########################################### [100%]
[root@DB71 local]# 
[root@DB71 local]# 
[root@DB71 local]# ps -ef|grep pmm
root     13578 13847  0 10:29 pts/3    00:00:00 grep pmm
[root@DB71 local]# pmm-admin add mysql --query-source perfschema  --user zhangjl --password zhangjl@che2017 --socket=/tmp/mysql_aucs.sock 
PMM client is not configured, missing config file. Please make sure you have run 'pmm-admin config'.
[root@DB71 local]# pmm-admin config --server 10.100.15.60:8080
\OK, PMM server is alive.

PMM Server      | 10.100.15.60:8080 
Client Name     | DB71
Client Address  | 10.100.12.71 
[root@DB71 local]# pmm-admin add mysql --query-source perfschema  --user zhangjl --password zhangjl@che2017 --socket=/tmp/mysql_aucs.sock 
[linux:metrics] OK, now monitoring this system.
[mysql:metrics] OK, now monitoring MySQL metrics using DSN zhangjl:***@unix(/tmp/mysql_aucs.sock)
[mysql:queries] OK, now monitoring MySQL queries from perfschema using DSN zhangjl:***@unix(/tmp/mysql_aucs.sock)
[root@DB71 local]# 



-- 添加 10.100.12.70 client monitor .在 10.100.12.70机器配置如下,注意 --query-source 配置为slowlog

[root@DB70 local]# rpm -ivh pmm-client-1.2.0-1.x86_64.rpm 
warning: pmm-client-1.2.0-1.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:pmm-client             ########################################### [100%]
[root@DB70 local]#  pmm-admin config --server 10.100.15.60:8080
OK, PMM server is alive.

PMM Server      | 10.100.15.60:8080 
Client Name     | DB70
Client Address  | 10.100.12.70 
[root@DB70 local]# pmm-admin add mysql --query-source slowlog  --user zhangjl --password zhangjl@che2017 --socket=/tmp/mysql_aucs.sock 
[linux:metrics] OK, now monitoring this system.
[mysql:metrics] OK, now monitoring MySQL metrics using DSN zhangjl:***@unix(/tmp/mysql_aucs.sock)
[mysql:queries] OK, now monitoring MySQL queries from perfschema using DSN zhangjl:***@unix(/tmp/mysql_aucs.sock)



-- 添加 10.100.12.30 client monitor .在 10.100.12.30 机器配置如下,注意 --client-name 修改主机名为 DB30
[root@iZbp17vlqunx6iduomayheZ local]# pmm-admin config --server 10.100.15.60:8080
OK, PMM server is alive.

PMM Server      | 10.100.15.60:8080 
Client Name     | iZbp17vlqunx6iduomayheZ
Client Address  | 10.100.12.30 
[root@iZbp17vlqunx6iduomayheZ local]# pmm-admin config --client-name DB30 --server 10.100.15.60:8080
OK, PMM server is alive.

PMM Server      | 10.100.15.60:8080 
Client Name     | DB30
Client Address  | 10.100.12.30 
[root@iZbp17vlqunx6iduomayheZ local]# pmm-admin add mysql --query-source slowlog  --user zhangjl --password zhangjl@che2017 --socket=/tmp/mysql.sock 
[linux:metrics] OK, now monitoring this system.
[mysql:metrics] OK, now monitoring MySQL metrics using DSN zhangjl:***@unix(/tmp/mysql.sock)
[mysql:queries] OK, now monitoring MySQL queries from slowlog using DSN zhangjl:***@unix(/tmp/mysql.sock)


此时排查发现10.100.12.30 iptables 有做限制
[root@iZbp17vlqunx6iduomayheZ local]# cat /etc/sysconfig/iptables  -- 添加如下内容：

-A INPUT -s 10.100.15.60 -p tcp -m tcp --dport 42002 -j ACCEPT
-A INPUT -s 10.100.15.60 -p tcp -m tcp --dport 8080 -j ACCEPT
-A INPUT -s 10.100.15.60 -p tcp -m tcp --dport 42000 -j ACCEPT


-- 重启防火墙
[root@iZbp17vlqunx6iduomayheZ local]# /etc/init.d/iptables restart
iptables: Setting chains to policy ACCEPT: filter          [  OK  ]
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Unloading modules:                               [  OK  ]
iptables: Applying firewall rules:                         [  OK  ]


[root@iZbp17vlqunx6iduomayheZ local]# pmm-admin list
pmm-admin 1.2.0

PMM Server      | 10.100.15.60:8080 
Client Name     | DB30
Client Address  | 10.100.12.30 
Service Manager | unix-systemv

-------------- ----- ----------- -------- ---------------------------------- ------------------------------------------
SERVICE TYPE   NAME  LOCAL PORT  RUNNING  DATA SOURCE                        OPTIONS                                   
-------------- ----- ----------- -------- ---------------------------------- ------------------------------------------
mysql:queries  DB30  -           YES      zhangjl:***@unix(/tmp/mysql.sock)  query_source=slowlog, query_examples=true 
linux:metrics  DB30  42000       YES      -                                                                            
mysql:metrics  DB30  42002       YES      zhangjl:***@unix(/tmp/mysql.sock)                   




-- 添加 10.100.12.40 client monitor .在 10.100.12.40 机器配置如下,注意 --client-name 修改主机名为 DB40

[root@iZbp1g3u73g9nxqbval97tZ local]# rpm -ivh pmm-client-1.2.0-1.x86_64.rpm 
warning: pmm-client-1.2.0-1.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:pmm-client             ########################################### [100%]
[root@iZbp1g3u73g9nxqbval97tZ local]# 
[root@iZbp1g3u73g9nxqbval97tZ local]# 
[root@iZbp1g3u73g9nxqbval97tZ local]# 
[root@iZbp1g3u73g9nxqbval97tZ local]# ps -ef|grep pmm
root     28714 22323  0 14:43 pts/1    00:00:00 grep pmm

[root@iZbp1g3u73g9nxqbval97tZ local]# pmm-admin config --client-name DB40 --server 10.100.15.60:8080  --配置客户端名称
OK, PMM server is alive.

PMM Server      | 10.100.15.60:8080 
Client Name     | DB40
Client Address  | 10.100.12.40 

[root@iZbp1g3u73g9nxqbval97tZ local]# pmm-admin add mysql --query-source slowlog  --user zhangjl --password zhangjl@che2017 --socket=/tmp/mysql.sock 
[linux:metrics] OK, now monitoring this system.
[mysql:metrics] OK, now monitoring MySQL metrics using DSN zhangjl:***@unix(/tmp/mysql.sock)
[mysql:queries] OK, now monitoring MySQL queries from slowlog using DSN zhangjl:***@unix(/tmp/mysql.sock)



-- 添加 10.100.12.20 client monitor .在 10.100.12.20 机器配置如下,注意 --client-name 修改主机名为 DB20

[root@iZbp10yn32hnb3iufubeksZ /usr/local]# rpm -ivh pmm-client-1.2.0-1.x86_64.rpm 
warning: pmm-client-1.2.0-1.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:pmm-client             ########################################### [100%]
[root@iZbp10yn32hnb3iufubeksZ /usr/local]# echo $?
0
[root@iZbp10yn32hnb3iufubeksZ /usr/local]#  pmm-admin config --client-name DB20 --server 10.100.15.60:8080
OK, PMM server is alive.

PMM Server      | 10.100.15.60:8080 
Client Name     | DB20
Client Address  | 10.100.12.20 
             
[root@iZbp10yn32hnb3iufubeksZ /usr/local]# pmm-admin add mysql --query-source slowlog  --user zhangjl --password zhangjl@che2017 --socket=/tmp/mysql.sock 
[linux:metrics] OK, now monitoring this system.
[mysql:metrics] OK, now monitoring MySQL metrics using DSN zhangjl:***@unix(/tmp/mysql.sock)
[mysql:queries] OK, now monitoring MySQL queries from slowlog using DSN zhangjl:***@unix(/tmp/mysql.sock)



-- 添加 10.100.12.35 client monitor .在 10.100.12.35 机器配置如下,注意 --client-name 修改主机名为 DB35

[root@iZ23c1lbu1qZ local]# rpm -ivh pmm-client-1.2.0-1.x86_64.rpm 
warning: pmm-client-1.2.0-1.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:pmm-client             ########################################### [100%]
[root@iZ23c1lbu1qZ local]# iptables-save
# Generated by iptables-save v1.4.7 on Wed Aug  2 15:54:49 2017
*filter
:INPUT ACCEPT [2946188597:350827865702]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [2717324391:818215623029]
COMMIT
# Completed on Wed Aug  2 15:54:49 2017
[root@iZ23c1lbu1qZ local]# 
[root@iZ23c1lbu1qZ local]#  pmm-admin config --client-name DB35 --server 10.100.15.60:8080
OK, PMM server is alive.

PMM Server      | 10.100.15.60:8080 
Client Name     | DB35
Client Address  | 10.100.12.35 
  
[root@iZ23c1lbu1qZ local]# pmm-admin add mysql --query-source slowlog  --user zhangjl --password zhangjl@che2017 --socket=/tmp/mysql.sock 
[linux:metrics] OK, now monitoring this system.
[mysql:metrics] OK, now monitoring MySQL metrics using DSN zhangjl:***@unix(/tmp/mysql.sock)
[mysql:queries] OK, now monitoring MySQL queries from slowlog using DSN zhangjl:***@unix(/tmp/mysql.sock)





-- 添加 10.100.12.40 client monitor .在 10.100.12.40 机器配置如下,注意 --client-name 修改主机名为 DB40
[root@iZbp1g3u73g9nxqbval97tZ local]# rpm -ivh pmm-client-1.2.0-1.x86_64.rpm 
warning: pmm-client-1.2.0-1.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID cd2efd2a: NOKEY
Preparing...                ########################################### [100%]
   1:pmm-client             ########################################### [100%]


[root@iZbp1g3u73g9nxqbval97tZ local]# pmm-admin config --server 10.100.15.60:8080
OK, PMM server is alive.

PMM Server      | 10.100.15.60:8080 
Client Name     | iZbp1g3u73g9nxqbval97tZ
Client Address  | 10.100.12.40 

[root@iZbp1g3u73g9nxqbval97tZ local]#  pmm-admin list
pmm-admin 1.2.0

PMM Server      | 10.100.15.60:8080 
Client Name     | iZbp1g3u73g9nxqbval97tZ
Client Address  | 10.100.12.40 
Service Manager | unix-systemv

No monitoring registered for this node identified as 'iZbp1g3u73g9nxqbval97tZ'.

[root@iZbp1g3u73g9nxqbval97tZ local]# pmm-admin config --client-name DB40 --server 10.100.15.60:8080
OK, PMM server is alive.

PMM Server      | 10.100.15.60:8080 
Client Name     | DB40
Client Address  | 10.100.12.40 
               
[root@iZbp1g3u73g9nxqbval97tZ local]# pmm-admin add mysql --query-source slowlog  --user zhangjl --password zhangjl@che2017 --socket=/tmp/mysql.sock 
[linux:metrics] OK, now monitoring this system.
[mysql:metrics] OK, now monitoring MySQL metrics using DSN zhangjl:***@unix(/tmp/mysql.sock)
[mysql:queries] OK, now monitoring MySQL queries from slowlog using DSN zhangjl:***@unix(/tmp/mysql.sock)



-- 10.100.15.60 机器强制修改 client-name ,由于之前主机名无法识别

[root@izbp117baq7y62i7k8manfz lib]# pmm-admin list
pmm-admin 1.2.0

PMM Server      | 10.100.15.60:8080 
Client Name     | izbp117baq7y62i7k8manfz
Client Address  | 10.100.15.60 
Service Manager | linux-systemd

-------------- ------------------------ ----------- -------- ------------------------------------------ ---------------------------------------------
SERVICE TYPE   NAME                     LOCAL PORT  RUNNING  DATA SOURCE                                OPTIONS                                      
-------------- ------------------------ ----------- -------- ------------------------------------------ ---------------------------------------------
mysql:queries  izbp117baq7y62i7k8manfz  -           YES      zhangjl:***@unix(/tmp/mysql_monitor.sock)  query_source=perfschema, query_examples=true 
linux:metrics  izbp117baq7y62i7k8manfz  42000       YES      -                                                                                       
mysql:metrics  izbp117baq7y62i7k8manfz  42002       YES      zhangjl:***@unix(/tmp/mysql_monitor.sock)                                               


[root@izbp117baq7y62i7k8manfz lib]# pmm-admin config --force --client-name DB15.60 --server 10.100.15.60:8080
OK, PMM server is alive.

PMM Server      | 10.100.15.60:8080 
Client Name     | DB15.60
Client Address  | 10.100.15.60 
[root@izbp117baq7y62i7k8manfz lib]# pmm-admin add mysql --query-source slowlog  --user zhangjl --password zhangjl@che2017 --socket=/tmp/mysql_monitor.sock 
[linux:metrics] OK, already monitoring this system.
[mysql:metrics] OK, already monitoring MySQL metrics.
[mysql:queries] OK, already monitoring MySQL queries.


