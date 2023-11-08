---
title: Come on, let's get a netbook system
date: 2023-11-06 08:04:00
categories: 
  - Backend
tags: 
  - Backend Technology Sharing
  - Backend, , Design
  - Architecture
  - Design
  - development
  - framework
  - network
  - massive
description: Baidu.com is a popular cloud storage and file sharing platform with over 800 million users and up to 10w+PB of storage capacity. In this article, we will delve into the core functionality of the Baidu Netdisk system and how to deal with the challenges that can arise from high concurrency and massive storage.
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/b4d8594758454caa9095584856f0e405%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp
---
# 1. Introduction

## 1.1 The Melody of Youth

For example, I used to like the music talent VAE, whether it is the small bridge eaves under the night in Jiangnan, or the wild store in Guanwai where the smoke and fire are extinguished and the customers can't sleep, or the purple smoke and fragrance, or the astonishing side of the shadow of the shadow of the shadow of the shadow of the shadow of the shadow of the shadow of the shadow of the shadow of the shadow of the shadow of the shadow.

Those touching melodies would suddenly haunt my mind in the dusk after a nap, or in the morning after a heavy rain, and could not be dispersed for a long time.![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/b4d8594758454caa9095584856f0e405~tplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

I don't know if it's because I miss the people I used to listen to songs with or the memories of my youth.

## 1.2 The Wonderful Use of Netflix

Nostalgic or not, the songs are to be listened to, and the membership is impossible to charge 🐶

So, the witty (pinqiong) me began to look for free resources on all major platforms. I have to say, the Internet is really a great invention, as long as the electricity is connected to the Internet, there is no resource that can not be found.

Especially the net disk system, really a good hand for resource sharing!

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/048c70c470f5435a938bcd9d338f377f%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

I'm sure you've all used a netbook, from storing photos to sharing work documents, it has become an integral part of our lives.

But have you ever wondered what kind of system design is behind to support these functions? Today, let's explore the architecture design of a Netdisk system.

# 2. Web Disk System

Baidu.com is a popular cloud storage and file sharing platform with over 800 million users and 10w+PB of storage capacity.

In this article, we will delve into the core features of the Baidu Netdisk system and how to deal with the challenges that may arise from high concurrency and massive storage.

## 2.1 Architecture Overview

The system design of Baidu.com Disk adopts a distributed architecture to cope with the huge number of users and massive storage demand. The core components include:

1. **Client Layer**: for receiving and distributing user requests from different devices, splitting and assembling file resources, and interacting directly with back-end services.
2. **Application microservices**: handle core business logic, such as file upload and download, file sharing, permission control, VIP speed limit, etc.
3. **Relational DB system**: used for persistent storage of user files and metadata, as well as basic information such as user rights.
4. **Message Queue**: asynchronous peak shaving decoupling, improve write performance, reduce database load and the pressure of frequent communication between applications.
5. **Registry and Cache**: Application nodes regularly report the IP nodes and ports of servers to the registry so that other servers can call them in real time. The cache can store authentication information such as Token or application hotspot data.
6. **Distributed File System**: Used to store unstructured files, such as pictures, audio, video and other data, with high storage efficiency and good scalability.

## 2.2 Challenges of Billions of Users

For a storage system like NetDisk, a large amount of data is generated and transmitted every day.

Taking Baidu NetDisk as an example, the number of users has exceeded 800 million by the end of 2022, and the storage capacity has long exceeded 10w+PB (i.e., 100+ billion GB).

Therefore, designing an online disk system has the following challenges.

### Large storage volume

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/e917a61b4f164279b41635269807fc00%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

Baidu.com currently has more than 800 million users, the average user's maximum storage capacity is 1 TB, the storage capacity is calculated on the basis of 100 billion gigabytes (GB), each user is almost 100+GB [1000GB/8] of storage, and the average utilization rate of the storage space is 10%.

### High throughput

The daily activity of Baidu.com is 200 million, and the proportion of daily active users is about 25%, and each user visits the disk 4 times on average.

Therefore, the QPS of Baidu.com is about 10,000 [200 million users _4 times / (24_3600 seconds)], and the peak period is twice the average QPS, i.e. 20,000

### Network bandwidth

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/03dc3aff4380446e91d55706729df0b4%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

Assuming that the average file size of each user's last download is 2MB, the network bandwidth load is about 18GB/s (200 million _4_2M/(24 _3600_1024G)), i.e., 144Gb/s. The peak traffic bandwidth is about 2 times the average, which is about 288Gb/s.

## 2.3 Functional Requirements

The common functions are as follows:

1. support users to register and log in to NDN, open VIP, and log out of the account. 2. upload files and download files.
2. Upload and download files. 3.
3. add friends and share files among friends. 4. add, modify and delete storage accounts.
4. add, modify and delete storage directories. 5.
5. Rename file data or delete unwanted files. 6.
6. Allow to send files to friends, or share files to strangers via links.

## 2.4 Non-Functional Requirements

The following requirements are needed for the current design of the web hosting system:

1. massive data storage: 800 million registered users, about 25% of active users, 100 million TB of space.
2. high concurrent access: average 10,000 QPS, peak 20,000 QPS. 3. high traffic load: average network traffic, average network traffic, average network traffic, average network traffic, average network traffic, average network traffic.
3. High traffic load: average network bandwidth 144Gb/s, peak 280Gb/s. 4. Highly reliable storage: files cannot be stored on the network.
4. Highly reliable storage: files cannot be lost, and the reliability of persistent storage reaches 6 9s, i.e. 1 million files are lost or damaged at most 1 file.
5. Highly available service: Users can upload normally, and the availability of download function reaches 4 9s, i.e., it is unavailable for at most 53 minutes (365 _24_60*0.0001) a year.
6. Privilege Control: Files need to be stored in isolation, except for the user's own and shared files, the rest of the files can not be seen by others.

# 3. Core Functions

## 3.1 File Upload and Download

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/bd2752a4449447de8e059b1aa20aa17f%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

#### File Upload

Users upload files through the Nethub client or web interface. After the upload request passes through the client application layer, in order to ensure the reliability of large file uploads, we can slice and upload files according to their sizes.

Then the client calls the application microservice to process the file basic data (metadata) and file content, and asynchronously upload the metadata and file content data respectively.

#### File Download

When a user requests to download a file, the client layer sends the request to the application microservice.

In order to increase the download speed, file blocks can be downloaded concurrently from the server, and then the files are assembled on the client side and returned to the user's device.

## 3.2 File Sharing

#### Friends Sharing

Users can share files or folders with their friends. When sharing, they can specify read-only or storage permissions for their friends and the time period for file sharing.

* Read-only permission: When a friend receives read-only permission, he/she can only view the contents of the file or folder, but cannot save, modify or delete the file.
* Dump permission: When a friend receives storage permission, he/she can choose to dump the file to his/her own storage space within the time limit, and he/she can share the file again.

#### Link Sharing

Users can share files or folders via links. The permission to share via **links is dumping permission** by default, and the sharing scope can be set as public, private or restricted to specific users.

* Public scope: Anyone can access the file or folder, and can dump the file to their own storage space.
* Private Scope: Generate a link to facilitate opening the file, only the user himself can access it.
* Specific user scope: allow the user's friends or specified to share to someone, when other people open the link shows no permission to access.

# 4. Detailed Design

## 4.1 File storage and metadata management

### Separate storage

Since relational databases like MySQL are not suitable for storing large data files, and file systems like HDFS and Ceph are very slow in data query.

So we divide file data into metadata and file content and store them separately, where:

* Metadata: including file owner, file permissions, file type, sharing information and other basic information, stored in the relational database MySQL inside.
* File content: the specific information of the file, such as pictures, audio, video and other multimedia data, saved in the object storage service, such as Ceph distributed object storage server.

The system responsible for responding to requests for metadata and file content is also divided into two systems, File Metadata Management (FMM) and File Content Management (FCM).

The architecture diagram is as follows:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/73381891e1e6412a9b117f632582f24e%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

Since user files may include large files such as video, audio, etc., but Ceph is not suitable for storing too large files, we split the uploaded file content and divide the large file into many small blocks in order to better upload and download large files.

This also has the advantage that when large files are uploaded and downloaded in blocks, these blocks can be processed concurrently and then assembled in the SDK to speed up the file transfer. Moreover, when the user's network is disconnected, we only need to re-transmit the remaining file blocks, thus realizing the function of resuming uploading after a break.

### File upload

The sequence of file upload is as follows:

! [](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/0470423387444ea188ce4cd887a284cd%7Etplv-k3u1fbpfcp-jj-mark%3A3024% 3A0%3A0%3A0%3Aq75.awebp)

After the user uploads a file, the client application divides the file into blocks according to the size of the file uploaded by the user, assuming that one block is generated for every 8M, and then uploads the corresponding MD5 value information of the block to the metadata management system (FMM).

The FMM determines whether there are duplicates in the MD5 values from the list of uploaded blocks. If they are new MD5 file blocks, they are assigned ids and stored in each file metadata table.

The FMM then generates an access token and returns it to the client along with the list of blockIds and a list of available FMM servers.

When the client receives the response from the FMM, it compares the MD5 values and determines which blocks of files need to be uploaded. Then, with the Token, the block IDs and the contents of the blocks to be uploaded, the client passes them into the available nodes of the FMM and stores the real blocks in the object storage system Ceph.

When a client requests FCM with a list of blockIds, FCM needs to call FMM again for user authentication to ensure that the blockIds are from FMM and not forged by the user.

However, for the sake of overall architectural simplicity, we use cached tokens instead of internal API calls, which on one hand reduces system interactions and on the other hand improves the overall response speed.

### File Download

The file download timeline is as follows:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/ae6293eb1345438f85e4cb685b46a07c%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

When a user downloads a file, the client passes in the file name, user, and other information to get the metadata of the file from the FMM.

Then, the FMM server queries MySQL for the blockId list of the corresponding user's file, obtains the list of accessible FMM servers from ZK, generates an access token from Redis, and returns it to the client.

Based on the server list of FCM and the blockId list, the client calls the FCM server to download the file blocks concurrently. After downloading all the file blocks, the client assembles the file blocks into a complete file and returns it to the user's device.

### Table Design

1. **User table**: record user key information, ID, user name, cell phone number, used space, user type (VIP, civilian), etc. 2. **File table**: record user key information, ID, user name, cell phone number, used space, user type, etc. 3.
2. **File table**: record file metadata information, store the tree structure of files, including file ID, name, user, parent file ID, number of child files, creation time, file size, etc. 3. **File_block table**: record user key information, ID, user name, cell phone number, used space, user type (VIP, civilian, etc.).
3. **File_block table**: records the specific information of the file block, ID, file ID, MD5 value of the file block, and so on.

### Upload and download speed limit

When designing the web disk, considering the relatively large number of users and storage volume of the system, we add both the application system and storage servers into the cluster, and integrate load balancing, service gateway and other infrastructures in order to provide the ability of failover, high availability, and elastic scaling.

And based on the cost considerations of memory and network bandwidth, we can't just add machines to ensure the upload and download rate of users, and based on commercialization considerations, we can limit the speed of ordinary users who are not members.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/fcda5736b8594a5b9e4161a85a7a07c8%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

The implementation is as follows: when a client requests the FMM system to perform an upload or download task, we first get the user type of the user, and if it is a civilian user, we can reduce the number of servers appropriately when returning the list of available FCM nodes to the client.

For example, a VIP user can enjoy 50 servers for uploading and downloading at the same time, while a common user is only assigned 5 servers for uploading and downloading files.

## 4.2 File Sharing

### RBAC Privilege Control

Since the file sharing of NDN can be modified in real time, we adopt the idea of RBAC (Role-Based Access Control) to control the user's access to the files.

The permission-related tables are designed as follows:

1. **User table**: stores information about system users, as above, including user ID, user name, etc.
2. **Role table**: defines the roles in the system, each role includes role ID, role name and so on. Common roles are Supervisor, Normal User, Buddy, Read-Only User, Restricted User, and so on.
3. **UserRole table**: establishes the association between users and roles, records which users have which roles, including user ID and role ID. 4. **File table**: defines the roles in the system.
4. **File table**: represents the file metadata information in the system, as above, including file ID, file name, etc. 5. **Permission table**: represents the file metadata information in the system, including file ID, file name, etc. 5.
5. **Permission table**: defines the permissions of roles on resources, including permission ID, role ID, user ID, file ID, expiration time and so on.

Through the mechanism of RBAC, we can easily manage users' permissions on resources, assign permissions according to roles, and reclaim permissions when needed.

### Share a file with a friend

With RBAC for permission control, the business process of registering an account and uploading files to share with friends is as follows:

1. **User registration and login**:
   2.

  - Assign roles to users and insert the related records into the UserRole table, which is initially an ordinary user role that can upload, download, share and manage their files.
  - The user creates an account through the registration function and its information is stored in the User table.

1. **Creating and sharing files**:
   2.

  - Users can create files or folders, and information about these resources is stored in the File table.
  - When a user wishes to share a file, he or she can choose to specify who to share it with (other users or friends) and give the file permissions.

1. **Permission Assignment**:
   2.

  - The file owner grants permissions to a specific buddy by inserting the buddy's user ID, what role they are assigned, and the corresponding file ID into the Permission table.
  - For example, the File Owner role can have full access, while the Buddy role can have Dump permission and Continue Sharing permission, and the Read-Only role can have access only. 1. **File Access**: 2.

1. **File Access**:
   2.

  - When a user tries to access a file, the system checks the user's own role permissions [to determine if the user is an offending user, or a restricted user], as well as file-related permissions. If the user's role has file-specific permissions, the user's role can be used to access the file.
  - If the user's role has file-specific permissions (for example, read or write permissions), the user is allowed to access the file.

### Link sharing

The overall process is similar to buddy sharing, the only difference is that when inserting a record into the Permission table, you can set the permissions of the file to public access, the corresponding user is set to NULL, and all users can access and dump the file by default.

```sql
insert into permission (file_id, role_id, user_id) values ('被共享的文件ID', '公开角色的ID', NULL)
```

In this way, when the user accesses the file and determines that the file's permissions are public access, he or she can access or dump the shared file.

### Privilege reclamation

When a resource owner or administrator decides to reclaim a user's or role's permissions on a resource, the system deletes the related permission records.

The implementation is to add a new expiration time field in the Permission table, so that when a user shares a file with a friend or generates a link to share it, he/she needs to set a specific expiration time.

The file system can enable a timed task to clean up expired permissions periodically to ensure that files can only be accessed by users within the validity period.

> If you set a link to be accessible indefinitely, you can set the expiration time to some point in the past.

### File deletion

When user deletes a file, we first need to get the file block list via FMM's interface, then delete the metadata information to free up user's storage space, and at the same time transfer the deleted file block list to FCM via message queue to delete the file content.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/7f0029f7b91e47fead6b999c36f5043d%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

In order to ensure the transactional consistency of file metadata and file content, we adopt the idea of **maximum effort notification** in distributed transactions.

It is implemented as follows: a new monitoring and alerting system is added so that when file content deletion fails, SMS or email can be used to notify the administrator to manually handle the unsynchronized data.

To the user, the file metadata is no longer visible, so the file content and file metadata only need to ensure eventual consistency.

# 5. Summary

Currently, the Internet market is highly competitive, and the netdisk field is no exception.

There are many domestic netdisk providers, such as Baidu.com, Tencent Micro Cloud, 360 Cloud Disk, etc. However, Baidu.com is still a dominant player, with more than 80% of the market share.
