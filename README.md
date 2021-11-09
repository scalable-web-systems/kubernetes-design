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



![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/f87b89bc-d7f8-40b8-bddd-f6c017499851/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20211109%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211109T161947Z&X-Amz-Expires=86400&X-Amz-Signature=a1265c4de4a2c219f915deed547cecf5152d5986a9dc218f00ad8c5c757a85c1&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

## Acknowledgements 
