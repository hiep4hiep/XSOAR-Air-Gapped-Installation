# XSOAR Air Gapped Installation for RHEL 8
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
```[root@localhost ~]# ll /media/iso
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
-r--r--r--. 1 root root  1796 Oct 13 18:57 TRANS.TBL
```

- Remove everything in /etc/yum.repos.d `rm -f /etc/yum.repos.d/*`
- Create a new repo file `vi /etc/yum.repos.d/local.repo` with below content:

```[LocalRepo_BaseOS]
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
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```
- Then yum update to make sure it is using the offline repos
```
yum clean all
yum update
```
### Step 3: Check Podman is available or install it
```
touch /etc/subuid /etc/subgid
yum -y module install container-tools
```

### Step 4: Install XSOAR
- Copy the XSOAR installation file (demisto-xxxx.sh) to server and run it
```chmod +x demisto-xxxx.sh
./demisto-xxxx.sh -- -tools=false
```

### Step 5: Load container images
- Copy the XSOAR container images tar file (dockers.tar) to server (e.g to /images/dockers.tar path)
```
chown demisto:demisto /images/dockers.tar # Change owner of this file to demisto:demisto
sudo -su demisto # To change to bash under demisto user
podman load -i /images/dockers.tar # It will take a while to load the 30GB file
```
- Verify the images are loaded (still run these command under **demisto** user)
```
bash-4.4$ podman image list
REPOSITORY                          TAG           IMAGE ID      CREATED        SIZE
localhost/demisto/python3           3.10.1.26972  33e3a7c3d40a  3 weeks ago    77.9 MB
localhost/demisto/py3-tools         0.0.1.26536   09696734e66b  6 weeks ago    129 MB
localhost/demisto/parse-emails      1.0.0.26393   b83f8a2a7b22  6 weeks ago    112 MB
localhost/demisto/pandas            1.0.0.26289   50b4b3bed2aa  7 weeks ago    181 MB
localhost/demisto/chromium          1.0.0.26112   285c0e9847e7  8 weeks ago    1.02 GB
localhost/demisto/python3           3.10.1.25933  d7b4ce8c51cd  2 months ago   76 MB
localhost/demisto/slackv3           1.0.0.25130   40929da38b0b  3 months ago   80.1 MB
localhost/demisto/python            2.7.18.24398  f0122ce9b81f  4 months ago   91.5 MB
localhost/demisto/python3           3.9.8.24399   d00197b251b6  4 months ago   68 MB
localhost/demisto/readpdf           1.0.0.24272   383aae9416c9  4 months ago   127 MB
localhost/demisto/netutils          1.0.0.24101   08e38f87b7da  6 months ago   83.7 MB
localhost/demisto/python3           3.9.7.24076   aa16d1d576b9  6 months ago   67.8 MB
localhost/demisto/python            2.7.18.24066  902d88a4524f  6 months ago   91.5 MB
localhost/demisto/tesseract         1.0.0.24037   c102097e6521  6 months ago   315 MB
localhost/demisto/docxpy            1.0.0.24033   0dd4cf15a8e3  6 months ago   118 MB
localhost/demisto/bs4               1.0.0.24033   0c21d9e40a6b  6 months ago   118 MB
localhost/demisto/teams             1.0.0.23674   d8b500a8d902  7 months ago   99.5 MB
localhost/demisto/unzip             1.0.0.23423   335cd9390db6  7 months ago   75.7 MB
localhost/demisto/tld               1.0.0.23423   3071f8766351  7 months ago   70.2 MB
localhost/demisto/pandas            1.0.0.23402   7bc281478a52  7 months ago   173 MB
localhost/demisto/ml                1.0.0.23334   8b1ab5e2c14a  7 months ago   2.12 GB
localhost/demisto/py3ews            1.0.0.23229   4837615bffee  8 months ago   108 MB
localhost/demisto/taxii             1.0.0.23209   d85b84a65c1b  8 months ago   97.9 MB
localhost/demisto/fetch-data        1.0.0.22177   93691001c6ae  8 months ago   791 MB
localhost/demisto/sane-pdf-reports  1.0.0.20952   928024659903  9 months ago   1.86 GB
localhost/demisto/sane-doc-reports  1.0.0.20692   4f80e7c460b0  9 months ago   1.24 GB
localhost/demisto/python3-deb       3.8.2.6981    bbb3182e09b1  24 months ago  224 MB
localhost/demisto/powershell        6.2.3.5563    85d095f4644b  2 years ago    175 MB
localhost/demisto/python            1.3-alpine    dccfc7e86281  3 years ago    104 MB
localhost/demisto/stix              latest        755ce7548e62  5 years ago    725 MB
```
- Verify that they are in the /home/demisto/.local/share/containers/storage/overlay
```
du -sh /home/demisto/.local/share/containers/storage/overlay
ll /home/demisto/.local/share/containers/storage/overlay
```

### Step 6: Log in to XSOAR and verify everything
- Run `/docker_images` in Playgroud
<img width="882" alt="image" src="https://user-images.githubusercontent.com/41276379/158926938-9808cae7-a272-4830-aa49-00e715fe60ea.png">
- Run `!py script="demisto.results('hello world')"`
<img width="461" alt="image" src="https://user-images.githubusercontent.com/41276379/158927014-57c5a3de-5e4c-40a7-8930-6dcd87cbf3dc.png">

### Note
This guide was written based on RHEL 8.5 and its default Podman
```
[root@localhost ~]# cat /etc/redhat-release 
Red Hat Enterprise Linux release 8.5 (Ootpa)
[root@localhost ~]# podman --version
podman version 3.3.1
```

