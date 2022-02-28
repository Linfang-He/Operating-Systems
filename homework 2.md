# CS 221 hw2
###### tags: `UCSD`

Question 1
---
a) What kinds of thread workloads (mix of reader and writer threads) work particularly well with this kind of synchronization?
- With this kind of synchronization, reader threads work paricularly well. Fort reader threads, just reading the data structure do not acquire a lock and are not blocked by other threads. They happily use the value of that pointer to read data. No copy, not blocking, no lock waiting, so reader threads work pretty well with RCU synchronization.

b) Does this kind of synchronization work equally well for all kinds of data structures, or does it work better for some than others?
- The RCU locks is more suitable for synchronizing pointer-based data structures, such as lists and hash tables. For other data structures, the copy operation is more time and space consuming.

c) Can readers starve other readers?
- No, readers will not affect each other.

d) Can writers starve readers?
- No, writers would keep a copy and operate on that private copy, while readers read the original shared version of the data structure. They will not affect each other.

e) Can readers starve writers?
- No, similar as above, readers read the original shared version, while writers would keep a copy and operate on that private version. They will not affect each other.

f) Can writers starve writers?
- Yes. For multiple writers, there may be endless aborting for one writer and this writer is then starved. That is, one writer is always slower than other writers when updating the reference, thus needs to abort every time and never succeeds.

g) Can deadlock occur?
- No. For readers, they do not change the data structure. For writers, they keep their private copy and update on it, just need to abort if conflict happens. There is no need for waiting anyone, so there will be no deadlock.

h) Is it safe for a thread to be arbitrarily suspended or stopped while it is accessing a shared data structure?
- It is safe. As mentioned above, there will be no waiting between threads. So even if a thread is suspended or stopped while it is accessing a shared data structure, it will not affect other thread.


Question 2
---
a) Identify the various protection domains in the system for this scenario. Which domains are privileged, and which are unprivileged?

* Exokernel: The exokernel, the libOS and the web server process are the protection domains. Exokernel is privileged, the process is unprivileged.

* L4: The L4 microkernel and the web server process are the protection domains. L4 microkernel is privileged, the process is unprivileged.

* Xen: The protection domian are Xen hypervisor (domain0) and the guest OS (domainU) running the web server process. Only Xen can execute at a sufficiently privileged level, and guest OS must run at a lower privilege level than Xen. So, Xen hypervisor (domian 0) running control software are privileged. The guest OS (domainU) including the server process is unprivileged.

b) Describe the journey of the packet as a sequence of steps through the protection domains identified above. For each protection domain crossing, state the communication mechanism used for that packet to cross protection domains.

* Exokernel: When a packet is received, Exokernel use packet filter to determine what application it is for in access time, then performs an upcall to the application level (web server). 

* L4: When a packet is received, the inter-machine IPC is automatically redirected to the network server. That is, the IO hardware interrupts the L4 kernel, the packet is then tranfered from L4 microkernel to L4 Linux, then to the web server process by IPC message.

* Xen: When a packet is received, Xen determines the destination VIF(virtual network interface), and exchanges the packet buffer for a page frame on the relevant receive ring. The guest OS exchanges an unused page frame for each packet it receives. That is, Xen triggers an interrupt to domain0, and domain0 triggers an interrupt to domainU to the web server process.

c) Argue which of these systems will likely provide the highest performance Web service without violating protection.
* Exokernel is likely to provide the highest performance web service. It exposes hardware, allocation, names and revocation. Traditional abstractions are implemented at the application level, which increases the flexibility of application builders and advantages of domain-specific optimizations. Also, the procedure call replaces system call, thus decreasing the overhead. In this senario, there is no need to copy and transfer the packet data, and no overwhelming general abstractions and system calls.


d) Further consider the Web server process triggering a page fault on a page in its address space. As with the network packet, trace the propagation of the page fault through protection domains. Which domain handles the page fault? Whose pool of physical memory is used to satisfy the page fault?

* Exokernel: The privileged domain Exokernel handles the page fault. Similar to standard monolithic Linux, the CPU would raise an interrupt, halting the Web server process, and vector to the Exokernel. The Exokernel performs an upcall to the application level (web server), the page fault handler would allocate a physical page from free physical pages and update the page table entry with the valid mapping. The Exokernel would then return from the interrupt. 


* L4: The privileged domain L4 kernel handles the page fault. L4 sends an IPC to the pager thread, and the pager thread in user level OS resolves page faults by allocating memory and mapping memory to the page fault location, then responds to the L4 kernel and the process.

* Xen: The privileged domain Xen hypervisor handles the page fault. The guest OS triggers an exception of page fault. The page fault handler delivers faults via Xen. Xen’s handler creates a copy of the exception stack frame on the guest OS stack and returns control to the appropriate registered handler. The handler would allocate a free physical page and update the page table entry with the valid mapping. Then the guest OS would return from the interrupt.



Question 3
---
a) First argue why thin clients were seen as a compelling computing platform, and justify your argument with supporting claims.

- Easy Distribution
  Thin client has a big advantage that it enables making changes to the application without having to push software to every desktop that uses it.
  
- Less Expensive Terminals
  Thin client applications tend to have much of their complex business logic on the remote server, because the thin client software is not capable of running such logic. This means the terminals can be less powerful and less expensive, because they do not need to run complicated applications such as transactions interacting with a database.

- More Secure Terminals
  Thin clients are protected from the use of unauthorized software or the introduction of viruses because data cannot be saved to a thin client. "They have a simple, locked-down operating system that’s optimised to run on either Linux or Windows Embedded (WES7/WE8). And they’re very secure – the built-in write-protection of Windows Embedded will crush any threats by cycling the power and the filter stops anything being transferred to the storage." (refer to [The Benefits of a Thin Client](https://www.escape-technology.com/news/the-benefits-of-a-thin-client))
  
- Easy Rescaling
  The remote server keeps the data and storage. It is very simple to add a thin client. For clients, there is no need to install dozens of softwares, and no need to copy large data. Therefore, it is very easy to scale up and down for thin clients.


b) Then argue why thin clients have not been widely adopted as a platform, also justifying your argument.
- Longer Response Time
  The thin client leaves the majority of the logic on the server, and it must call that server for any change. Even the operation is simple, the client has to require a trip to the server and back. The response time includes waiting for the data to be sent to the server, reviewed and then sent back, rather than running locally, thus become longer.

- Less Robust Transactional Support
  A thin client does not maintain a permanent link to the server and then to the database. However, a thick client can make and maintain a connection, so if something happens to the transaction, the thick client can recover much more simpler. For thin clients, the connection is dropped and neither knows the current status of the data, thus the recovery is more complicated.
  
- Higher Demand for the Server
  All the logic from thin clients are done on the server, so there is high demand on the server resource and the server must be robust enough to handle these requests.

- Depend heavily on the Network
  For every logic, the thin client cannot operate locally. It needs to send data to and wait for response from the remote server. Once the network fails, then the thin client fails and cannot operate even very simple logic. So, thin clients depend on the network heavily.
  
c) Which factors in favor of thin clients are still valid today?
- Easy rescaling and cheap terminals are still valid today. There are still huge demands for cheap terminals, and these cheap terminals cannot do large operations. It is a good solution to use thin clients and push heavy computation to the server. For end terminals, it is also important to make it easily scale up.
