# Web Solution With WordPress
## Implementing LVM on Linux servers (Web and Database servers)
### Step 1 — Prepare a Web Server
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
10. Verify that your VG has been created successfully by running sudo vgs 

