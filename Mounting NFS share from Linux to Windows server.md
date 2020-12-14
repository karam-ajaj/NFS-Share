# Server
1. configure the host file
2. configure the exports
3. configure iptables
4. create share directory
5. restart services


server
```shell
mkdir /var/avamar/SQLACC
chown nfsnobody:nfsnobody /var/avamar/SQLACC/
chmod 755 /var/avamar/SQLACC/
vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
172.25.43.42    COPISHDBACC02
172.25.43.41    COPISHDBACC01
cat <<EOF>> /etc/exports
/var/avamar/SQLACC        COPISHDBACC01(rw,sync,no_root_squash,all_squash,anonuid=65534,anongid=65534)
/var/avamar/SQLACC        COPISHDBACC02(rw,sync,no_root_squash,all_squash,anonuid=65534,anongid=65534)
EOF
exportfs -a
systemctl restart nfs
systemctl restart rpcbind
sudo iptables -A INPUT -s 172.25.43.0/24 -j ACCEPT
iptables-save | sudo tee /etc/sysconfig/iptables-config
```


# client
Start Windows Powershell as the administrator
Verify that the feature is available using the command Get-WindowsFeature -Name NFS*
Run the command Install-WindowsFeature -Name NFS-Client to install the feature
Open command prompt as admin and run command nfsadmin client stop
Navigate to HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default
Add new DWORD 32-bit as follows 
AnonymousUID value 65534 binary
AnonymousGID value 65534 binary
In the command prompt opened as admin, type nfsadmin client start
Run the following command in a command prompt (not Powershell) to set the NFS configuration:
nfsadmin client localhost config fileaccess=755 SecFlavors=+sys -krb5 -krb5i
mount the share on Windows server: mount -o anon \\<nfs server>\<exported share path> <drive letter>:
