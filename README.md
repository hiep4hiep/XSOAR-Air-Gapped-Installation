# XSOAR-Air-Gapped-Installation for RHEL 8
### Step 1 - Activate subscription for your server
- Go to Redhat website, create a System profile
- Attach subscription in Subscription tab
- Download the entitlement certificate > unzip and copy the .pem file to your server
- Run: `subscription-manager import --certificate=/tmp/Name_Of_Downloaded_Entitlement_Cert.pem`

### Step 2 - Create a offline repo
- Mount Redhat installation DVD ISO to your server. (e.g the DVD ISO is at /dev/sr0 or change that to suite your environment)

`echo "/dev/sr0   /media/iso                       iso9660     defaults        0 0" >> /etc/fstab`
Then reboot the server
- Check the content of /media/iso
`[root@localhost ~]# ll /media/iso
total 48
dr-xr-xr-x. 4 root root  2048 Oct 13 18:57 AppStream
dr-xr-xr-x. 4 root root  2048 Oct 13 18:57 BaseOS
dr-xr-xr-x. 3 root root  2048 Oct 13 18:57 EFI
-r--r--r--. 1 root root  8154 Oct 13 18:52 EULA
-r--r--r--. 1 root root  1455 Oct 13 18:52 extra_files.json
-r--r--r--. 1 root root 18092 Oct 13 18:52 GPL
dr-xr-xr-x. 3 root root  2048 Oct 13 18:57 images
dr-xr-xr-x. 2 root root  2048 Oct 13 18:57 isolinux
-r--r--r--. 1 root root   103 Oct 13 18:52 media.repo
-r--r--r--. 1 root root  1669 Oct 13 18:52 RPM-GPG-KEY-redhat-beta
-r--r--r--. 1 root root  5135 Oct 13 18:52 RPM-GPG-KEY-redhat-release
-r--r--r--. 1 root root  1796 Oct 13 18:57 TRANS.TBL`

- Remove everything in /etc/yum.repos.d `rm -f /etc/yum.repos.d/*`
- Create a new repo file `vi /etc/yum.repos.d/local.repo` with below content:
- 
`[LocalRepo_BaseOS]
name=LocalRepository_BaseOS
baseurl=file:///media/iso/BaseOS
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

[LocalRepo_AppStream]
name=LocalRepository_AppStream
baseurl=file:///media/iso/AppStream
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release`
