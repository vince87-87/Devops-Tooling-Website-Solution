# Devops Tooling Website Solution

# Setup up NFS Server

# Provision NFS Storage Server Using RHEL



# Provision 3 ebs volume and attached each volume to nfs server




# Create partition for all 3 volumes using gdisk






# Install LVM2 & Create 3 pv & volume group nfsdata



# create 3 logical volume-> opt , apps , logs verify lv created




# format physical volume to xfs filesystem


# create 3 directory used in web server , Jenkins & mount lv to the directory



# update fstab to persist mount after reboot


# test configuration & reload daemon


# Update repo & Install nfs

sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server
sudo systemctl enable nfs-server
sudo systemctl status nfs-server


# take note of the subnet id




# setup permission on the 3 mount directory to allow web servers to rwx 

sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server










# configure access for remote nfs mount, check port used by nfs

create /etc/exports
/mnt/apps 172.31.16.0/255.255.240.0(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.16.0/255.255.240.0(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.16.0/255.255.240.0(rw,sync,no_all_squash,no_root_squash)

Exportfs -arv # -a export all directories , -r reexport all directories , -v verbose



# Allow tcp/udp 2049 & 111 on security group attached to nfs server







# Setup & Configure Database

# provision ec2 install for database server using RHEL

# update repo & install mysql






# start & configure mysql
sudo mysql
CREATE DATABASE tooling;
CREATE USER `webaccess`@`172.31.16.0/255.255.240.0` IDENTIFIED BY 'WeBa44e$$@123';
GRANT ALL ON tooling.* TO 'webaccess'@'172.31.16.0/255.255.240.0';
FLUSH PRIVILEGES;
SHOW DATABASES;
Exit


# Setup Web Servers
# Provision Web Server with RHEL

# install nfs client
sudo yum install nfs-utils nfs4-acl-tools -y

# Create directory /var/www/ and target the NFS server’s export for apps
Sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid 172.31.26.62:/mnt/apps /var/www
df -h

# mount nfs on /etc/fstab to persist after reboot

# Install Apache & start apache service
sudo yum install httpd -y


# Provision Another 2 web servers 

# Do the same for another 2 web servers 
sudo yum install nfs-utils nfs4-acl-tools -y
Create directory /var/www/ and target the NFS server’s export for apps
mount nfs on /etc/fstab to persist after reboot
install apache & start apache service





# verify nfs is mounted on all 3 web server by create a test file on /var/www 



# mount /var/log/httpd to export nfs /mnt/logs to each web server

# update my function.php



# use tooling database

# update security group of database server to allow security group of web server to access & Apply tooling-db.sql script.
Sudo yum install mysql-server -y
# configure SELINUX policies
sudo chown -R apache:apache /var/www/html
#change file selinux security contest
sudo setsebool -P httpd_can_network_connect=1
sudo setsebool -P httpd_use_nfs 1
sudo mysql -h 172.31.29.162 -u webaccess -p tooling < tooling-db.sql


# Setup php
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo dnf module list php
sudo dnf module reset php
sudo dnf module enable php:remi-7.4
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
php -v
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo systemctl status php-fpm
setsebool -P httpd_execmem 1
sudo systemctl restart httpd

# browse to web browser
http://ec2-18-139-160-98.ap-southeast-1.compute.amazonaws.com/login.php


# verify you are able to login successfully





