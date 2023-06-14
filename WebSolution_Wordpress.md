 # Web Solution With WordPress

### Prerequisites

### Implementation

####  Step 1 — Prepare a Web Server
Open your PC browser and login to https://aws.amazon.com/ 
 
• A region is selected by default (change if necessary), from the search bar type EC2 and 
click. 
 
• From the EC2 dashboard, click on the button “Launch instance” to start using a virtual 
server. 
•	Create 2 EC2 instances
•	Create 3 volumes and mount these on the Web Server EC2 instance  


#### Step 2 - Connect on PowerShell
•	Copy  SSH key from AWS EC2 instance
•	Run the following commands:

` Lsblk` 

![Screenshot (284)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/782c2810-65a8-42fd-92af-449342726248)

`df -h`    # to check for  see free space and see all mounts on server


`sudo gdisk /dev/xvdf` 

`sudo gdisk /dev/xvdg`

`sudo gdisk /dev/xvdh`     # to create a single partition on each of the 3 disks

To check partition created run    `lsblk`

![Screenshot (289)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/298682e8-dea2-42a5-b9a2-b9d3580a5207)

The next step is to install the "lwm2" package and check for available partitions.

`sudo yum install lvm2`  &&  `sudo lvmdiskscan`

Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
![Screenshot (293)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/03675165-59b9-4a78-80f8-932d8e679f54)

`sudo pvs`   # to check if physical volumes have been created and running successfully



![Screenshot (291)](https://github.com/ettebaDwop/Project6-Wordpress/assets/7973831/e9e96965-8c36-4887-a64e-1379a2e79975)




