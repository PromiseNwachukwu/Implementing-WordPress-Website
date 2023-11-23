# Web Solution With WordPress
## Implementing LVM on Linux servers (Web and Database servers)
## Step 1 — Prepare a Web Server
![Screenshot from 2023-11-19 22-26-04](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/8c021158-fbdc-4438-bbec-812f1038776a)

1. Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
![Screenshot from 2023-11-19 22-27-36](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/164a9d4e-b74a-42e2-9c4d-1b2267bd95d6)

2. Attach all three volumes one by one to your Web Server EC2 instance.
![Screenshot from 2023-11-19 23-14-48](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/083fb6ed-ae36-4256-bcf7-a786b7cb04c4)
 
2. Open up the Linux terminal to begin configuration.
![Screenshot from 2023-11-19 23-19-49](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/ee0d2888-05f7-4df5-a8f5-229cf2cbdd1b)
  
3. Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. 
![Screenshot from 2023-11-20 21-22-03](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/83b0d04d-e22a-41a3-b0e0-fca6f3fdcd63)
All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there - their names will likely be xvdf, xvdh, xvdg.
![Screenshot from 2023-11-20 21-41-21](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/ea019936-def1-45bb-822b-68c7061999f1)

4. Use df -h command to see all mounts and free space on your server
![Screenshot from 2023-11-20 21-47-04](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/948594c5-bd95-4713-b285-d202530e812d)

5. Use gdisk utility to create a single partition on each of the 3 disks.
sudo gdisk /dev/xvdf
![Screenshot from 2023-11-20 22-06-01](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/57122569-33b3-4274-82ce-f4b024a04784)
Creating single partition on disk.
![Screenshot from 2023-11-20 23-05-31](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/b92c64c1-7ccb-4d18-a43e-d238c76c1f89)

5. Use lsblk utility to view the newly configured partition on each of the 3 disks. 
![Screenshot from 2023-11-20 23-07-01](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/b15430e6-b9ac-45e3-a93e-4ced11187ee0)

6. Install lvm2 package using sudo yum install lvm2. 
![Screenshot from 2023-11-20 23-14-00](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/774c5ffe-e515-4480-b469-d18bb0ad4e15)
Run sudo lvmdiskscan command to check for available partitions.
![Screenshot from 2023-11-20 23-18-40](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/51bbcb8b-466d-4081-b12f-c137a11b237e)

7. Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.
    sudo pvcreate /dev/xvdf1
    sudo pvcreate /dev/xvdg1
    sudo pvcreate /dev/xvdh1
![Screenshot from 2023-11-20 23-23-23](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/76b7b4ba-91eb-4ad5-9d05-90ee15a2b313)

8. Verify that your Physical volume has been created successfully by running sudo pvs.
![Screenshot from 2023-11-20 23-25-03](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/a23dd50d-8d5b-4930-ae18-24f818ef8e41)

9. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg 
    sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
![Screenshot from 2023-11-20 23-28-30](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/02677547-8c2d-4bd5-a88b-b5c8e0231bff)

10. Verify that your VG has been created successfully by running sudo vgs 
![Screenshot from 2023-11-20 23-29-45](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/d4175a98-77ca-4887-9bb8-377ce14097ca)

11. Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs. 
  sudo lvcreate -n apps-lv -L 14G webdata-vg
  sudo lvcreate -n logs-lv -L 14G webdata-vg
![Screenshot from 2023-11-20 23-39-47](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/c650f301-1bb5-4770-b493-dddb26c7b994)

12. Verify that your Logical Volume has been created successfully by running sudo lvs.
![Screenshot from 2023-11-20 23-41-16](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/bc947e05-d0cb-4cbe-b906-da0155209b06)

13. Verify the entire setup 
  sudo vgdisplay -v #view complete setup - VG, PV, and LV
  sudo lsblk 
![Screenshot from 2023-11-20 23-45-40](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/ab7f47c6-3283-4a29-a088-d589f7d180c0)

14. Use mkfs.ext4 to format the logical volumes with ext4 filesystem 
  sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
  sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
![Screenshot from 2023-11-20 23-49-14](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/577d8183-b3f9-45e8-9b9f-f8f04c910fc1)

15. Create /var/www/html directory to store website files
    sudo mkdir -p /var/www/html
![Screenshot from 2023-11-20 23-51-01](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/af52e1c1-d19a-4ad9-bf4c-dae8b809d90d)

17. Create /home/recovery/logs to store backup of log data
    sudo mkdir -p /home/recovery/logs
![Screenshot from 2023-11-20 23-53-24](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/7589a9d7-e25c-4b3c-9570-c61f0fcc36e2)

17. Mount /var/www/html on apps-lv logical volume
    sudo mount /dev/webdata-vg/apps-lv /var/www/html/
![Screenshot from 2023-11-20 23-54-46](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/e29b362b-4a95-4f01-9324-ed601e6a0de9)

18. Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system) 
    sudo rsync -av /var/log/. /home/recovery/logs/
![Screenshot from 2023-11-20 23-56-20](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/d3363e2a-6746-4c50-901f-d0fc90dccb1d)

19. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important) 
    sudo mount /dev/webdata-vg/logs-lv /var/log
![Screenshot from 2023-11-20 23-57-52](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/d1f5ed27-fd96-4a3c-ad94-fe16aa02fe59)

20. Restore log files back into /var/log directory 
    sudo rsync -av /home/recovery/logs/log/. /var/log
![Screenshot from 2023-11-21 00-00-48](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/5cfffed3-6c15-46e7-a6a7-56135947b8fc)

21. Update /etc/fstab file so that the mount configuration will persist after restart of the server. 
The UUID of the device will be used to update the /etc/fstab file;
    sudo blkid
![Screenshot from 2023-11-21 00-03-03](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/8e5238ba-d6ce-4bf2-b7b6-1228eb8c81e0)

sudo vi /etc/fstab
Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
![Screenshot from 2023-11-21 00-20-00](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/3d087218-9803-4780-990b-5750ead19e0d)

22. Test the configuration and reload the daemon
    sudo mount -a
    sudo systemctl daemon-reload
![Screenshot from 2023-11-21 00-23-59](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/da50292b-237b-4fba-9dc6-a8c26f715e1d)



24. Verify your setup by running df -h, output must look like this:
![Screenshot from 2023-11-21 00-26-05](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/b49f4328-bbab-42a8-aded-ddc5fed4662f)

# Installing wordpress and configuring to use MySQL Database
## Step 2 — Prepare the Database Server
Launch a second RedHat EC2 instance that will have a role - 'DB Server' Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.
Step 3 — Install Wordpress on your Web Server EC2
    1. Update the repository
       sudo yum -y update
![Screenshot from 2023-11-22 23-16-31](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/7f176a75-70a4-4e6b-b0ec-fd7f9952a2ec)

Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs. 
     sudo lvcreate -n db-lv -L 14G webdata-vg
     sudo lvcreate -n logs-lv -L 14G webdata-vg
![Screenshot from 2023-11-23 00-20-55](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/cbf4f498-78fc-4144-87a6-dfef4546f4eb)

Verify that your Logical Volume has been created successfully by running sudo lvs
![Screenshot from 2023-11-23 00-35-11](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/50d7c595-5ad2-49e4-a98a-ae1223eceb94)

Verify the entire setup 
     sudo vgdisplay -v #view complete setup - VG, PV, and LV
     sudo lsblk
![Screenshot from 2023-11-23 00-39-17](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/b07062a8-5911-4dda-9bbb-2167ef38f041)

Use mkfs.ext4 to format the logical volumes with ext4 filesystem 
     sudo mkfs -t ext4 /dev/webdata-vg/db-lv
     sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
![Screenshot from 2023-11-23 00-42-18](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/36a77227-0222-4c62-97dc-0a06f079e371)

Create /db directory to store database files
     sudo mkdir -p /db

Create /home/recovery/logs to store backup of log data
     sudo mkdir -p /home/recovery/logs
![Screenshot from 2023-11-23 00-53-29](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/2c254e4b-add8-445e-a923-69899d8fd6b7)

Mount /db on apps-lv logical volume
     sudo mount /dev/webdata-vg/db-lv /db/
![Screenshot from 2023-11-23 20-47-24](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/beb83014-4e49-4ba4-8e8f-6bd8c67e15ba)

## Step 3 — Install Wordpress on your Web Server EC2
    1. Update the repository
       sudo yum -y update
![Screenshot from 2023-11-23 21-04-13](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/9694ef63-0f15-470f-a7c6-e83125b1f888)

   2. Install wget, Apache and it's dependencies
       sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
![Screenshot from 2023-11-23 21-06-04](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/36569d7a-ea5e-4b84-929e-cf059edcc292)

   3. Start Apache
      sudo systemctl enable httpd
      sudo systemctl start httpd
![Screenshot from 2023-11-23 21-09-37](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/f0324eba-dc6e-42a4-9d35-96b5ed571a1d)

## To install PHP and it's depemdencies
   sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
![Screenshot from 2023-11-23 21-12-22](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/0dcd2dac-cdac-4b19-80f3-692e663391ed)
      
   sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
![Screenshot from 2023-11-23 21-12-22](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/4f8b8326-3d48-42d5-83d0-02e54ad43868)

   sudo yum module list php
   sudo yum module reset php
![Screenshot from 2023-11-23 21-20-17](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/cec2f018-82ca-44c6-9f4d-19ca4a5cadfa)

   sudo yum module enable php:remi-7.4
![Screenshot from 2023-11-23 21-25-54](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/ab0759a2-c2a0-4a1e-92a8-ba8a9ce47a9c)

   sudo yum install php php-opcache php-gd php-curl php-mysqlnd
![Screenshot from 2023-11-23 21-26-31](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/dc44c5c8-e6a6-436f-9632-4ccd0ce164d1)

   sudo systemctl start php-fpm
   sudo systemctl enable php-fpm
   setsebool -P httpd_execmem 1
![Screenshot from 2023-11-23 21-29-50](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/592d4d5b-ac3f-49a5-abb4-414d894d3ba6)

## Restart Apache
   sudo systemctl restart httpd
![Screenshot from 2023-11-23 22-10-23](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/4c4fbfbd-8bdb-4d8e-a340-e45124a9a76e)

### Download wordpress and copy wordpress to var/www/html
   mkdir wordpress
   cd wordpress
   sudo wget http://wordpress.org/latest.tar.gz
![Screenshot from 2023-11-23 22-13-34](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/a0489c97-4976-4ab1-a05b-3d71462d384f)

   sudo tar xzvf latest.tar.gz
![Screenshot from 2023-11-23 22-14-39](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/3819bae7-fc89-48fe-a66c-caa69956a534)

   sudo rm -rf latest.tar.gz
   cp wordpress/wp-config-sample.php wordpress/wp-config.php
   cp -R wordpress /var/www/html/
![Screenshot from 2023-11-23 22-17-37](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/70cc0b41-0b5f-4002-8361-87ec779def90)

## Configure SELinux Policies
   sudo chown -R apache:apache /var/www/html/wordpress
   sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
   sudo setsebool -P httpd_can_network_connect=1
![Screenshot from 2023-11-23 22-20-36](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/cb9c3d6a-59d1-457f-8ca3-b366ee46b099)

## Step 4 — Install MySQL on your DB Server EC2
   sudo yum update
![Screenshot from 2023-11-23 22-25-29](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/568c6066-eb45-45e4-b0de-5b67f58fae96)

   sudo yum install mysql-server
![Screenshot from 2023-11-23 22-31-51](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/007ccaf5-c615-408a-b52e-a972be3a4e67)

### Verify that the service is up and running by using sudo systemctl status mysqld, if it is not running, restart the service and enable it so it will be running even after reboot:
   sudo systemctl restart mysqld
   sudo systemctl enable mysqld
![Screenshot from 2023-11-23 22-35-22](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/5123de40-ee1b-4350-bb14-ef87d6ef1f69)

## Step 5 — Configure DB to work with WordPress
   sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
![Screenshot from 2023-11-23 22-38-30](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/d73c5c2a-f0fa-468e-ac24-89583c1186d8)

## Step 6 — Configure WordPress to connect to remote database.
Opening MySQL port 3306 on DB Server EC2. For extra security.
![Screenshot from 2023-11-23 23-44-23](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/38455ab5-3d0a-4ce4-9017-27373228f8f3)

## Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client 
   sudo yum install mysql
![Screenshot from 2023-11-23 23-48-18](https://github.com/PromiseNwachukwu/Implementing-WordPress-Website/assets/109115304/498dbbd4-b20b-41ec-80fb-e4a6c30cc4ca)

sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
    2. Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.
    3. Change permissions and configuration so Apache could use WordPress:
    4. Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation's IP)
    5. Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/

