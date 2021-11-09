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

Diagram of master slave approach:

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/c6359d96-393a-4e7d-873f-6a8b143d01a0/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20211109%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211109T180743Z&X-Amz-Expires=86400&X-Amz-Signature=45721f14d8517f0973634ab62f6907a8191755705b529311479a632c46793dd2&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)


## Databases 



## Acknowledgements 

- [CAP](https://stackoverflow.com/questions/12346326/cap-theorem-availability-and-partition-tolerance)