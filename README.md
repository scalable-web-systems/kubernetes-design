# kubernetes-design

High level topics: Databases, Caching, some basics(CAP Theorem)


## Introduction 

When it comes to building scalable systems, kubernetes is a great tool and a quite a popular one. However, there are many fundamental concepts that one needs to think about from a system design perspective when thinking about scalable systems in the context of kubernetes. This tutorial hopes to cover some of the fundamental concepts such as databases, caching, the CAP theorem. 


## The CAP Theorem

CAP stands for:
```
CAP: Consistency, Availability, Partition Tolerance
```

Let us define these terms:
- Consistency: any read operation that begins after a write operation completes must return that value, or the result of a later write operation
- Availability: every request received by a non-failing node in the system must result in a response
- Partition Tolerance: the network will be allowed to lose arbitrarily many messages sent from one node to another

The theorem:

```
The CAP theorem states that a distributed system cannot simultaneously be consistent, available, and partition tolerant. 
```

An [illustrated proof](https://mwhittaker.github.io/blog/an_illustrated_proof_of_the_cap_theorem/) of the theorem. 

An Example: 

In order to get both availability and partition tolerance, you have to give up consistency. Consider if you have two nodes, X and Y. Now, there is a break between network communication between X and Y, so they can't sync updates. At this point you can either:

A) Allow the nodes to get out of sync (giving up consistency), or

B) Consider the cluster to be "down" (giving up availability)

The different combinations in CAP:


- CA - data is consistent between all nodes - as long as all nodes are online - and you can read/write from any node and be sure that the data is the same, but if you ever develop a partition between nodes, the data will be out of sync (and won't re-sync once the partition is resolved).
- CP - data is consistent between all nodes, and maintains partition tolerance (preventing data desync) by becoming unavailable when a node goes down.
- AP - nodes remain online even if they can't communicate with each other and will resync data once the partition is resolved, but you aren't guaranteed that all nodes will have the same data (either during or after the partition)


An image displaying the proof:

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/f87b89bc-d7f8-40b8-bddd-f6c017499851/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20211109%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211109T161947Z&X-Amz-Expires=86400&X-Amz-Signature=a1265c4de4a2c219f915deed547cecf5152d5986a9dc218f00ad8c5c757a85c1&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)


**Important Note**: CAP is a spectrum. What that means is that it is not a hard and fast rule that only 2 out of the 3 will work but rather it is a theorem to think about tradeoffs. Certain transactions require high availability some require high consistency. It helps a system designer to think about trade offs, optimize for these 3 important constraints. 

## Single Point of Failure (SPOF)

In any scalable system, we must avoid single point of failures. As the name suggests, it is a a node,(it could be any part of the system) if it fails the whole system would go down. 

As an example, this image displays how a single load balancer could be an SPOF. 

![](https://avinetworks.com/wp-content/uploads/2020/09/single-point-of-failure-diagram.png)

As the image suggests, having multiple load balancers is the solution. 

To mitigate SPOFs we have 3 key approaches:

- More Nodes: With more nodes one can duplicate the node or service and distribute traffic among them. Another way is to use the secondary node/nodes as a backup service, though that would potentially lead to wasting resources. 
- Master Slave Approach: This approach is quite useful especially in the context of databases. The main question here is what if a database goes down. This can be mitigated by having a master database off which slaves replicate. So in the case master goes down, one of the slaves can become a master. There can be additional complexity such as having multiple masters or having read only slaves/write only slaves. 
- Multiple Regions: Having all resources in one region can also mean a SPOF. For instance, all your services in the cloud are in the US east region. To avoid, infrastructure going down in one area or a natural disaster happenning, having services and resources split across multiple regions is a good practice.

An Example Diagram of master slave approach:

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/c6359d96-393a-4e7d-873f-6a8b143d01a0/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20211109%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211109T180743Z&X-Amz-Expires=86400&X-Amz-Signature=45721f14d8517f0973634ab62f6907a8191755705b529311479a632c46793dd2&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)


## Databases 

The following are some key concepts in databases. 

### ACID Compliance

**Atomicity**: Database transactions, like atoms, can be broken 
down into smaller parts. When it comes to your database, atomicity 
refers to the integrity of the entire database transaction, not just a 
component of it. In other words, if one part of a transaction doesn’t 
work like it’s supposed to, the other will fail as a result—and vice 
versa. For example, if you’re shopping on an e-commerce site, you must 
have an item in your cart in order to pay for it. What you can’t do is 
pay for something that’s not in your cart. (You can add something into 
your cart and not pay for it, but that database transaction won’t be 
complete, and thus not ‘atomic’, until you pay for it).

**Consistency**: For any database to operate as it’s intended to 
operate, it must follow the appropriate data validation rules. Thus, 
consistency means that only data which follows those rules is permitted 
to be written to the database. If a transaction occurs and results in 
data that does not follow the rules of the database, it will be ‘rolled 
back’ to a previous iteration of itself (or ‘state’) which complies with
 the rules. On the other hand, following a successful transaction, new 
data will be added to the database and the resulting state will be 
consistent with existing rules.

**Isolation**: It’s safe to say that at any given time on Amazon, 
there is far more than one transaction occurring on the platform. In 
fact, an incredibly huge amount of database transactions are occurring 
simultaneously. For a database, isolation refers to the ability to 
concurrently process multiple transactions in a way that one does not 
affect another. So, imagine you and your neighbor are both trying to buy
 something from the same e-commerce platform at the same time. There are
 10 items for sale: your neighbor wants five and you want six. Isolation
 means that one of those transactions would be completed ahead of the 
other one. In other words, if your neighbor clicked first, they will get
 five items, and only five items will be remaining in stock. So you will
 only get to buy five items. If you clicked first, you will get the six 
items you want, and they will only get four. Thus, isolation ensures 
that eleven items aren’t sold when only ten exist.

**Durability**: All technology fails from time to time… the goal 
is to make those failures invisible to the end-user. In databases that 
possess durability, data is saved once a transaction is completed, even 
if a power outage or system failure occurs. Imagine you’re buying 
in-demand concert tickets on a site similar to Ticketmaster.com. Right 
when tickets go on sale, you’re ready to make a purchase. After being 
stuck in the digital waiting room for some time, you’re finally able to 
add those tickets to your cart. You then make the purchase and get your 
confirmation. However if that database lacks durability, even after your
 ticket purchase was confirmed, if the database suffers a failure 
incident your transaction would still be lost! As you might expect, this
 is a really bad thing to happen for an online e-commerce site, so 
transaction durability is a must-have.

### Database Sharding 

Database sharding is a popular concept when dealing with high volume data and is a very popular way to manage in No-SQL databases. Shards are autonomous and is an example to scale horizontally. Shards are basically partitions of the database based on a key. This key is what decides the shards, it could be a column value, it could be location or user ID or any other attribute that makes sense and works universally on the incoming data. Each shard can be protected by the master slave architecture for further scalability. 

An example of shards:

![](https://assets.digitalocean.com/articles/understanding_sharding/DB_image_3_cropped.png)


## Caching

Caching is a way to speed up retrieval of frequently or commonly accessed data. This means storing data in something like Redis that helps retrieve data very quickly. 

### Advantages of Caching 
- Reduce network calls
- Reduce recomputations
- Reduce DB load

### Caching Policies

Caching policies are used to delete unnecessary cache values to make sure the cache is up to date and always fresh and relevant to the incoming requests. 

- LRU Cache - Least Recently Used
- LFU Cache - Least Frequently Used Cache
- Sliding window

### Updating the Cache


- Cache Aside
- Write Through
- Write Behind
- Refresh Ahead

## Storing Images 

| Syntax      | Description |
| ----------- | ----------- |
| - Header      | Title       |
| - Paragraph   | Text        |

## Acknowledgements 

- [CAP](https://stackoverflow.com/questions/12346326/cap-theorem-availability-and-partition-tolerance)
- [Updating cache](https://dev.to/vishnuchilamakuru/4-ways-to-update-your-cache-jmg)
