<img width="1171" height="911" alt="2-3-4 magento-recovery" src="https://github.com/user-attachments/assets/b9eb5724-2828-409f-809e-beed975f4d90" /># 🌆 Huawei Cloud Project: Enterprise Cloud Migration & Scalable Magento E-Commerce Platform

**A production-grade Magento e-commerce website on Huawei Cloud, built with ECS + RDS, hardened with cross-network connectivity (VPC Peering & VPC Endpoint), protected by CSBS backup/restore, scaled with ELB, and accelerated with DCS (Redis) and DDM for read/write splitting, validated end-to-end with CPTS performance testing.**

This project simulates a real enterprise cloud-migration scenario: a 15-year-old company moving its internal IT systems to the cloud. The exercise walks through standing up the cloud data center, deploying and securing a Magento website, proving connectivity between simulated on-premises and cloud environments, adding backup protection with full disaster-recovery validation, and finally load-testing the platform before and after scaling it out with a load balancer and a database middleware layer.

---

## 🌐 Why This Project?

Most cloud demos stop at "deploy a server." This one goes further: it reproduces the full lifecycle a migrated enterprise workload actually needs, including network isolation and controlled cross-VPC access, a real application stack (Apache/PHP/Composer/Magento) wired to a managed database, backup and disaster-recovery drills performed *for real* (delete the data, then prove it comes back), and a measurable before/after performance story once load balancing, caching, and database middleware are introduced. It's a solid demonstration of how availability, security, and performance decisions actually show up in a cloud console rather than just on a whiteboard.

---

## 🗺️ Network Topology

(paste screenshot here)

**AZ2** simulates the local (on-premises) data center used for O&M and backup. **AZ1** simulates the cloud data center running the live website. The two are connected via **VPC Peering** (bidirectional) and a **VPC Endpoint** (unidirectional, point-to-point access for security-sensitive services), reflecting how a real migration keeps some legacy systems talking to the new cloud environment without fully opening the network.

---

## 🧱 Key Huawei Cloud Services Used

- **ECS (Elastic Cloud Server)**: Hosts the Magento web server, load-balancer backend, and connectivity-test instances.
- **RDS (Relational Database Service)**: Managed MySQL backend for Magento, including a read replica.
- **VPC + VPC Peering + VPC Endpoint (VPCEP)**: Network isolation, bidirectional peering between data centers, and unidirectional service exposure.
- **EVS (Elastic Volume Service)**: Additional data disk, manually partitioned and mounted.
- **CSBS (Cloud Server Backup Service)**: Scheduled and manual ECS backups, restore, and image creation.
- **IMS (Image Management Service)**: Private full-ECS image built from a backup, used to clone the web tier.
- **ELB (Elastic Load Balance)**: Layer-4 (SSH) and Layer-7 (HTTP) load balancing across backend servers.
- **DCS (Distributed Cache Service, Redis)**: In-memory caching layer.
- **DDM (Distributed Database Middleware)**: Sits between the application and RDS, enabling read/write splitting against a primary + read replica.
- **CPTS (Cloud Performance Test Service)**: Concurrency/load testing with gradient-increment pressure profiles.

---

## 🛠️ Tasks

### 🚀 Task 1: Setting Up a Data Center and Application Website

#### Subtask 1: Create a VPC and configure a security group
Created `vpc-web`, `VPC-Peering`, and `VPC-EP`, and configured security group `sg-web` with the minimum required ports, binding it to all instances used later in the exam.

**📷 `1-1-1 vpc-all`**: <img width="1920" height="1080" alt="1-1-1 vpc-all" src="https://github.com/user-attachments/assets/a4f49e01-6107-464d-a117-ed8e0af7bdf5" />


#### Subtask 2: Purchase ECS instances
Launched `ecs-web` (AZ1, 2 vCPUs | 4 GiB, CentOS 7.2, `vpc-web` / `sg-web`).

**📷 `1-2-1 ecs-web`**: <img width="1920" height="912" alt="1-2-1 ecs-web" src="https://github.com/user-attachments/assets/66efe36f-910f-4659-9a5e-539178e6e9cf" />


Launched `ecs-EP` in AZ2.

**📷 `1-2-2 ecs-EP`**: <img width="1920" height="912" alt="1-2-2 ecs-EP" src="https://github.com/user-attachments/assets/6193fd87-9863-4620-bf54-0f47d30c7fc0" />


Launched `ecs-Peering` in AZ2.

**📷 `1-2-3 ecs-Peering`**: <img width="1920" height="912" alt="1-2-3 ecs-Peering" src="https://github.com/user-attachments/assets/d391ab03-4777-44e8-9b96-bfa7b4ecec7c" />


#### Subtask 3: Create RDS data
Provisioned `rds-web` (MySQL 5.7, 4 vCPUs | 8 GB) as the Magento backend database, in `vpc-web` / `sg-web`.

**📷 `1-3-1 rds-web`**: <img width="1920" height="912" alt="1-3-1 rds-web" src="https://github.com/user-attachments/assets/5bc189a5-65b2-4306-aaa3-3dc86c346bbd" />


#### Subtask 4: Perform a connectivity test
Before creating the VPC peering connection, checked connectivity between `ecs-web` and `ecs-Peering`: no connection yet.

**📷 `1-4-1 connect-before`**: <img width="1920" height="1016" alt="1-4-1 connect-before" src="https://github.com/user-attachments/assets/2983e4eb-3449-46d8-9715-7257d0d85016" />


Created the **VPC Peering** connection between `vpc-web` and `VPC-Peering`.

**📷 `1-4-2 vpc-peering`**: <img width="1920" height="912" alt="1-4-2 vpc-peering" src="https://github.com/user-attachments/assets/b92b0c56-e15f-46b8-a5c2-eda253e799c7" />


Added the local and peer routes so traffic can flow in both directions across the peering connection.

**📷 `1-4-3 peering-route`**: <img width="1920" height="912" alt="1-4-3 peering-route" src="https://github.com/user-attachments/assets/cdf164ed-6784-4ca2-89c8-202ca64cf26e" />


Re-verified connectivity between `ecs-web` and `ecs-Peering`: now bidirectional.

**📷 `1-4-4 connect-end`**: <img width="1920" height="1018" alt="1-4-4 connect-end" src="https://github.com/user-attachments/assets/86bbc4b4-ba3a-493b-914d-a6f985a01c6e" />


Created a **VPC Endpoint Service** exposing only SSH (port 22) on `ecs-web`.

**📷 `1-4-5 EP-service`**: <img width="1920" height="912" alt="1-4-5 EP-service" src="https://github.com/user-attachments/assets/1c52f87a-e922-46c2-b088-163fc6a71458" />


Accepted the connection request on the endpoint service's connection management page.

**📷 `1-4-6 EP-service-connect`**: <img width="1920" height="912" alt="1-4-6 EP-service-connect" src="https://github.com/user-attachments/assets/43274ce3-895e-43ae-86d3-69a616a63c33" />


Created the matching **VPC Endpoint** and connected it to the endpoint service.

**📷 `1-4-7 EP`**: <img width="1920" height="912" alt="1-4-7 EP" src="https://github.com/user-attachments/assets/2fe4ad44-eb88-47dd-b732-66cbbd9937b2" />


From `ecs-EP`, connected through the VPC endpoint's private IP address, which forwards the SSH session to `ecs-web`. Traffic flow: `ecs-EP → (SSH) → VPC Endpoint → ecs-web`, a unidirectional, point-to-point path, unlike the bidirectional peering connection above.

**📷 `1-4-8 EP-connect`**: <img width="697" height="251" alt="1-4-8 EP-connect" src="https://github.com/user-attachments/assets/8284566e-e52d-4e14-8b93-c184be0ad4bd" />


Checked the inbound rules of `sg-web` to confirm only the required ports were opened.

**📷 `1-4-9 sg`**: <img width="1920" height="912" alt="1-4-9 sg" src="https://github.com/user-attachments/assets/7afe627c-d4f0-40c6-898e-fcfd80fc5f66" />


#### Subtask 5: Configure and install the Magento e-commerce website
Connected to `ecs-web` via PuTTY, installed and configured Apache, then edited `/etc/httpd/conf/httpd.conf` to allow overrides and enable `mod_rewrite`.

**📷 `1-5-1 httpd.conf`**: <img width="1144" height="1015" alt="1-5-1 httpd conf" src="https://github.com/user-attachments/assets/fdfb65b7-e279-452b-8ef3-b268e1fe143d" />

Installed PHP 7.0.33 with the required extensions and tuned `php.ini` (memory limit, timezone). Then confirmed the installed PHP version after configuration.

**📷 `1-5-3 php`**: <img width="706" height="190" alt="1-5-3 php" src="https://github.com/user-attachments/assets/b9d3d14d-f43e-4dab-a142-62356e1494f2" />


Installed Composer 1.10.19.

**📷 `1-5-4 composer`**: <img width="995" height="793" alt="1-5-4 composer" src="https://github.com/user-attachments/assets/9d67f5c5-ded9-4608-b218-4e65fb6362c7" />


Cloned Magento 2.1.0 into the web root and ran `composer install`, then accessed `http://<server-IP>/magento` in a browser to reach the setup wizard.

**📷 `1-5-5 magento`**: <img width="1186" height="964" alt="1-5-5 magento" src="https://github.com/user-attachments/assets/22401667-dd27-45e6-9e09-a9f5e979f279" />


Created the Magento database on RDS through the web installer.

**📷 `1-5-6 create-db`**: <img width="1920" height="912" alt="1-5-6 create-db" src="https://github.com/user-attachments/assets/714a4b2a-1bb4-48dc-a807-ec295c07e5f7" />


Created the `admin` database user with full privileges on the `magento` database, accessible from any host.

**📷 `1-5-7 add-db-user`**: <img width="1920" height="912" alt="1-5-7 add-db-user" src="https://github.com/user-attachments/assets/05bcc584-6bad-4095-b80e-e32ea45b44ea" />


Pointed the installer at the `rds-web` database to complete the database connection step.

**📷 `1-5-8 connect-db`**: <img width="1186" height="965" alt="1-5-8 connect-db" src="https://github.com/user-attachments/assets/2849d0c8-21d9-4083-a2a3-9c2c0b61def1" />


Set the store's front-end and admin URLs during installation.

**📷 `1-5-9 web-conf`**: <img width="1186" height="911" alt="1-5-9 web-conf" src="https://github.com/user-attachments/assets/74a742ce-0aec-4514-8068-717a2977584d" />


Finished the install and confirmed the Magento admin dashboard loaded successfully.

**📷 `1-5-10 magento-end`**: <img width="1221" height="963" alt="1-5-10 magento-end" src="https://github.com/user-attachments/assets/5f8cdb94-f961-4e36-8f5c-e09ce1af9e40" />

#### Subtask 6: Buy an EVS disk and mount it to ecs-web
Purchased a 10 GiB General Purpose SSD, `volume-web-data`, in the same AZ as `ecs-web`, and attached it as a data disk.

**📷 `1-6-1 volume`**: <img width="1221" height="906" alt="1-6-1 volume" src="https://github.com/user-attachments/assets/6aa17371-d51e-4a57-a22a-de035a90b278" />


Initialized the disk by hand: `lsblk` → `fdisk -l` → `fdisk /dev/vdb` (new primary partition, write changes) → `mkfs.ext4 /dev/vdb1` → mounted to `/mnt/sdc`, then ran `df -TH` to confirm the mount.

**📷 `1-6-2 mount`**: <img width="1121" height="195" alt="1-6-2 mount" src="https://github.com/user-attachments/assets/49f51982-177c-43d7-925e-0dce32beca5f" />


Pulled up `history` to show the exact partitioning/formatting/mounting commands used.

**📷 `1-6-3 evs-conf`**: <img width="520" height="258" alt="1-6-3 evs-conf " src="https://github.com/user-attachments/assets/86ab7bbe-9908-4bb5-947e-2208d0ad8e85" />


Created `test.txt` in `/mnt/sdc` containing `hello ICT competition` and displayed its contents to confirm the mounted partition was writable.

**📷 `1-6-4 testfile`**: <img width="674" height="110" alt="1-6-4 testfile" src="https://github.com/user-attachments/assets/98583424-3e18-4c4c-ab5a-f3faf4aa0738" />

---

### 💾 Task 2: Performing Backup Protection

#### Subtask 1: Create an ECS backup vault
Purchased a Cloud Server Backup vault, `vault-ecs`, associated it with `ecs-web`, and attached an automatic weekly backup policy (`policy_ecs`, Mondays 22:00, 1-month retention).

**📷 `2-1-1 vault-ecs`**: <img width="1171" height="911" alt="2-1-1 vault-ecs" src="https://github.com/user-attachments/assets/953656e3-a14e-4134-9460-a6b3e1e8af59" />


Reviewed the backup policy configuration page.

**📷 `2-1-2 ecs backup policy`**: <img width="1171" height="911" alt="2-1-2 ecs backup policy" src="https://github.com/user-attachments/assets/2751bed8-ff0d-4576-82f1-4dcf0585c424" />


Ran an immediate manual backup and confirmed it appeared in the backup replica list.

**📷 `2-1-3 ecs manualbal`**: <img width="1171" height="911" alt="2-1-3 ecs manualbal" src="https://github.com/user-attachments/assets/7f354de2-2ea6-47a2-9192-28eee4efea54" />


#### Subtask 2: Verify backup protection
Simulated data loss by deleting the entire website directory (`/var/www/html`) on `ecs-web`.

**📷 `2-2-1 delete`**: <img width="454" height="119" alt="2-2-1 delete" src="https://github.com/user-attachments/assets/d85316fd-4ae3-4a9d-bc1d-477c20b7e72b" />


Tried loading the site via its EIP: it now fails.

**📷 `2-2-2 web error`**: <img width="1171" height="960" alt="2-2-2 web error" src="https://github.com/user-attachments/assets/9b9e785f-736d-4756-bd3a-89bc207c9a62" />


Restored `ecs-web` from the CSBS backup and checked the restore task's details.

**📷 `2-2-3 task`**: <img width="1171" height="911" alt="2-2-3 task" src="https://github.com/user-attachments/assets/52142210-2f6a-47d6-9686-d2c6ec8323de" />


Reloaded the site via the same EIP: it's back online, no redeployment needed.

**📷 `2-2-4 web recovery`**: <img width="1171" height="968" alt="2-2-4 web recovery" src="https://github.com/user-attachments/assets/c8143df1-d39a-4945-a6ed-9efe821d6364" />


Built a full-ECS private image, `image-web`, from the backup.

**📷 `2-2-5 image-web`**: <img width="1171" height="911" alt="2-2-5 image-web" src="https://github.com/user-attachments/assets/b2e1bec5-f534-4999-a623-fc35fef2fdf7" />

Launched `ecs-AZ2` in AZ2 from that image, proving the backup doubles as a portable golden image (this image is reused to clone the load-balancer backend in Task 3).

**📷 `2-2-6 ecs-AZ2`**: <img width="1920" height="912" alt="2-2-6 ecs-AZ2" src="https://github.com/user-attachments/assets/50fb894a-fd79-481e-8206-b2a30345edb8" />


Ran the mount/config commands on `ecs-AZ2` to bring the cloned data disk online.

**📷 `2-2-7 AZ2-conf`**: <img width="562" height="380" alt="2-2-7 AZ2-conf" src="https://github.com/user-attachments/assets/5706cbab-be39-4524-85a8-cc22449ce5a0" />


Confirmed `test.txt` from Task 1 was present and intact on `ecs-AZ2`.

**📷 `2-2-8 AZ2-testfile`**: <img width="621" height="76" alt="2-2-8 AZ2-conf" src="https://github.com/user-attachments/assets/654010f5-e08b-4927-a8f1-12048f84dea7" />


#### Subtask 3: Restore the database
Registered a real customer account on the live Magento site as an end user.

**📷 `2-3-1 magento-customer-addr-1`**: <img width="1171" height="964" alt="2-3-1magento-customer-addr-1" src="https://github.com/user-attachments/assets/15e3c528-eff4-4c56-9c08-88ef3043c040" />


Confirmed the new account appeared in the admin's customer list.

**📷 `2-3-1 magento-customer-addr-2`**: <img width="1171" height="965" alt="2-3-1magento-customer-addr-2" src="https://github.com/user-attachments/assets/7f7648ca-1570-4a47-be95-05628db6078d" />


Manually created a database backup, `magento-backup-1`.

**📷 `2-3-2 magento-backup-1`**: <img width="1171" height="911" alt="2-3-3 magento-backup-1" src="https://github.com/user-attachments/assets/3a66498e-546b-4860-8d77-5180b82c3a4b" />


Deleted the Magento database outright to simulate an administrator's mistake.

**📷 `2-3-3 magento-delete-1`**: <img width="1171" height="129" alt="2-3-3 magento-delete-1" src="https://github.com/user-attachments/assets/e22a7905-6f60-40fd-b40c-c9f0480dd552" />


Refreshed the Magento site: it's now broken with the database gone.

**📷 `2-3-3 magento-delete-2`**: <img width="1171" height="965" alt="2-3-3 magento-delete-2" src="https://github.com/user-attachments/assets/fb714cbc-17f6-48e2-83b4-61d47908095a" />


Restored the RDS instance from the database backup, keeping the same private IP so the application config needed no changes.

**📷 `2-3-4 magento-recovery`**: <img width="1171" height="911" alt="2-3-4 magento-recovery" src="https://github.com/user-attachments/assets/b5637cd0-5181-4ec4-bde0-073b22d6d668" />


Logged back in as an administrator and confirmed the customer's original registration data had come back intact.

**📷 `2-3-5 web recovery`**: <img width="1171" height="911" alt="2-3-5 web recovery" src="https://github.com/user-attachments/assets/e344048e-8899-4a2b-a316-5571e441bbd4" />


---

### ⚡ Task 3: Performing a Website Performance Test

#### Subtask 1: Baseline performance test
Created a CPTS test project, `webproject`.

**📷 `3-1-1 webproject`**: <img width="1920" height="912" alt="3-1-1 webproject" src="https://github.com/user-attachments/assets/7e3838a2-3d9e-4a26-9489-da5d0d85f862" />


Created a concurrent-mode test task, `webtask`, with a case `web` targeting the live Magento URL.

**📷 `3-1-2 webtask`**: <img width="1171" height="911" alt="3-1-2 webtask" src="https://github.com/user-attachments/assets/934e2160-541e-415b-9148-01271298effc" />


Configured the pressure profile: concurrency mode, gradient increment enabled, starting at 100 and stepping up to 400 concurrent users, 1 minute per pressure step.

**📷 `3-1-3 pressure-conf`**: <img width="1171" height="911" alt="3-1-3 pressure-conf" src="https://github.com/user-attachments/assets/f914b349-f3a7-401e-84e1-91bfef763e3b" />

Ran the test and downloaded the offline PDF report as the pre-scaling baseline.

**📄 `3-1-4 report`**: <img width="1920" height="912" alt="3-1-4 report" src="https://github.com/user-attachments/assets/0955de25-e3f7-4ddc-9ba4-b1d9a364aacb" />


#### Subtask 2: Load balancing and concurrency
Reused the `image-web` golden image from Task 2 to launch a second, identically-configured backend, `ecs-LB`.

**📷 `3-2-1 ecs-LB`**: <img width="1920" height="912" alt="3-2-1 ecs-LB" src="https://github.com/user-attachments/assets/84b036ec-17a2-415a-821d-c86a35ec7afe" />


Created load balancer `elb-web` with a public EIP.

**📷 `3-2-2 elb`**: <img width="1920" height="912" alt="3-2-2 elb" src="https://github.com/user-attachments/assets/0c55049c-7de1-4532-8b36-f3ae4941910f" />


Added a Layer-4 (TCP/SSH) listener and verified it by SSHing into the ELB's address and landing on `ecs-web`. And Repeated the SSH test to confirm the listener also routes to `ecs-LB`.

**📷 `3-2-3 ssh-ecs-LB`**: <img width="1920" height="912" alt="3-2-3 ssh-ecs-web   Lb" src="https://github.com/user-attachments/assets/33f2acf0-26c7-43fe-b679-29b6239c3465" />


Checked the health status of the listener and both backend servers together.

**📷 `3-2-4 ssh-status`**: <img width="1904" height="569" alt="3-2-4 ssh-status" src="https://github.com/user-attachments/assets/80a10180-158c-4e16-a518-600069b27281" />


Before adding the Layer-7 (HTTP) listener for Magento, checked the site's status: still pointed directly at `ecs-web`, public access URL unchanged.

**📷 `3-2-5 magento-before`**: <img width="1171" height="963" alt="3-2-6 magento before" src="https://github.com/user-attachments/assets/0400d49f-dd3c-44cf-a02d-3bfa5b78d9e7" />


After adding the HTTP listener, confirmed the Magento site now loads through the load balancer, still on the same public URL. Checked the health status of the HTTP listener and its backend group.

**📷 `3-2-7 magento-status`**: <img width="1904" height="561" alt="3-2-7 magento-status" src="https://github.com/user-attachments/assets/4022ddd9-505a-4d75-affa-42416e9711e7" />


Re-ran the CPTS test against the load-balanced endpoint and downloaded the report. *Result: concurrency didn't actually improve*, since both backends still pointed at the same single RDS instance, so the database, not the web tier, was the bottleneck. This is what motivated adding DCS and DDM next.

**📄 `3-2-8 report`**: <img width="1920" height="912" alt="3-2-8 report" src="https://github.com/user-attachments/assets/719ad264-fab1-4593-8d9b-96cdec8aa5b8" />


#### Subtask 3: Configure DCS (Redis)
Provisioned a Master/Standby DCS Redis instance, `dcs-web` (2 GB, 2 replicas), in `vpc-web`.

**📷 `3-3-1 dcs-web`**: <img width="1920" height="912" alt="3-3-1 dcs-web" src="https://github.com/user-attachments/assets/f6014330-c798-43ad-b096-0bcbf5a2549f" />


Compiled the Redis client from source on `ecs-web` and `ecs-LB` and connected to the managed instance with `redis-cli -h <dcs_instance_address> -p 6379`, authenticating with the instance password.

**📷 `3-3-2 dcs-conf-n`**: <img width="1628" height="260" alt="3-3-2 dcs-conf3-3-2 dcs-conf" src="https://github.com/user-attachments/assets/566dbe7d-ba75-4b09-8687-de1b08825442" />


#### Subtask 4: Configure DDM for read/write splitting
This is the step that actually fixes the database bottleneck found in Subtask 2.

Created an RDS read replica (4 vCPUs | 16 GB) for `rds-web` and confirmed the new primary/replica topology.

**📷 `3-4-1 rds-web`**: <img width="1920" height="912" alt="3-4-1 rds-web" src="https://github.com/user-attachments/assets/75baea6b-ad79-4ce4-8270-fca0a6608ddc" />


Created a DDM instance, `ddm-instance` (2 nodes, General-enhanced 8 vCPUs | 16 GB), in `vpc-web` / `sg-web`.

**📷 `3-4-2 ddm-instance`**: <img width="1221" height="912" alt="3-4-2 ddm-instance" src="https://github.com/user-attachments/assets/56814137-fbd9-44af-84fb-6d777e5c7deb" />


Created a DDM account, `root`, with full permissions.

**📷 `3-4-3 ddm-account`**: <img width="1221" height="877" alt="3-4-3 ddm-account" src="https://github.com/user-attachments/assets/bf12f34c-120d-4113-bc96-d3b945c8fee1" />


Created an unsharded schema, `db_web`, bound to `rds-web`, and recorded its connection address.

**📷 `3-4-4 db-web`**: <img width="1221" height="909" alt="3-4-4 db-web" src="https://github.com/user-attachments/assets/0acffb07-981e-42bb-b198-0b02150c74a6" />


Migrated the live Magento schema from RDS into DDM by exporting the table structure with `mysqldump --no-data`, exporting the table data separately with `mysqldump --no-create-info`, then importing both into the DDM schema over its connection string, and verified matching `admin%` tables existed on both `rds-web` and the new DDM schema.

Re-ran the CPTS performance test one final time against the DDM-backed application, this time with a measurable improvement, since writes and reads were now split across primary and replica instead of hammering a single database.

---

## ✅ Outcome

The final environment runs a real Magento storefront on RDS, reachable through both a legacy-style VPC Peering link and a locked-down VPC Endpoint for cross-network access. It survives a full simulated disaster: website files deleted and restored from CSBS, and a wiped Magento database recovered from an RDS backup with customer data intact. Its performance was measured, bottlenecked, and then genuinely improved: baseline, then load-balanced (no change, DB-bound), then cached and read/write-split via DCS and DDM (measurable gain). A complete, resilient, and *performance-validated* cloud migration.

---

## 🧠 Skills Demonstrated

Compute & application deployment (ECS, LAMP stack, Magento) · Managed databases & scaling (RDS, read replicas, DDM) · Caching (DCS/Redis) · Backup & disaster recovery (CSBS, IMS) · Load balancing (ELB L4 + L7) · Networking (VPC, VPC Peering, VPC Endpoint) · Storage (EVS, manual disk partitioning) · Performance/load testing (CPTS) · Linux administration (PuTTY, MySQL, Composer).

---

> ℹ️ *This project is my own hands-on implementation based on the 2022-2023 Huawei ICT Competition (Cloud Track) Global Final scenario. It documents the work I performed; it is not an official solution.*

---

## ✍️ Author

**Made with 💻 by Nidhal Labri**
🔗 [LinkedIn](https://www.linkedin.com/in/nidhal-labri/)
