# DEVOPS TOOLING WEBSITE SOLUTION

### Project Objective 
**In this project I will implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible.**

In this project I will implement a solution that consists of following components:

1. Infrastructure: AWS
2. Webserver Linux: Red Hat Enterprise Linux 8
3. Database Server: Ubuntu 20.04+ MySQL
4. Storage Server: Red Hat Enterprise Linux 8 + NFS Server
5. Programming Language: PHP
6. Code Repository: GitHub

![3 tier web application](<img/00 tier web application.jpg>)

### Implementing a business website using NFS for the backend file storage

#### Step 1 Spinning up EC2 instances

I lunched 5 ec2 instance as stated below and the respective platforms

* one DB server on Ubuntu OS
* one NFS server on on Redhat OS
* Three Webservers on Redhat os


![launched 5 instances](<img/1 create 5 ec2 instance four redhat and one ubuntu.png>)


# NOTE - For the begining stage of this project i will make use of the knowledge acquired from my previous project ***[here](https://github.com/brightfav/9.IMPLEMENTING-WORDPRESS-WEBSITE-WITH-LVM-STORAGE)*** 

next i created three volume and attached it to the NFS server using amazon portal.

![create and attach three volume](<img/2 created three volume and attached it to NFS server.png>)

next i SSh into my into my NFS

![ssh into my nfs server](<img/3 ssh into nfs server.png>)


```
lsblk 
```

![attached volumes](<img/4 attached volumes.png>)


i typed the following after typing the command below 
`n`, `1` `8300` `p` `w` `y` for 

`8300` will change the partition type to `Linux file system`

```
sudo gdisk /dev/xvdf
```

![gdisk xvdf](<img/5 gdisk xvdf.png>)


```
sudo gdisk /dev/xvdg
```

![gdisk xvdg](<img/6 gdisk xvdg.png>)


```
sudo gdisk /dev/xvdh
```

![gdisk xvdh](<img/7 gdisk xvdh.png>)


```
lsblk
```


![lsblk to see new partitions](<img/8 new partitions created.png>)


```
sudo yum install lvm2 -y
```

![install lvm2](<img/9 sudo yum install lvm2 -y.png>)


confirm *lvm* installation


```
which lvm
```

![lvm installed successfully](<img/10 lvm installed successfully.png>)


```
sudo lvmdiskscan
```

![available partition](<img/11 available partition.png>)


```
sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1  
```

![volume created](<img/12 physical volume created.png>)


```
sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1  
```

![volume group created](<img/13 volume group successfully created.png>)


```
sudo vgs
```

![volume group created](<img/14 confirm the creation of volume groups.png>)


```
sudo lvcreate -n lv-apps -L 9G webdata-vg
```

```
sudo lvcreate -n lv-logs -L 9G webdata-vg
```

```
sudo lvcreate -n lv-opt -L 9G webdata-vg
```

![logs apps opt created](<img/15 logical volume created apps logs opt.png>)


```
sudo lvs
```

![lvs created](<img/16 view created lvs.png>)


```
lsblk
```

![lsblk](<img/17 lvs viewd.png>)


```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
```

![display full setup](<img/18 view entire setup.png>)

after the step above i formatted the disks as `xfs` and not `ext4` 

```
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
```

![lv-apps](<img/19 make lv-apps an xfs file system.png>)


```
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
```
<br />

```
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
```

![lv logs lv opt](<img/20 make lv-logs and lv-apt an xfs file system.png>)

next i created a mount point on `/mnt` directory for  disks `apps`, `logs`, `opt`


using the commands below

```
sudo mkdir /mnt/apps
```

```
sudo mkdir /mnt/logs
```

```
sudo mkdir /mnt/opt
```

![mount point created](<img/21 create mount point  for apps logs and opt.png>)

next i mounted 

`lv-apps` on `/mnt/apps` 

`lv-logs` on `/mnt/logs`

`lv-opt` on `/mnt/opt`

using the commands below

```
sudo mount /dev/webdata-vg/lv-apps /mnt/apps
```

```
sudo mount /dev/webdata-vg/lv-logs /mnt/logs
```

```
sudo mount /dev/webdata-vg/lv-opt /mnt/opt
```

![mount apps logs and opt](<img/22 mount apps logs and opt to their respective mount point.png>)


next i install NFS utility on the `NFS Ec2 instance`  to start on reboot and make sure it is up and running using the commands below

```
sudo yum -y update
```

![update repo](<img/23 update repository.png>)


```
sudo yum install nfs-utils -y
```

![install nfs](<img/24 sudo yum install nfs utilis.png>)


```
sudo systemctl start nfs-server.service
```

```
sudo systemctl enable nfs-server.service
```

```
sudo systemctl status nfs-server.service
```

![enable nfs server](<img/25 start enable status of nfs server.png>)

Next i obtained the subnet of my webservers. To do so i followed the pictorial steps below

![subnet obtain](<img/000 select subnet id to input in database creation.png>)

i copied the subnet of my webservers in this case it is

`172.31.16.0/20`


![subnet obtain 2](<img/001 subnet id copied and pasted into database creation.png>)


next i typed the command to set up permission that will allow my webservers to read, write and execute files on NFS

```
sudo chown -R nobody: /mnt/apps
```

```
sudo chown -R nobody: /mnt/logs
```

```
sudo chown -R nobody: /mnt/opt
```

```
sudo chmod -R 777 /mnt/apps
```

```
sudo chmod -R 777 /mnt/logs
```

```
sudo chmod -R 777 /mnt/opt
```

```
sudo systemctl restart nfs-server.service
```

```
sudo systemctl status nfs-server.service
```

![change permission](<img/26 chown and chmod command.png>)


next i configured access to NFS for clients within the same subnet using the code below

```
sudo vi /etc/exports
```

![sudo vi export](<img/27 sudo vi etc export.png>)

```
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
```

![subnet output](<img/28 type code and change subnet sidr later.png>)


on the vi file i changed `<Subnet-CIDR>` to `172.31.16.0/20`

![changed subnet](<img/29 changed subned sider in vi file.png>)


next i exported all the mount points to webserver using the code below


```
sudo exportfs -arv
```

![export mount points](<img/30 export subntcdr.png>)



next i check which port is used by `NFS` server using the command below

```
rpcinfo -p | grep nfs
```

![ports used by nfs](<img/31 check port used by nfs.png>)


next i open it using security groups (i edited the inbound rules)

**Note:- i opened the following ports**

`TCP 111`

`UDP 111`

`UDP 2049`

`TCP 2049`

**Note :- 2049 is the port of NFS obtained after using the command above**

![inboud rule](<img/32 add inbound rule of nfs server.png>)


### Configure Backend Database as Part of Three Tier Architecture

first i ssh into my Database Server

Next i updated the database server repository using the command below

```
sudo apt update
```

![update ubuntu](<img/33 sudo apt update.png>)


Next i installed `mysql-server` using the command below

```
sudo apt install mysql-server
```

![install mysql server](<img/34 install mysql server.png>)


Next i opened `mysql`

```
sudo mysql
```

afterwards i created a database and named it `tooling`

```
create database tooling;
```

next i create a database user and named it `webaccess` with password as `password`

```
CREATE USER 'webaccess'@'172.31.16.0/20' IDENTIFIED BY 'password';
```

next i granted permission to `webaccess` user on `tooling` database to do anything only from the webservers `subnet cidr` which is `172.31.16.0/20`

```
grant all priviliges on tooling.* to 'webaccess'@'172.31.16.0/20';
```

next i flushed privileges using the command below

```
flush privileges
```

![create database and grant permissions](<img/35 create database user password grant access and priviledges.png>)


### Prepare The Webservers

i need to make sure that my webservers can serve the same content form shared storage solutions in this scenario `NFS Server` and `MYSQL Database`.

#### in the next steps i will do the following

1. Configure NFS client (for all three webservers)

2. Deploy a Toling application to our Web Servers into a shared NFS folder

3. Configure the Web Servers to work with a single MySQL database


#### Lets begin

first i launch my Webserver Ec2 instance and SSH into it 

![ssh into webserver](<img/36 ssh into webserver 1.png>)

next i install `NFS client` using the command below

```
sudo yum install nfs-utils nfs4-acl-tools -y
```

![install nfs client](<img/37 install nfs utility on webserver 1.png>)


next i make `/var/www` directory using the code below

```
sudo mkdir /var/www
```

![make www directory](<img/38 make mount directory.png>)

next i mount `/var/www` using the command below

**note `<NFS-server-private-IP-Address>` is  `172.31.26.64`**

```
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
```
afterwards i verified the mount was successful using the command below 

```
df -h
```

![mount Nfs](<img/39 mount unto nfs server and use df -h to check if mount is successful.png>)


To make sure the changes persist after a reboot i added a line to the `/etc/fstab` using the command below to first open the file

```
sudo vi /etc/fstab
```

next i typed the line below into the file

```
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
```

![line of code](<img/41 insert this link of code.png>)

![added private address of nfs](<img/42 insert nfs private ip address.png>)


afterwards i reloaded the daemon for changes to take effect using the command below

```
sudo systemctl daemon-reload
```

![daemon reload](<img/43 daemon reload.png>)


**Next i installed Remi's Repository using the commands below**

```
sudo yum install httpd -y
```

![web install 1](<img/44 webserver install 1.png>)



```
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

![web install 2](<img/45 webserver install 1.png>)


```
sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

![webserver install 3](<img/46 webserver install 1.png>)


```
sudo dnf module reset php
```

```
sudo dnf module enable php:remi-7.4
```

![webserver install 4](<img/47 webserver install 1.png>)


```
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
```

![web install 5](<img/48 webserver install 1.png>)


```
sudo systemctl start php-fpm
```

```
sudo systemctl enable php-fpm
```

```
sudo setsebool -P httpd_execmem 1
```

![web install 5](<img/49 webserver install 1.png>)



next i verified that apache files and directories are available on the webserver in `/var/www` using the command below

```
ls /var/www
```

![apache files on webserver confirmed](<img/50 verify apache files are available on the webserver.png>)


i also confirmed that apache files and directories are available on the NFS server in `/mnt/apps` using the command below

```
ls /mnt/apps
```

![also files on nfs](<img/51 files is also on the nfs server.png>)


next i created a file `test.txt` on my webserver to verify if it will appear on NFS server using the command below

```
sudo touch /var/www/test.txt
```

![create test file](<img/52 created file in webserver 1.png>)


next i confirmed if `test.txt` is visible on nmy NFS server using the command below

```
ls /mnt/apps
```

![files is visible](<img/53 files seen in nfs server.png>)


next i located the log folder for apache on the webserver 

```
ls /var/log/httpd
```

the NFS server export for logs is found at 

```
ls /mnt/logs
```


next i mounted the logs folder for webserver onto the NFS Servers export for logs using the command below

```
sudo mount -t nfs -o rw,nosuid 172.31.26.64:/mnt/logs /var/log/httpd
```

![mount log webserver to nfs](<img/54 mount to log on nfs server.png>)


To be able to use Git i had it installed on my webserver using the command below

```
sudo yum install git
```

![install git ](<img/55 install git on webserver.png>)


next i initialized git using the command below

```
git init
```

next i renamed the branch to `main` using the command below

```
git branch -m main
```

![initialize git and rename it](<img/56 initialize git repository and next rename it to main.png>)


next i have to clone git repository but first i have to get the git repository link as shown in the image below

![copy git link](<img/58 copy darey repository url that i want to clone.png>)

next i clone the the remote `git` repository using the command below

```
git clone <git-repository-https-address>
```

![git clone repository](<img/57 clone git repository from darey.png>)


afterward i use the `ls` command to list and `cd` change directory into `tooling` directory and `ls` to display the components of the `tooling` directory

![list file and directories](<img/59 list files and directories change directory into tooling list files.png>)

******************************

![copy content](<img/60 list var-www copy content of html directory in tooling into var-www-html list content of var-www-html.png>)


next i allowed port `80` `http` on inbound rule for webserver

![add port 80](<img/61 add port 80 to webserver.png>)


next i opened the public address of the webserver but got an error as seen below

![error webpage](<img/62 error encountered.png>)

#### Troubleshoot 1

i checked apache status using the command below

```
sudo systemctl status httpd
```

![apache is dead](<img/63 apache is dead not running.png>)
**apache is dead**


#### to solve this 

i typed the command below

```
sudo setenforce 0
```

![setenforce 0](<img/66 sudo setenforce 0.png>)


next i opened `selinux` using the command below and 

```
sudo vi /etc/sysconfig/selinux
```

![edit selinux](<img/64 edit selinux file.png>)



afterward i set `SELINUX=disabled`


![change selinux to disabled](<img/65 change selinux to disabled.png>)


next i start httpd and checked the status to confirm its running using the command below

```
sudo systemctl start httpd
```

```
sudo systemctl status httpd
```

![httpd now working](<img/67 httpd now working.png>)


webserver public address is now working

![ipaddress now displaying pages](<img/68 ipaddress displaying pages.png>)


Next i updated the website configuration to connect to the database in `/var/www/html/functions.php` file using the command below

```
sudo vi /var/www/html/functions.php
```

![open vi file](<img/69 functions php.png>)

`172.31.26.86` = Database Private IP address

`webaccess` = Database username

`password` = Database password


![private address and ip added](<img/70 db private ip address username and password.png>)


next i installed mysql on my webserver using the command below

```
sudo yum install mysql -y
```

![install mysql in webserver](<img/71 install mysql in webserver.png>)


next i allowed inbound rule from on my `database server` from the security groups

`type` = MYSQL/Aurora

`protocol` = TCP

`port range` = 3306

`source` = webserver subnet cidr

![Alt text](<img/72 configured inbound to allow traffic from webserver subnet.png>)

afterward i checked the status of mysql to ensure it is running using the command below

```
sudo systemctl status mysql
```

next i configured the bind address of my db server using the command below

```
sudo vi /etc/mysql/mysql.conf.d/mysql.cnf
```

![check status and bind address](<img/73 didnt work checked mysql status and check conf file.png>)

next i set bind address to on my db server

`bind-address          = 0.0.0.0`

`mysqlx-bind-address   = 0.0.0.0`


![change bind address](<img/74 change bind address to 0 0 0 0.png>)


next i restart and checked the status of mysql on my dbserver


![restart and check mysql status](<img/75 restart and check status.png>)


#### on my webserver i checked if i can access mysql db server


i change directory to tooling 

```
cd tooling
```

`172.31.26.86 = database server private address`

`webaccess = database username `

a password will be required


after typing the command below no error should be displayed if any error is displayed there is need for trouble shooting

```
sudo mysql -h 172.31.26.86 -u webaccess -p tooling < tooling-db.sql
```

![error and correction](<img/76 errors and final process.png>)



```
sudo vi tooling-db.sql
```

![sudo vi tooling](<img/77 sudo vi tooling-db sql.png>)


![content of vi file](<img/78 vi file.png>)


to display the final result i typed any of the following command


```
http://<public-IP-of-webserver-or-public-DNS-name>
```

![ipaddress display page](<img/68 ipaddress displaying pages.png>)

next i typed the log in details to have access

`username = admin`

`password = admin`

![logged in](<img/79 final webpage.png>)


```
http://<public-IP-of-webserver-or-public-DNS-name>/index.php
```

![final webpage](<img/80 final webpage 2.png>)

