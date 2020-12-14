# Server
```shell
yum -y install nfs-utils
systemctl enable nfs-server.service
systemctl start nfs-server.service
mkdir /var/nfs
chown nfsnobody:nfsnobody /var/nfs
chmod 755 /var/nfs
vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
172.25.44.11    COPISHAPPPRD01
172.25.44.12    COPISHAPPPRD02
172.25.44.50    COPISHNFSPRD01
cat <<EOF>> /etc/exports
/var/nfs        COPISHAPPPRD01(rw,sync,no_subtree_check)
/var/nfs        COPISHAPPPRD02(rw,sync,no_subtree_check)
EOF
exportfs -a
systemctl restart nfs
systemctl restart rpcbind
systemctl stop firewalld
systemctl disable firewalld
sudo iptables -A INPUT -s 172.25.44.0/24 -j ACCEPT
iptables-save | sudo tee /etc/sysconfig/iptables-config
```

Give 'root' also access
See: https://kerneltalks.com/troubleshooting/access-denied-error-in-nfs-for-root-account/

```shell
vi /etc/exports
/var/nfs        COPISHAPPPRD01(rw,sync,no_root_squash,no_subtree_check)
/var/nfs        COPISHAPPPRD02(rw,sync,no_root_squash,no_subtree_check)
exportfs -ra
```

# Client
```shell
yum -y install nfs-utils
mkdir /data/intershop/eserver1/share
vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
172.25.44.11    COPISHAPPPRD01
172.25.44.12    COPISHAPPPRD02
172.25.44.50    COPISHNFSPRD01
systemctl restart nfs
systemctl restart rpcbind
systemctl stop firewalld
systemctl disable firewalld
sudo iptables -A INPUT -s 172.25.44.0/24 -j ACCEPT
iptables-save | sudo tee /etc/sysconfig/iptables-config
mount COPISHNFSPRD01:/var/nfs /data/intershop/eserver1/share
cat <<EOF>> /etc/fstab
COPISHNFSPRD01:/var/nfs  /data/intershop/eserver1/share   nfs      rw,sync,hard,intr  0     0
EOF
```
