# Devops Tooling Website Solution

# Setup up NFS Server

# Provision NFS Storage Server Using RHEL

![image](https://user-images.githubusercontent.com/49937302/118361600-c34dc680-b5be-11eb-86dc-d0898f0106c6.png)


# Provision 3 ebs volume and attached each volume to nfs server

![image](https://user-images.githubusercontent.com/49937302/118361608-c8127a80-b5be-11eb-95f8-2c5138c09bae.png)

![image](https://user-images.githubusercontent.com/49937302/118361615-cd6fc500-b5be-11eb-9667-cea28321e440.png)

# Create partition for all 3 volumes using gdisk

![image](https://user-images.githubusercontent.com/49937302/118361660-09a32580-b5bf-11eb-8a2e-d1761be6cc87.png)


# Install LVM2 & Create 3 pv & volume group nfsdata

![image](https://user-images.githubusercontent.com/49937302/118361665-0f007000-b5bf-11eb-86fe-463a42e22be7.png)


# create 3 logical volume-> opt , apps , logs verify lv created

![image](https://user-images.githubusercontent.com/49937302/118361673-132c8d80-b5bf-11eb-91e1-988fdd99ce72.png)


# format physical volume to xfs filesystem

![image](https://user-images.githubusercontent.com/49937302/118361677-19bb0500-b5bf-11eb-9355-cb0610b939cb.png)


# create 3 directory used in web server , Jenkins & mount lv to the directory

![image](https://user-images.githubusercontent.com/49937302/118361684-1de72280-b5bf-11eb-959c-4b0cda9214ad.png)


# update fstab to persist mount after reboot

![image](https://user-images.githubusercontent.com/49937302/118361692-23dd0380-b5bf-11eb-8fc6-36b1c0a8b432.png)


# test configuration & reload daemon

![image](https://user-images.githubusercontent.com/49937302/118361696-28092100-b5bf-11eb-8c5a-e3ac0a862823.png)

# Update repo & Install nfs

sudo yum -y update

sudo yum install nfs-utils -y

sudo systemctl start nfs-server

sudo systemctl enable nfs-server

sudo systemctl status nfs-server

![image](https://user-images.githubusercontent.com/49937302/118361702-2d666b80-b5bf-11eb-9d81-87d5cbcfc47f.png)


# take note of the subnet 

![image](https://user-images.githubusercontent.com/49937302/118361706-322b1f80-b5bf-11eb-9065-3c61e7895997.png)

![image](https://user-images.githubusercontent.com/49937302/118361707-35bea680-b5bf-11eb-8cc7-5dd220a5a750.png)



# setup permission on the 3 mount directory to allow web servers to rwx 

sudo chown -R nobody: /mnt/apps

sudo chown -R nobody: /mnt/logs

sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps

sudo chmod -R 777 /mnt/logs

sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server

![image](https://user-images.githubusercontent.com/49937302/118361714-3a835a80-b5bf-11eb-9262-b95f6185b650.png)









# configure access for remote nfs mount, check port used by nfs

vi /etc/exports

/mnt/apps 172.31.16.0/255.255.240.0(rw,sync,no_all_squash,no_root_squash)

/mnt/logs 172.31.16.0/255.255.240.0(rw,sync,no_all_squash,no_root_squash)

/mnt/opt 172.31.16.0/255.255.240.0(rw,sync,no_all_squash,no_root_squash)

# -a export all directories , -r reexport all directories , -v verbose

Exportfs -arv 

![image](https://user-images.githubusercontent.com/49937302/118361758-60a8fa80-b5bf-11eb-9abd-3bea97f27ad8.png)


# Allow tcp/udp 2049 & 111 on security group attached to nfs server



![image](https://user-images.githubusercontent.com/49937302/118361774-6ef71680-b5bf-11eb-9aba-5e13419d4d82.png)




# Setup & Configure Database

# provision ec2 install for database server using RHEL

![image](https://user-images.githubusercontent.com/49937302/118361781-73bbca80-b5bf-11eb-962c-ccc60a4b3756.png)

# update repo & install mysql

sudo yum update -y

sudo yum install mysql-server -y

sudo systemctl restart mysqld

![image](https://user-images.githubusercontent.com/49937302/118361799-87673100-b5bf-11eb-91ba-d36d027908f8.png)




# start & configure mysql
sudo mysql

CREATE DATABASE tooling;

CREATE USER 'webaccess'@'172.31.16.0/255.255.240.0' IDENTIFIED BY 'WeBa44e$$@123';

GRANT ALL ON tooling.* TO 'webaccess'@'172.31.16.0/255.255.240.0';

FLUSH PRIVILEGES;

SHOW DATABASES;

Exit


![image](https://user-images.githubusercontent.com/49937302/118361809-8d5d1200-b5bf-11eb-81e3-d25495f7936f.png)


# Setup Web Servers
# Provision Web Server with RHEL

![image](https://user-images.githubusercontent.com/49937302/118361819-95b54d00-b5bf-11eb-83eb-8bfe139d51ac.png)


# install nfs client
sudo yum install nfs-utils nfs4-acl-tools -y

![image](https://user-images.githubusercontent.com/49937302/118361826-9a7a0100-b5bf-11eb-866a-cecb2e9ca384.png)


# Create directory /var/www/ and target the NFS server’s export for apps
Sudo mkdir /var/www

sudo mount -t nfs -o rw,nosuid 172.31.26.62:/mnt/apps /var/www

df -h


![image](https://user-images.githubusercontent.com/49937302/118361833-9fd74b80-b5bf-11eb-9294-0bc704bc899d.png)


# mount nfs on /etc/fstab to persist after reboot

![image](https://user-images.githubusercontent.com/49937302/118361840-ab2a7700-b5bf-11eb-8fb7-bd5e86569802.png)

# Install Apache & start apache service

sudo yum install httpd -y


![image](https://user-images.githubusercontent.com/49937302/118361844-b2ea1b80-b5bf-11eb-8906-867c4bee910d.png)


# Provision Another 2 web servers 

# Do the same for another 2 web servers 

#Create directory /var/www/

Sudo mkdir /var/www

#mount the target to the NFS server’s export for apps

sudo mount -t nfs -o rw,nosuid 172.31.26.62:/mnt/apps /var/www

#Install nfs client
sudo yum install nfs-utils nfs4-acl-tools -y

#mount nfs on /etc/fstab to persist after reboot

sudo vi /etc/fstab

sudo mount -a

sudo systemctl daemon-reload

#add the following:

172.31.26.62:/mnt/apps  /var/www nfs defaults 0 0

#install apache & start apache service

sudo yum install httpd -y

sudo systemctl start httpd

sudo systemctl enable httpd

sudo systemctl status httpd

![image](https://user-images.githubusercontent.com/49937302/118361852-baa9c000-b5bf-11eb-880d-c99e4a0f6e86.png)

![image](https://user-images.githubusercontent.com/49937302/118361859-c1d0ce00-b5bf-11eb-8732-99f3961c671f.png)


# verify nfs is mounted on all 3 web server by create a test file on /var/www 

![image](https://user-images.githubusercontent.com/49937302/118396905-39683100-b684-11eb-8c0f-23b44d018226.png)

![image](https://user-images.githubusercontent.com/49937302/118396909-3e2ce500-b684-11eb-9c8b-03375e2e3482.png)


# mount /var/log/httpd to export nfs /mnt/logs to each web server

![image](https://user-images.githubusercontent.com/49937302/118361893-df9e3300-b5bf-11eb-81fe-6d429096aab2.png)

# fork the github to my github account

![image](https://user-images.githubusercontent.com/49937302/118398288-8222e880-b68a-11eb-8858-524a641cb751.png)

#install git

sudo yum install git -y

#clone the repo from my github

git clone https://github.com/vince87-87/tooling.git

#copy html to /var/www/html

cp -r /home/ec2-user/html/tooling/html/. /var/www/html

ls -la /var/www/html

![image](https://user-images.githubusercontent.com/49937302/118398575-d37fa780-b68b-11eb-8cce-966a1217bbac.png)


# update my function.php

sudo vi /var/www/html/function.php

#update the db connection 

$db = mysqli_connect('172.31.29.162', 'webaccess', 'WeBa44e$$@123', 'tooling');

![image](https://user-images.githubusercontent.com/49937302/118397009-abd91100-b684-11eb-8f0b-e3bb55454880.png)


# update security group of database server to allow security group of web server to access

![image](https://user-images.githubusercontent.com/49937302/118361914-ef1d7c00-b5bf-11eb-9137-ed236b93813e.png)

#  Install mysql on web server & apply tooling-db.sql script

sudo yum install mysql-server -y

cd /home/ec2-user/html/tooling

sudo mysql -h 172.31.29.162 -u webaccess -p tooling < tooling-db.sql

#Enter the password of the webaccess

![image](https://user-images.githubusercontent.com/49937302/118361931-f6448a00-b5bf-11eb-9c64-a298f4278b57.png)

# configure SELINUX policies
sudo chown -R apache:apache /var/www/html

#change file selinux security contest

#allow httpd/apache2 to make network connections.

sudo setsebool -P httpd_can_network_connect=1

#allow httpd to use nfs

sudo setsebool -P httpd_use_nfs 1

#allows Apache HTTP Server scripts and modules to connect to database servers.

sudo setsebool -P httpd_can_network_connect_db 1

![image](https://user-images.githubusercontent.com/49937302/118361924-f2186c80-b5bf-11eb-8709-7d7454b5a1b3.png)



# Setup php
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm -y

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm -y

sudo dnf module list php

sudo dnf module reset php

sudo dnf module enable php:remi-8.0 -y

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd -y

php -v

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo systemctl status php-fpm

setsebool -P httpd_execmem 1

sudo systemctl restart httpd


![image](https://user-images.githubusercontent.com/49937302/118361939-fba1d480-b5bf-11eb-82a4-04b3b881e652.png)


# browse to web browser
http://ec2-18-139-160-98.ap-southeast-1.compute.amazonaws.com/login.php

![image](https://user-images.githubusercontent.com/49937302/118361946-ffcdf200-b5bf-11eb-910f-7ce874353cb6.png)

# verify you are able to login successfully

![image](https://user-images.githubusercontent.com/49937302/118361948-03617900-b5c0-11eb-82fd-7960c07c64c4.png)




