# WEB SOLUTION WITH WORDPRESS

## This project is aimed at preparing storage infrastructure on two Linux servers and implement a basic web solution using WordPress

### Step 1 - Prepare a Web Server

Spin your instance and attached 3 volumes to the web server

open up the Linus terminal to begin configuration

1. Inspect block deviced using `lsblk`

 ![InspectBlockDev](./Images/Inspect_block_devices.PNG)

 Check for all  mounts and free space

 ![Available_M&F](./Images/1-Available_mounts_freespace.PNG)

- Create single partition for all the disks. You will get outcomes that looks like what you see below;

 ![Partition_Disk1](./Images/1-Partition_disk_web1.PNG)

 ![Partition_Disk2](./Images/1-Partition_disk_web2.PNG)

 ![Partition_Disk3](./Images/1-Partition_disk_web3.PNG)

- Use the `lsblk` command to view the newly configured partitions

 ![ViewConfig_Partitions](./Images/1-newly_configured_partition.PNG)

- Install lvm2

 `sudo yum install lvm`

 ![Install_LVM2](./Images/1-install_lvm2.PNG)

 ![Install_LVM2](./Images/1-install_lvm2(b).PNG)

- Use pvcreate utility to mark each of 3 disks as physical volumes

 ![Mark_disk_as_PVs](./Images/1-mark_disks_as_PV.PNG)

- Verify Physical Volumes

 ![PVs](./Images/1-verify_pvs.PNG)

- Use lvcreate utility to create 2 logical volumes

 `sudo lvcreate -n apps-lv -L 14G webdata-vg`
 `sudo lvcreate -n logs-lv -L 14G webdata-vg`

- Verify it was successfully created

 `sudo lvs`

 ![VerifyLVs](./Images/1-verify_lv.PNG)

- Verify the entire setup

 `sudo vgdisplay -v`

 ![Verify_Setup](./Images/1-verify_entire_setup.PNG)

 `sudo lsblk`

 ![Verify_blks](./Images/1-verify_entire_setup2.PNG)

- Use mkfs.ext4 to format the logical volumes with ext4 filesystem

 `sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`

 `sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

 ![Format_LVs](./Images/1-create_filesystem.PNG)

- Create /var/www/html directory to store website files

 `sudo mkdir -p /var/www/html`

- Create /home/recovery/logs to store backup of log data

 `sudo mkdir -p /home/recovery/logs`

- Mount /var/www/html on apps-lv logical volume

 `sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

 ![Mount_on_apps](./Images/1-mount_on_apps-lv.PNG)

  Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs

 `sudo rsync -av /var/log/. /home/recovery/logs/`

- Mount /var/log on logs-lv logical volume

 sudo mount /dev/webdata-vg/logs-lv /var/log

 ![Mount_on_logs](./Images/1-mount_on_logs-lv.PNG)

### UPDATE THE `/ETC/FSTAB` FILE

- Get the UUID of the device

 `sudo blkid`

 ![check_4_UUID](./Images/2-Check_for_UUID.PNG)

- Update /etc/fstab with the UUID

 `sudo vi /etc/fstab`

 ![Update_fstab](./Images/2-update_fstab.PNG)

- Test the configuration

 `sudo mount -a`

 ![Test_Config](./Images/2-test_config.PNG)

- Reload daemon

 `sudo systemctl daemon-reload`

 ![Reload_daemon](./Images/2-reloaded_the_daemon.PNG)

- Verify Setup

 `df -h`

 ![Verify_setup](./Images/2-verify_setup.PNG)

### Repeat same process for the Database Server

Spin the database server instance and attach 3 volumes to it.

- Create single partition for all the disks. You will get outcomes that looks like what you see below;

 ![Partition_Disk1](./Images/db-partition_disk_db1.PNG)

 ![Partition_Disk2](./Images/db-partition_disk_db2(xvdg).PNG)

 ![Partition_Disk3](./Images/1-Partition_disk_web3.PNG)

- View newly configured partition

 `lsblk`

 ![view_new_config](./Images/db-view%20configured_partition.PNG)

- Install lvm2 package.

 `sudo yum install lvm2`

 ![Install_lvm2](./Images/db-install_lvm2.PNG)

- Use pvcreate utility to mark each of 3 disks as physical volumes

 `sudo pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

 ![mark_disks_as_PVs](./Images/db-mark_disks_as_physicalvolume.PNG)

- Add all 3 PVs to a volume group (VG)

 `sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

 ![Add_PVs_to_VGs](./Images/db-add_PVs_to_VGs.PNG)

- Use lvcreate utility to create 2 logical volumes

 `sudo lvcreate -n db-lv -L 20G database-vg`

 ![create_lvs](./Images/db-create_LVs.PNG)

- Use mkfs.ext4 to format the logical volumes with ext4 filesystem

 `sudo mkfs.ext4 /dev/database-vg/db-lv`

 ![Format_LVs](./Images/db-format_LVs.PNG)

- Create /db directory to store data

 `sudo mkdir /db`

- Mount /dev/database-vg/db-lv on /db

 `sudo mount /dev/database-vg/db-lv /db`

 ![Mount_on_db](./Images/db-mount_on_db.PNG)

- Updated FSTAB

 `sudo vi /etc/fstab`

- Test Configuration

 `sudo mount -a`

 ![Verify_configuration](./Images/db-verify_config.PNG)

- Reload Daemon

 `sudo systemctl daemon-reload`

 ![Reload_daemon](./Images/db-reload_daemon.PNG)

### Install WordPress on your Web Server EC2

- Update the repository

 `sudo yum -y update`

- Install wget, Apache and it’s dependencies

 `sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

 ![Install-apache](./Images/step3-install_wget_etal.PNG)

- Start Apache
 `sudo systemctl enable httpd`
 `sudo systemctl start httpd`

 ![StartApache](./Images/step3-start_apache.PNG)

- Install PHP and it’s dependencies

`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![php1](./Images/step3-install_php_dependencies1.PNG)

`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![php2](./Images/step3-install_php_dependencies2.PNG)

`sudo yum module list php`
`sudo yum module reset php`
`sudo yum module enable php:remi-7.4`
`sudo yum install php php-opcache php-gd php-curl php-mysqlnd`

![php3](./Images/step3-install_php_dependencies3..PNG)

`sudo systemctl start php-fpm`
`sudo systemctl enable php-fpm`
`setsebool -P httpd_execmem 1`

- Restart Apache

 `sudo systemctl restart httpd`

 ![Restart_Apache](./Images/step3-start_apache.PNG)

- Download wordpress and copy wordpress to var/www/html
 `mkdir wordpress`
 `cd   wordpress`
 `sudo wget http://wordpress.org/latest.tar.gz`

 ![Download_wordpress](./Images/step3-download_wordpress.PNG)

 `cp wordpress/wp-config-sample.php wordpress/wp-config.php`

 `cp -R wordpress /var/www/html/`

 ![copy_wordpress](./Images/step3-copy_wordpress1.PNG)

- Configure SELinux Policies

 `sudo chown -R apache:apache /var/www/html/wordpress`

 `sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`

 `sudo setsebool -P httpd_can_network_connect=1`

 ![Configure_SELinux](./Images/step3-config_SELinux_Policy.PNG)

### Install MySQL on your DB Server EC2

- Update server and install MySql-Server

 `sudo yum update`

 `sudo yum install mysql-server`

 ![Install_Mysql](./Images/step4-install_mysql.PNG)

- Verify MySQL server is running

 `sudo systemctl restart mysqld`
 `sudo systemctl enable mysqld`
 `sudo systemctl status mysqld`

 ![verify_service](./Images/step4-verify_service.PNG)

### Configure DB to work with WordPress

- Configurw Database to work with WordPress

 Run the command `sudo mysql`
 Then;
 `CREATE DATABASE wordpress;`

 ![Database](./Images/Step5-Database.PNG)

### Configure WordPress to connect to remote database

- Open MySQL port 3306 on DB Server EC2

- Install MySQL client and test that you can connect from your Web Server to your DB server
 `sudo yum install mysql`

  `sudo mysql -u verve -p -h 172.31.84.77`

- Change permissions and configuration so Apache could use WordPress

 ![changed_Perm_Config](./Images/step6-Change_P%26Config.PNG)

- Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2

 ![Access_wordpress_from_browser](./Images/Step6-Wordpress.PNG)

 Set up your login details and login. You should see something like the screenshots below

 ![wordpressLogin](./Images/Step6-Wordpress_installed_successfully.PNG)

 ![Wordpress_dashboard](./Images/Step6-Wordpress-Dashboard.PNG)
