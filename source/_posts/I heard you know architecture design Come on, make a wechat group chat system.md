---
title: I heard you know architecture design? Come on, make a wechat group chat system
date: 2023-11-06 19:04:00
categories: 
  - Backend
tags: 
  - Backend Technology Sharing
  - Architecture
  - recognize
  - development
  - framework
  - strangers
  - WeChat
description: Grab the red envelopes! I'm sure most of you are no strangers to this, so how is this group chat system of WeChat designed to make it easy for us to chat, share pictures and emoticons, and that magical Red Packet feature?？
cover: https://raw.githubusercontent.com/zqwuming/blogimage/img/img/2023-11-07_222844.png
---
## 1. Introduction

As I was holding my cell phone the other day, chatting freely with my friends' WeChat group about gossip news and upcoming weekend plans, suddenly a message with a joyful message came to me with eight big words written right in the middle of the message: **Congratulations on your fortune, great luck**.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/2023-11-07_222844.png)

Grab the red envelopes! I'm sure most of you are no strangers to this, so how is this group chat system of WeChat designed to make it easy for us to chat, share pictures and emoticons, and that magical red packet feature?

This question has been bothering me for a long time, so I decided to dig a little deeper and see how the design behind WeChat's group chat system works.

### WeChat Group Chat System Design

WeChat, as a universal App with 1 billion users, must have been used by all of you. The WeChat group building function is a core ability inside WeChat, which can put hundreds of friends or strangers into a group space.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/2023-11-07_222954.png)

Maybe you've experienced group chat on WeChat many times, but have you ever wondered how the system behind this is designed?

Let's explore it today.

## 2. System Requirements

## 2.1 System Features and Functional Requirements

WeChat Group Chat is one of the core features of the social application, which allows users to create their own social circles to communicate with family members, friends, or enthusiasts of common interests in a friendly way.

The following are the core features of WeChat group chat system:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/2e6101baee724a39986dfd722408992c%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

* **Create Group Chat**: Users can create new chat groups, invite other friend users to join or build groups with strangers face to face.
* **Group Management**: Group owners and administrators are able to manage group members, set rules and permissions.
* **Message Sending and Receiving**: Allows group members to send multiple types of messages such as text, image, audio, video, etc. and push them to all group members.
* **Real-time communication**: messages should be able to be delivered quickly to ensure real-time interaction.
* **Red Packet Grabbing**: Users can send any number and amount of red packets in the group chat, and group members can grab the red packets with random amount.

### 2.2 Non-functional Requirements: Coping with High Concurrency, High Performance, and Mass Storage

When we face the scenario that 1 billion WeChat users may use the group building function every day, we need to deal with large-scale user concurrency. This leads to the non-functional requirements of the system, including:

* **High Concurrency**: the system needs to support a large number of users creating and using groups simultaneously to ensure a latency-free user experience.
* **High performance**: fast messaging and instant response are key to digital socialization.
* **Massive Storage**: The system must be scalable to accommodate massive amounts of user-generated message text, images, and audio/video data.

## 3. Outline Design

In the outline design, we consider the outline design of the core components and basic business of the system.

## 3.1 Core Components

The following core components and protocols are involved in the WeChat group chat system.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/5d2d0557d4ee4c8ca4dacbe60c74e2f2%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

* **
* Client **: Receive messages from WeChat group chat on cell phone or PC and transmit them to the backend server in real time.
* **Websocket transfer protocol**: supports real-time interaction between the client and the backend server, with low overhead and high real-time performance, commonly used in WeChat, QQ and other IM systems communication systems
* **Long-connection cluster**: a cluster of systems that make long Websocket connections with clients and forward messages to application servers through middleware.
* **Message Processing Server Cluster**: provides real-time message processing capability, including data storage, query, and interaction with the database.

* **Message Push Server Cluster**: this is the relay station for messages and is responsible for delivering messages to the correct cluster members
* **Database server cluster**: it is used to store user text data, thumbnails of images, audio and video metadata, etc.
* **Distributed file storage cluster**: for storing user's pictures, audio and video files.

### 3.2 Business Outline Design

#### Group Chat Creation

* **Unique ID Assignment**: When a user requests to create a new group, the system generates a unique group ID, which can be generated by a distributed ID generator such as Snowflake, or by using a database incremental ID. here, we adopt MySQL's incremental ID for the sake of simplicity.
* **Group Information Storage**: Stores group IDs and related information (e.g. group name, creator ID, etc.) in the group database.
* **Member association**: add the group owner as the founding member of the group, and the creator will also become the administrator.
* **Message History**: To ensure that new members can access previous messages, the group ID of this new group is stored in association with user messages.

In addition to pulling friends to build groups, WeChat has also implemented the ability to build groups face-to-face.

## 4. Face-to-face group building

The user initiates a face-to-face group creation and inputs a 4-digit random code, and users around the group can join the group chat after inputting the random code. The face-to-face group creation function usually involves the following data table design and core business interaction processes.

### 4.1 Database table design

1. **User table**: store user information, including user ID, nickname, avatar, etc. 2. **Group table**: store user information, including user ID, nickname, avatar, etc. 3.
2. **Group table**: store group information, including group ID, group name, creator ID, number of group members, etc. 3. **GroupMember table**: store group information, including group ID, group name, creator ID, number of group members, etc. 4.
3. **GroupMember table**: associated with user and group, including user ID and group ID. 4. **RandomCode table**: associated with user and group, including user ID and group ID. 5.
4. **RandomCode table**: stores the random code for face-to-face group creation and the associated group ID.

### 4.2 Core Business Interaction Flow

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/f6498bf859a14790aa6f618f7c46dea2%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

User A initiates a face-to-face group in the mobile application, enters a random code, passes the verification, and waits for users in the surrounding area (within 50 meters) to join. At this time, the system stores the user information in the cache as a `HashMap` and sets the expiration time to `3min`.

```css
{random code, user list [User A (ID, name, avatar)]}
```

User B initiates a face-to-face group creation on another cell phone, inputs the specified random code, **if there is such a random code around the user, the user enters the same group chat waiting page, and can see the avatars and nicknames of other group members**.

At this point, in addition to obtaining all user information based on the random code, the system will also update the user information in the cache in real time.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/bd1da07dc8884f3fb9466255b13b48b4%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

When the first user clicks **Enter the group** to join the group chat, the system stores the generated random code in the `RandomCode` table and associates it with the newly created group ID to update the number of group members.

Then, the system stores the user information and the newly generated group chat information in the `Group&#x3001;GroupMember` table

#### member joins and refreshes the group member information

Later, when user B and C join the group chat with the random code, the mobile client sends a request to the server backend to verify whether the random code is valid. The server backend verifies the random code and checks whether the random code exists in the cache and whether it is within the validity period.

Then, it determines whether the current group members are full or not (at present, the maximum number of group chats created by ordinary users is 500). If the validation passes, the server back-end adds users B and C to the group member table `GroupMember` and returns a successful response.

When the mobile client application receives the success response, it updates the list of group chats for users B and C to show the new group chats they have joined.

#### Other Technical Components

In this way, user A successfully creates a face-to-face group by creating a random code and scanning the QR code by the surrounding users. This feature involves several technical components, including distributed caching, database, QR code generation and validation.

Meanwhile, a quite important capability in the face-to-face group building process is to identify the user's area, e.g., within 50 meters. This can be done by using **Redis' GeoHash algorithm to get information about all users within a range**.

Due to space limitations, here does not expand the details, and want to understand more and QR code generation and location algorithm of the details, you can see my previous article: [I heard that you will be architectural design? Come on, get a bus & subway ride system]. 

## 5. Message sending and receiving

When a member speaks in a WeChat group, the system needs to ** handle the distribution of the message, notify other members, and ensure that the message is displayed**. The following are the detailed interaction steps for this function, as well as the database storage scheme.

### 5.1 Interaction Flow

The message sending and receiving timing diagram is shown below:

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/914596ee246041e8b062f5d344f0ed08%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

1. User A sends a message with an image, video or audio to the group.
2. The mobile client application uploads the message content and media files to the server backend. 3.
3. The server backend receives the message and the media file, stores the message content in the Message table, and stores the media file in the distributed file storage cluster. **In the Message table, not only the MediaID of the media file is recorded to associate the message with the media, but also the thumbnail image, video cover image, and so on.
4. The server backend broadcasts the message to all cluster members. The mobile client application receives the message and loads the corresponding presentation based on the message type (text, image, video, audio).
5. when the user clicks to view the image, video or audio thumbnail, the client application fetches the corresponding media file path from the object storage cluster based on the `MediaID` and displays it to the user.

This process ensures that messages and media files are stored and displayed efficiently. Users can upload and view various types of media data, while the server backend realizes effective message storage and presentation by associating `Message` with information in the object storage server.

### 5.2 Message Storage and Presentation

Saving and displaying user's image, video or audio data in WeChat Groups usually requires the design of data storage and display. In addition to the user table and group table mentioned in the face-to-face group building function above, the following table structures are needed:

1. **Message table:** Used to store messages, each message has a unique MessageID, message type (text, picture, video, audio), message content (text, picture thumbnail, video cover image, etc.), sender UserID, receiver group GroupID, send time and other fields.
2. **Media table:** stores media data such as pictures, videos, audios uploaded by users. Each media file has a unique MediaID, file path, uploader UserID, upload time, and other fields. 3.
3. **MessageState table:** Used to store user's message state, including MessageID, UserID, whether it is read or not. When the message is pushed, the unread count is calculated through this table, and uniformly pushed to the user, and a small number representing the unread count of the message is displayed on the offline user's phone.

As we know, MySQL triggers a full table scan every time you query a `select count` type statement, so loading the unread count of messages is slow every time.

For query performance, we can store the user's message count into Redis and record an unread value in real time. And, when the unread count is greater than 99, the unread count is set to 100 and will not be increased.

When pushing user messages, ** as long as the unread count is 100, set the number of pushed messages to `99+` to improve storage performance and interaction efficiency. **

## 6. Grab Red Packet

Grab Red Envelope allows users to send any number and amount of red envelopes in a group chat, and group members can grab red envelopes with random amounts, but it is necessary to ** ensure that the amount of red envelopes for each user is not less than $0.01**.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/97d9a56d3b2843d597763bf8e88b1431%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

The detailed interaction flow of grabbing red packets is as follows:

1. the user receives the notification of grabbing the red packet, and clicks on the notification to open the group chat page.
2. The user clicks on the red packet, and the background service verifies the user's eligibility to ensure that the user has not yet received the red packet.
3. If the user's eligibility is verified, the background service allocates the red packet amount and stores the collection record.
4. The user sees the red packet amount in the WeChat group, and the red packet status is updated to "Claimed".
5. Asynchronously call the payment interface to update the red packet amount to the wallet.

The red packet function requires attention to the database design, real-time red packet grabbing and red packet distribution algorithm.

### 6.1 Database Design

The fields of the redpack table `redpack` are as follows:

* **id: ** primary key, redpack id
* **totalAmount: ** total amount
* **surplusAmount：*** remaining amount
* **total: ** total number of red packets
* **surplusTotal：*** Remaining red packets total amount
* **userId：*** The ID of the user who sent the red packets.

This table is used to record how many red packets a user has sent and the remaining amount to be maintained.

The redpack record table `redpack_record` is as follows:

* **id:** primary key, record ID
* **redpackId: ** redpackId, foreign key
* **userId: ** userId
* **amount: ** amount grabbed

The record table is used to store information about the specific redpacks grabbed by users, and is also a sub-table of the redpack table.

### 6.2 Real-time

#### 1. Send red packet

1. After the user sets the total amount and number of red packets, add a data in the red packet table and start sending red packets.
2. In order to ensure the real-time performance and the efficiency of grabbing red envelopes, add a record in Redis, .
3. Grab the red envelope message pushed to all group members

#### 2. Grab Red Envelope

Since 2015, WeChat Red Envelope's grabbing red envelopes and splitting red envelopes have been separated, and users need to perform two operations after clicking on grabbing red envelopes. That's why you can sometimes grab a red packet, but when you click on it, you'll find that the red packet has already been claimed**.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/0dd3c0ee8510446fa01d38ca3665f49d%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

The interaction steps of grabbing red envelopes are as follows. 1:

1. Grab the red packets: the grabbing operation is done in the `Redis` cache layer, ** through the atomic decrement operation to update the number of red packets **, after reaching 0, it means that all the red packets have been grabbed.
2. opening red packets: when opening red packets, the first real-time calculation of the amount of money, generally through the **two times the average method** (i.e., between 0.01 and 2 times the remaining average). 3. red packet records: the user to get the red packet records, the user to get the red packet record.
3. Red Packet Records: After the user obtains the amount of the red packet, the number and amount that have been received are totaled through the transaction operation of the database, and the red packet table and record table are updated.
4. transfer: in order to improve efficiency, the final **transfer is an asynchronous operation**, which is why during the Chinese New Year, the red packet cannot be seen in the balance immediately after receiving it.

### 6.3 Red Packet Allocation Algorithm

When the red packet amount is allocated, there are two implementation options: real-time splitting and pre-generation because it is randomly allocated.

#### 1. Real-time splitting

Real-time splitting refers to the process of real-time calculation of the amount of each red packet when ** grabbing red packets, in order to realize the process of red packet splitting.

This requires us to design a good splitting algorithm, so that the red packet splitting has been to ensure that the amount of the subsequent red packet to be split can not be empty.

When splitting in real time, it is not easy to do the split red packet amount obeys the **normal distribution** law.

#### 2. Pre-generation

Pre-generation, refers to the red packets ** before ** have completed the red packet ** amount of split **, grab the red packet is only taken out in order to split the amount of red packets.

This way of splitting algorithm requirements are lower, you can split the red packet amount of randomness is very good, but usually need to be combined with the use of queues, and the need to design a table to store the split amount of red packets.

#### 3. Twice the mean method

Considering the advantages and disadvantages of the above, as well as the number of people in the WeChat group chat is not large (currently up to 500 people), so we use real-time splitting, using **Double Mean Method** to generate random red envelopes, only to meet the random can be, do not need to be normally distributed.

> Therefore, there may be a large difference between the red packets, but it is more exciting, isn't it 🐶.

Random numbers generated using the doubled mean method will have a random amount between `0.01 ~ 2` each time.

Assuming that the remaining amount of the current red packet is $10 and the number of remaining red packets is 5, `10/5 = 2`, then the amount of red packets that the current user can grab is: `0.01 ~ 4` yuan.

#### 4. Algorithm Optimization

Although the random red packets generated by the twofold mean method are close to the average, I have previously seen a similar statement on a forum: **The randomness of the WeChat red packet amount has a relationship with the timing of receiving it, especially if the amount is not high**.

So, Xiao ❤ spent a huge amount of money to send multiple red envelopes in the WeChat group, and came to the conclusion that after sending 4 red envelopes totaling 0.05, the amount of red envelopes received by the last person must be `0.02`.

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/c2a71bd0cf994b088b5e6367855f2553%7Etplv-k3u1fbpfcp-jj-mark%3A3024%3A0%3A0%3A0%3Aq75.awebp)

No exceptions:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa4ffe2ce88a4361ac9e7b1cc562a13d~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1080&h=712&s=164056&e=png&b=fdfdfd)

So, the probability is that the red packet amount algorithm is not randomly assigned, but has been processed before handing out the red packets. For example, before the red packet amount is generated, it is first generated into a non-existent red packet, which totals 

And when the red packet amount is assigned, `0.01` is added to the random value base of each red packet as a way to ensure that the minimum value of each red packet is not 0.

So, suppose the user sends out 3 red envelopes with a total amount of 0.04, he needs to first extract `3*0.01` to the "fourth" non-existent red envelope, so the random value of the red envelope grabbed by the first person is `0 ~ (0.04-3*0.01)/3`.

The quotient of the divisor is taken down to two decimals, `0 ~ (0.04-3*0.01)/3 ==> (0 ~ 0) = 0`, plus the previously extracted guaranteed value of `0.01`, because of the concern about red packet overruns, so the first two grabbed red packet amounts are both `0.01`. The amount of the last red packet is the red packet balance, i.e. `0.02`.

The algorithm logic is implemented in Go as follows:

```go
import (
   "fmt"
   "math"
   "math/rand"
   "strconv"
)

type RedPack struct {

    SurplusAmount float64 

    SurplusTotal int 

}

func remainTwoDecimal(num float64) float64 {

    numStr := strconv.FormatFloat(num, 'f', 2, 64)

    num, _ = strconv.ParseFloat(numStr, 64)
    return num
}

func getRandomRedPack(rp *RedPack) float64 {
    if rp.SurplusTotal 0 {

        return 0
    }

    if rp.SurplusTotal == 1 {
        return remainTwoDecimal(rp.SurplusAmount + 0.01)
    }

    avgAmount := math.Floor(100*(rp.SurplusAmount/float64(rp.SurplusTotal))) / float64(100)
    avgAmount = remainTwoDecimal(avgAmount)

    rand.NewSource(time.Now().UnixNano())

    var max float64
    if avgAmount > 0 {
        max = 2*avgAmount - 0.01
    } else {
        max = 0
    }
    money := remainTwoDecimal(rand.Float64()*(max) + 0.01)

    rp.SurplusTotal -= 1
    rp.SurplusAmount = remainTwoDecimal(rp.SurplusAmount + 0.01 - money)

    return money
}

func main() {
    rp := &RedPack{
        SurplusAmount: 0.06,
        SurplusTotal:  5,

    }

    rp.SurplusAmount -= 0.01 * float64(rp.SurplusTotal)
    total := rp.SurplusTotal
    for i := 0; i 
```

Print results:

> 0.01、0.01、0.01、0.01、0.02

As expected!

## 7. Summary

Behind WeChat's group chat and red packet grabbing features lie complex interaction techniques and well-designed product experiences. Through these core components, database tables, and detailed interaction processes, users are able to easily participate and enjoy the convenience of the group chat system.

And, the addition of these fun-filled features is one of the reasons why WeChat has so many users, right?

The system design of WeChat's group building feature is not just a display of technological magnificence, it's part of the magic of digital socialization.

