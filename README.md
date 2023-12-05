# Project-10-Implementing-a-business-website-solution

# Implementing a business website using NFS for the backend File storage

In this project we will implement a solution that consist of the following:

1. Infrastructure : AWS
2. Webserver Linux : RedHat enterprise linux 8
3. Database server: Ubuntu 20.04 + MYSQL
4. Storage server: RedHat enterprise linux 8 + NFS server
5. Programming Language: PHP
6. Code repository: Github

We will be implementing the above 6 points as the the diagram below:

![image](https://github.com/DevopMudi/Project10-devops-tooling-website-solution/assets/149855241/3686c61b-ef41-4ab0-8732-849bf406be65)


On the above diagram, we can see a common pattern where several stateless web servers shares a commmon database and also access the same file using **Network file system NFS** as a shared file storage. 

Note: Even though the NFS server might be located on completely separate hardware. For webservers, its look like a local file system from where they can serve the same file.

Furthermore, its is important to know what storage solution is suituable for what use cases, for this we will need to answer the following questions: what data will be stored, in what format, how will this data be access, by whom, from where, how frequently and so on. Base on these we will be able to choose the right system for our solution.

## IMPLEMENTING A BUSINESS WEBSITE USING NFS FOR THE BACKEND FILE STORAGE

# STEP 1: Prepare NFS Server

 1. Spin up a new EC2 instance with RHEL Linux 8 Operating system.

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/b0bd286b-bfa4-402f-a3ec-5d3dd0475e3f)

 
 2. Configure LVM on the server:

    I. Create and Attached EBS volume to the NFS server, name of volumes are nfs vol1, nfs vol2 and nfs vol3

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/e23b1950-5a1a-4302-b277-906a0dcb7672)

   II. Ensure all 3 volume of 10gig created are attached to the NFS server:

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/411c022a-384e-410c-a604-2253f8644d8d)
  
  III. Repeat the above steps for nfs vol2 and nfs vol3.

3. Open your terminal and check configuration of your disk as per all three volume created that was attached to the NFS with command :**lsblk**

Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be nvme1n1, nvme2n1, nvme3n1. As display in image below.
As display below:

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/f6dacab5-51e0-4a21-b9b3-668a690e62f4)

Perform the following steps

i. Use df -h command to see all mounts and free space on the servers:

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/633b4736-68f0-45f6-9b66-9ba2855e954b)

ii. Use gdisk to create single partition on each of the three disk using the command:**sudo gdisk /dev/nvme1n1**

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/5a3ac7ab-5dee-42b6-af5f-14f91ec3f5c3)

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/a6a93911-9f64-4ff9-8324-b3f641d02d74)

iii. Repeat the above steps for nvme2n1 and nvme3n1 using the command : **sudo gdisk /dev/nvme2n1** and **sudo gdisk /dev/nvme3n1** respectively.

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/2dd04909-2679-4fef-b78f-6f02a22667ed)

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/7eef0398-6a70-4408-a18b-13b2c8a3fd16)

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/bd31fec5-295a-43fb-875a-98d7b4650dd0)

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/1095f080-d9ff-4281-a986-af5c2f8cf445)

iv. Use lsblk command to view the newly created partition on each of the three disk as display below

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/7d8ef1b3-edf5-40a9-8979-69c728da1c0d)

v. Install lvm2 package using **sudo yum install lvm2** then Run **sudo lvmdiskscan** command to check for available partitions.

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/5a500c70-ea3f-46f8-b283-9c8a7e48cc4f)

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/5e440a09-65e9-4cb9-b7e6-3c24201802a3)

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/54c0d684-43ae-484f-a136-faa0c499d02f)

vi. Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM using the following command: sudo pvcreate /dev/nvme1n1p1 sudo pvcreate /dev/nvme2n1p1 sudo pvcreate /dev/nvme3n1p1

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/6d590956-4ca2-4533-b0d1-8b4e077dacc5)


vii. Verify that your Physical volume has been created successfully by running : sudo pvs

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/22c1cd80-77f3-4c8f-af03-a7fd6120c451)

viii. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg using the following command:**sudo vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1**

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/12a0396a-dff8-4ba0-a9a7-bdad05a91b58)

ix. Verify that your VG has been created successfully by running:sudo vgs

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/800b6bc6-dd74-4ef9-9cfe-3ebd3a885a42)

x. Use lvcreate utility to create 3 logical volumes. name them apps-lv (give it 9G), logs-lv(9G) and opt-lv(9G). NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs. the opt-lv will be use in project11 .
We will do this by running the command below:

sudo lvcreate -n apps-lv -L 9G webdata-vg

sudo lvcreate -n logs-lv -L 9G webdata-vg

Sudo lvcreate -n opt-lv  -L 9G webdata-vg

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/5cdbeb60-110e-4304-b035-e4517f87ef6f)


xi.Use the sudo lvs command to check the logical volumes created, image will be as below

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/e3289384-4678-4bcb-8ebf-5444b4fa434a)

xii. Verify the entire setup with the following command:sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk image will be as display below:

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/a8c1625f-2cf9-4b59-b738-ba1cf89b32fc)

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/8098e491-3274-4c7f-b01c-976ba78ccd15)

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/bd5d3d9c-4508-41d3-867c-8fef6d3a90de)




 4. Format the disk as xfs : that is we are going to use mkfs.xfs to format the logical volumes with xfs filesystem using the below command:

sudo mkfs -t xfs /dev/webdata-vg/apps-lv 

sudo mkfs -t xfs /dev/webdata-vg/logs-lv

sudo mkfs -t xfs /dev/webdata-vg/logs-lv

sudo mkfs -t xfs /dev/webdata-vg/opt-lv

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/3fc8eb03-fa64-405d-a62b-a7a51cb3ad40)


 5. Ensure there are 3 logical volume lv-opt, lv-apps, and lv-logs by running the command: sudo lvs

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/3ec120ca-5f7e-4ae0-910d-bb4c24828bf3)



 6. Create mount point on /mnt directly for the logical volumes as follows: lv apps on /mnt/apps,-to be used by the webservers, also mount lv-logs on /mnt/logs to be use by webservers logs and finally mount **Mount lv -opt on /mnt/opt to be use by jenkins in our next project.**

We will perform this mounting after creating directory or folder for them as mount point and then we use the following below command to mount for each.

sudo mount /dev/webdata-vg/apps-lv /mnt/apps/

sudo mount /dev/webdata-vg/logs-lv /mnt/logs/

sudo mount /dev/webdata-vg/opt-lv /mnt/opt/

Image will be as display below:

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/b3c0a65b-be1f-4bb3-9446-0423d2cfa034)

We may decide to check if all are mounted completely by running: lsblk command, image will be as below:

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/16e81ac4-c6c2-4e10-8840-1fab785c71b2)


7. Install NFS server, configure it to start on reboot and make sure it is running: we will use the below command:

sudo yum -y update

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/c0054d36-8e32-425d-9900-0a008f56eb3b)

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/b6878d61-8c04-4106-bcaf-4fd01dcdcdd5)

sudo yum install nfs-utils -y

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/f2c50ee3-033f-43a1-9002-95b02588ecb5)

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/2759a990-0acb-4b0f-b021-80c8307333cc)


sudo systemctl start nfs-server.service

sudo systemctl enable nfs-server.service

sudo systemctl status nfs-server.service

Image for the above three command will as below:

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/c77e4c4b-d8d2-4f1b-b3d6-da99c3d3503e)

 
8. Export the mount for the webservers subnet CIDR to connect as client. For simpicity we will install all our three webservers inside the same subnet however in production set we may probably want to separate each tier inside its own subnet for higher level of security. To check our subnet CIDR. Open our EC2 detail in AWS web console and locate **Networking tab** and then open a subnet link as below image display:

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/e2990175-1f37-420a-969a-b6fc159ae8c9)

The subnet CIDR is as pointed by the IPV4 CIDR in the below figure **(172.31.16.0/20)**


![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/399b439f-66f4-4c42-a861-8a44409231b5)

9. Lets set permission that will allow our webservers to read, write and execute files on our NFS:

   We will do this by running the below command:

sudo chown -R nobody: /mnt/apps

sudo chown -R nobody: /mnt/logs

sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps

sudo chmod -R 777 /mnt/logs

sudo chmod -R 777 /mnt/opt

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/e1c6c3b9-f7c8-4bd6-9582-987058d9a557)


10. We will now configure access to NFS for clients within the same subnet using our subsnet CIDR from step 8 above (172.31.16.0/20)

  We will do this by running the below code:

sudo vi /etc/exports
 
 We then copy and paste the below in the exports file
 
/mnt/apps 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.16.0/20(rw,sync,no_all_squash,no_root_squash)

Image display should be as below after running the sudo vi /etc/exports command:


![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/2322faf1-eba4-42d3-a9c0-ca791590f188)


We then press Esc + :wq! and enter to save the file.

finally we run the below command to exportfs.

sudo exportfs -arv

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/34d54ae3-daf0-4283-aeea-642aa047b209)


11. Lets check which port is used by NFS and open it using security group on our AWS console add new inbound rules:

To check we run the following command : **rpcinfo -p | grep nfs**

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/e62acac0-113f-414c-99fe-671c4ec7df09)

**Important note: In order for NFS server  to be accessible from our client, we must also open the following ports: TCP 111,UDP 111 UDP 2049,**

Image will be as below after setting the inbound rules

![image](https://github.com/DevopMudi/Project10s-devops-tooling-website-solution/assets/149855241/2235e570-48a1-440a-ad0b-d1524a8b5de5)



## CONFIGURE BACKEND DATABASE(DB) AS PART OF THE 3 TIER ARCHITECTURE


The following steps are involve here:

1. Update and install mysql server

![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/7a7db0c1-d96e-4ae4-bd9e-af48a703b803)

![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/e901e45a-f255-4605-a223-78a099596d7d)

2. Create a database and name it **tooling**

![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/6f67c957-1611-406b-851e-660c4c7f8e4c)


![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/9fb5b8fc-132a-4fcd-b59b-d4034c14ca3d)

3. Create a database user and name it **webaccess**
  
   ![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/9fb5b8fc-132a-4fcd-b59b-d4034c14ca3d)


4.Grant permission to webaccess user on tooling database to do anything from the webserver subnet CIDR


![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/9fb5b8fc-132a-4fcd-b59b-d4034c14ca3d)



## PREPARE THE WEBSERVER

We need to make sure our webservers can server  the same content from a share storage device, in this case (the NFS)

NOte : For storing shared file that our webservers will used, we will utilise nfs and mount previously created logical volumes lv -apps to the folder where Apache stores files to be serve to the users. (/var/www)
This approach will make our webservers **stateless** which means we will be able to add new ones or remove them whenever we need and the integrity of the data ( the database data and NFS data) will be preserved.

In this section, we will carry out the following:

- Configure NFS client ( this will be done on all three servers)

- Deploy tooling application to our webservers into a shared NFS folder

- Configure the webservers to work with a single mysql database.

  The following steps are involve.

  1. Launch 3 new EC2 instance with RHEL 9 Operating system

![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/34606fbb-e921-4576-9aa9-dbf5db948047)

  2. Install NFS using the following command: **sudo yum install nfs-utils nfs4-acl-tools -y**

![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/c23ccb69-a328-4d74-9dce-75678c9fbb89)

![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/d7528ad4-9a16-4c10-b40b-4b6b763838bf)

  3. Mount ?var/www and target the NFS server exports for apps using the following command:
  
  . **sudo mkdir /var/www**
    
  **sudo mount -t nfs -o rw,nosuid 172.31.20.217 :/mnt/apps /var/www**

  
  ![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/8ea26258-7320-4d90-8d14-1f1b10571144)

  
  4. Verify that NFS was mounted successfully by running **df -h** also make sure the changes wil persist in the webserver after the reboot by running:

     sudo vi /etc/fstab.

     and pasting the following then save with esc + :wq! + enter

     172.31.20.217 :/mnt/apps /var/www nfs defaults 0 0

![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/6c4e409e-d7ad-4065-80c2-4f508d9bc608)

![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/987099fa-3958-4337-9e3d-a4944339a01d)


  5. Install Remi repository Apache and PHP using the following code:

      sudo yum install httpd -y

     ![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/cc310e3e-6f9d-4e49-bf30-e8c3eab26e0d)


      sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

     ![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/46bfe497-5293-45b7-b834-07d1255408b2)
   
      sudo dnf module reset php

     ![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/565a1d89-3b1c-4870-8a69-8a570fa0895e)

     sudo dnf module enable php

     ![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/7e82177c-e9a3-4b72-9508-80927906265e)


      sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

 ![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/4efbd1c3-f34d-469a-9c96-4059203ff7c8)

    sudo systemctl start php-fpm
    sudo systemctl enable php-fpm
    setsebool -P httpd_execmem 1


 ![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/83f1cb87-2eaf-45ef-82b3-899e7847f977)


    Note: step 1 to 5 are performed on the webserver3 simultaneously.


   6.  Verify that attached files are available in the webservers (/var/www) and also on the NFS server at /mnt/apps. If a file  create in the webserver is available on 
      the NFS server, its means our NFS servers are mounted correctly

    
![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/b009bd2d-1c9b-400e-ae69-95b24298c9bd)


   7. We will now locate the log folder for Apache on the webserver and mount it to NFS server for logs and we will repeat step 4 above to make sure the mount point will be persistent after reboot.

![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/67b3535b-6475-4f23-bc10-2ca113f60337)

    
![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/7e3b18dc-682c-457c-bbe0-7cee28449798)

         
  8. We will fork the tooling source code from Dare.io Github Account into our git hub accout., then clone it by running the following command :

      sudo yum install git

      git init

      git clone https://github.com/darey-io/tooling.git


![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/1aab5e6d-c3e5-4c64-a175-4362c1b53f5f)

    
 
 ![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/c0ceba57-899e-4c1e-b2f1-dc443817447f)

    
      

   10. We will deploy the tooling website of the code to our webserver and ensure that html folder from the repository is deploy to /var/www/html by coping using the commmand **sudo cp -R html/. /var/www/html**

   11.Run the SElinux command : **sudo setenforce 0**  and

  * sudo vi /etc/sysconfig/selinux*

    Then change enforce selinux , to disabled as in below image

![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/bb3f5687-ecd8-4ed9-a9ea-67eb04fc14a5)

    
   11. Use the website configuration to connect to the database in (/var/www/html/functions.php file) then apply **tooling-db.sql** script to the database using the command:**mysql -h 172.31.24.34 -u webaccess -p password < tooling-db.sql**

![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/10b8a71c-6f94-4a09-8b76-6faf2f1d9771)


![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/719a0d94-02cf-4d83-8ec9-f32dea4babe6)



   12. Create in mysql a new admin user with username **myuser** and password **password**

       INSERT INTO users ('id', 'username', 'password', 'email', 'user_type', 'status')
       VALUES (2, 'myuser', 'password', 'user@mail.com', 'admin', '2');



   14. Open the website with any of webserver public IP and loging

       ![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/178ffdd7-d8b6-49b1-a0f5-1bb28e91cf68)



  ![image](https://github.com/DevopMudi/Project-10-Devop-tooling-website-solution/assets/149855241/b61828fc-f7fb-4617-a28d-affebfa255dc)




























































































