# Steps on the server
1. configure the host file
2. configure the exports
3. configure iptables
4. create share directory
5. restart services


# server
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

# Steps on the client
1. Start Windows Powershell as the administrator
2. Verify that the feature is available using the command Get-WindowsFeature -Name NFS*
3. Run the command
   - Install-WindowsFeature -Name NFS-Client to install the feature
4. Open command prompt as admin and run command
   - *nfsadmin client stop
5. Navigate to HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\ClientForNFS\CurrentVersion\Default
6. Add new DWORD 32-bit as follows
   - *AnonymousUID value 65534 binary*
   - *AnonymousGID value 65534 binary*
7. In the command prompt opened as admin, type nfsadmin client start
8. Run the following command in a command prompt (not Powershell) to set the NFS configuration:
   - *nfsadmin client localhost config fileaccess=755 SecFlavors=+sys -krb5 -krb5i*
9. mount the share on Windows server:
   - *mount -o anon \\<nfs server>\<exported share path> <drive letter>:*
