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



`sudo pvs`   # to check if physical volumes have been created and running successfully



