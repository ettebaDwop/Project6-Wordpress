# Web Solution With WordPress
In this project we will try to run 
This project is made up of two parts:

1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give you practical experience of working with disks, partitions and volumes in Linux.

2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify your skills of deploying Web and DB tiers of Web solution.
   
### Set up
The 3-Tier Setup

1. A Laptop or PC to serve as a client

2. An EC2 Linux Server as a web server (WordPress installed here)

3.  An EC2 Linux server as a database (DB) server


### Implementation

###  Part 1 — Prepare a Web Server
Open your PC browser and login to https://aws.amazon.com/ 
 
•  From the EC2 dashboard, Launch instance to start using a virtual server. 

•	Create 2 EC2 instances

•	Create 3 volumes and mount these on the Web Server EC2 instance  

![Screenshot (366)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/95df2376-78fc-4976-85a0-a462435511eb)

#### Step 2 - Connect on PowerShell
•	Copy  SSH key from AWS EC2 instance

•	Run the following commands:

`lsblk` 

![Screenshot (284)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/782c2810-65a8-42fd-92af-449342726248)

`df -h`    # to check for free space and see all mounts on server


`sudo gdisk /dev/xvdf` 

`sudo gdisk /dev/xvdg`

`sudo gdisk /dev/xvdh`     # to create a single partition on each of the 3 disks

To check partition created run    `lsblk`

![Screenshot (289)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/298682e8-dea2-42a5-b9a2-b9d3580a5207)

The next step is to install the "lwm2" package and check for available partitions.

`sudo yum install lvm2`  &&  `sudo lvmdiskscan`

![Screenshot (291)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/e9e96965-8c36-4887-a64e-1379a2e79975)

Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```

![Screenshot (293)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/03675165-59b9-4a78-80f8-932d8e679f54)

`sudo pvs`   # to check if physical volumes have been created and running successfully

![Screenshot (295)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/a4206ccd-2669-44d0-9f64-de407e5c1802)

Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![Screenshot (303)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/0c68e40d-8d5f-4fe4-96b4-0f2037e9c8f0)

*Create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs

```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
![Screenshot (305)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/6ad90b76-3e76-4c77-b7c1-63eb6d5a6867)

Verify that your Logical Volume has been created successfully by running: 

`sudo lvs`

![Screenshot (306)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/12f4b7f1-daae-494f-abc0-fff6fccd5ac3)

Verify the enture setup
```
sudo vgdisplay -v  #view complete setup - VG, PV, and LV
sudo lsblk
```
![Screenshot (307)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/0e40963f-cef4-4558-bfed-0307ed48bd5f)

Use mkfs.ext4 to format the logical volumes with ext4 filesystem

```
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```
![Screenshot (302)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/7fd3b1e5-8e7c-41d9-b5ac-25bd2b0d990c)

Create /var/www/html directory to store website files

`sudo mkdir -p /var/www/html`

Create /home/recovery/logs to store backup of log data

`sudo mkdir -p /home/recovery/logs`

Mount /var/www/html on apps-lv logical volume

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)

`sudo rsync -av /var/log/. /home/recovery/logs/`

![Screenshot (304)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/80ed5960-a6e2-4158-a2dd-4170146e35c3)

Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)

`sudo mount /dev/webdata-vg/logs-lv /var/log`

Restore log files back into /var/log directory

`sudo rsync -av /home/recovery/logs/. /var/log`

Next thing to do is to update /etc/fstab file so that the mount configuration will persist after restart of the server.

#### Update  /etc/fstab file
To update the file, run the following commands:
```
sudo blkid
sudo vi /etc/fstab
```
![Screenshot (310)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/93a55c4b-e758-43ae-90e7-864df7f32f19)

Open the *fstab* file and make the following changes:

![Screenshot (311)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/fde6bf52-e903-43dd-9ab1-8a02d8668856)


Test the configuration, reload the daemon and verify the setup and by running the command:

```
    sudo mount -a
    sudo systemctl daemon-reload
    df -h
```

![Screenshot (313)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/bfa79c1c-3866-4e0c-8c6c-b1f3e6cd487e)

### Part 2 - Prepare the Database Server
Run codes as before in Part 1, Step 2 . This time create a /db file


### Part 3 Install Word Press on the Webserver EC2 instance

* Update the repository

`sudo yum -y update`

* Install wget, Apache and it’s dependencies

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

* Start Apache

```
sudo systemctl enable httpd
sudo systemctl start httpd
```

### Install PHP and it’s depemdencies

```
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
```

* There was an error here due to the existence of multiple/conflicting php  package versions. Thus, the application failed here. To correct this I ran the following commands:

```
sudo yum remove php-8.0.27-1.el9_1.x86_64 php-opcache-8.0.27-1.el9_1.x86_64 php-common-8.0.27-1.el9_1.x86_64 php-mysqlnd-8.0.27-1.el9_1.x86_64
sudo yum install php php-opcache php-gd php-curl php-mysqlnd --allowerasing
```
* Restart and enbled php

```
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1
```

### Restart Apache
  
  `sudo systemctl restart httpd`

### Download wordpress and copy wordpress to var/www/html

```
  mkdir wordpress
  cd   wordpress
  sudo wget http://wordpress.org/latest.tar.gz
  sudo tar xzvf latest.tar.gz
  sudo rm -rf latest.tar.gz
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  cp -R wordpress /var/www/html/
  ```
### Configure SELinux Policies

```
  sudo chown -R apache:apache /var/www/html/wordpress
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  sudo setsebool -P httpd_can_network_connect=1
```

  
* Install mysql -server in var/www/html

`sudo yum install mysql-server`

* Start MYQSL server on the webserver as well.

```
sudo systemctl start mysqld
sudo systemctl enable mysqld
```
### Configure DB to work with WordPress
 
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER 'wordpress'@'%' IDENTIFIED WITH mysql_native_password BY 'mypass';
GRANT ALL PRIVILEGES ON *.* TO 'wordpress'@'%' WITH GRANT OPTION;`
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

* Check to see the exsisting users

`mysql> select user, host from mysql.user`

![Screenshot (373)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/0d42dd43-2417-477a-8c9b-575c722072a0)

### Configure WordPress to connect to remote database.
On the Wordpress server, edit the configuration file:

`sudo vi wp-config.php`

On the Data base server, edit the configuration file by running:

`sudo vi /etc/my.cnf`

After editing the configuration file, restart mysql server:

`sudo systemctl restart mysqld`

`sudo vi wp-config.php`
`sudo systemctl restart httpd`


Disable default page of Apache, run:

`sudo mv /etc/httpd/conf.d/welcome.conf  /etc/httpd/conf.d/welcome.conf_backup` #Creating a backup copy instead of disabling the page.


Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, allow access to the DB server ONLY from the Web Server’s IP address, so in the Inbound Rule configuration specify source as /32




Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

`sudo yum install mysql`

`sudo mysql -u admin -p -h <DB-Server-Private-IP-address>`

Verify if you can successfully execute `SHOW DATABASES;`  command and see a list of existing databases.

* Change permissions and configuration so Apache could use WordPress:

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP)

Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/
![Screenshot (370)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/c298d0af-637e-4e0c-82e3-8c1d4723c480)

Fill out your DB credentials:

![Screenshot (371)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/d7f92783-6650-49f1-abe6-7fb9c5035d33)




