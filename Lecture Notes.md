# CSE 221 Operating Systems

###### tags: `UCSD`

Overall
---
Key point: High level
* the high level goal
* the approach it takes
* the problem it trying to solve and the solution



## Introduction
* What is an OS?
    * layer between apps/hardware
    * makes writing apps easier (abstract hardware resources)
    * manage resources / divide resources (when who ...)
        * "transaction" in resource abstraction
    * err/interrupt handler -- asynchronous operation
    * protection
    * toolset --> abstractions
    * illusions (larger than it has, virtual memory paging)
    * user-kernel boundary
* Part of OS?
    * windowing system? 
        * Y: managing screen
        * N: devices operate
    * web browser
        * Y: deal with different pages, can have OS in browser
        * N: install like an application
    * network stack
        * abstraction for net card
            * raw Ethernet <--> sockets
        * both 
            * TCP/IP --> OS
            * HTTP --> user level, application
            * resource management, how much bandwidth allocated --> OS

* How different are OSes
    * file systems --> abstractions
    * manage resources --> schedulers
    * syscall API --> abstractions
    * how easy is it to do the same task / porting apps
    * architecture
    * backwards compatibility

THE
---
Prove whether or not the system is deadlock-free
- Goal
    - Software engineering
    - Design contribution
    - Efficient use of hardware resources
    - Turn-around time: job queue, when done, user get answer
    - Paging: level of indirection, to page things to disk
    - Unusual: Prove correctness
        - Debugging is hard
    - **Verifying the design**. Whether there are deadlocks, not the implementation (code).
    - Correctness: prove absence of circular weights or being able to prove the absence of security dependencies (we have circular dependencies now)

- Layer design -- Reduces states

    0. Processes (abstract CPU)
    1. Memory (Paging)
    2. Console, message interpreter
    3. I/O for all peripherals
    4. All other apps
    5. Operator


| Pros | Cons |
| -------- | -------- |
| Verification makes debugging easier | Verification Hard |
| Reduces testing task | Performance overhead |
| | Careful system design

* Modern OS
    * Two layers: Kernel & User
    * More layers may introduce extra overhead for layer crossing, hurt performance

* Hardware Trends
    1. absolute numbers of performance
    2. change over time (trend)
    3. relative chance to other hardware
    
* Semaphore -> used in two different ways
    * Mutual exclusion
    * Coordination

Nucleus - Microkernel
---
* Goal: (Build a user level OS)
    * Flexible OS design
    * Extensibility
    
* Two approaches for building user-level OS
    * VM tech: OS gets direct access to the hw
    * Nucleus layer: 
        * A layer of software in between of hardware, and these uses of OS.
        * provide a set of abstractions and it's interfaces of these abstractions, on top of which all of user-level OSes can be built. 
        * ==Key abstraction: process==


- Abstraction
    - Process
        - control -> hierarchy
        - Two processes: 
            - internal: normal processes
            - external: wrap hw devices behind process API -> same abstraction for I/O. *Reuse existing abstractions*. Make peripherals the same as normal processes
    - Interrupt handlers
    - Scheduling -- Not exported to the user level OS, determined by nucleus itself
    - IPC -- messages
    - Resource allocation
        - Normally resource allocation is handled by the underlying OS. In nucleus, to support these user level OS, resource allocation decisions (how much and which) is pushed to user level OS. 
    - Memory & memory protection

- Economy of abstractions 
    - small set
    - System can be vary small

- Synchronization for nucleus
    - messages
    - Why not semaphores?
        - reuse message API
        - the nucleus replies to unanswered messages on behalf of dead process to avoid deadlock waiting for dead process

- Drawbacks to use messages
    Buffers & buffer management
    - Extra key fundamental resource the system manegement must be aware of
    - how many buffers to give to each process (resource allocation across all processes in the system)
    - process sends messages using its one of its own buffers, can hall the system by sending a lot of messages

TENEX
---
- Terminology
    - Diff terms for same thing: 
        - ==Monitor = Kernel==
        - Executive = shell
    - Same term for diff things: Virtual machine
        - OS syscall interface
        - hardware (VMWare)
        - language (Java)

- Goals
    - Virtual machines
    - UI (exec)
    - Maintainable
    - Backwards compatibility (don't want to rewrite softwares)
    - Protection
    - Efficiency

- Virtual Memory System
    - addr spaces
    - memory maps -> page tables
    - BBN pager -> TLB (cache of page table entries), memory management unit (MMU)

- Sharing
    Three types of memory visits
    1. Direct -> private pages
    2. Indirect
    3. Shared -> pointer into page cache or buffer cache. (for file data)

    - Today: Two processes have two different virtual addresses, but pointing to the same physical pages
    - TENEX: one address space belongs to owner, owner points the the physical page. Processes have an indirect pointers pointing to the owner to share it.
    - TENEX advantage: reduce the overhead of implementing demand paging and updating page table entries.

- Copy-on-write
    - make private copy
    - write
    - update the page table

- Compatibility
    - Jsys
    - compatibility: emulating ==syscall== for binary
    - packages / libraries : implementing code that emulates old syscall in terms of the new syscall.
    - load the compatibility package into the address space of the process into a region that the old processes would never use (load code and data into the address space). Then, TENEX transfers control into the compatibility package

![](https://i.imgur.com/GVBwaC9.png)

- File System
    - Hierarchical (actually simple)
        - C:\Dir\File.ext.version
        - Versioning
    
- no conflicts
    - thaw: multiple processes can access and write the files at the same time, processes have to coorperate with others (today use)
    - unthaw: use lock to prevent corruption


Hydra
---
never give full access to anyone

- Goal
    - Allow users to create user-level operating system functionality by creating subsystems
    - Hydra is a micro kernel to provide the necessary support and primitives for implementing user-level ==subsystems==
    - Push subsystems up to user level

- OS should contain
    - hw abstraction
    - resource allocation
    
- Key terms
    1. protection domains
        - A point of execution where the rights you have are equivalent
        - Cross domains: when a set of ==rights== that you have in terms of how you execute and the set of ==objects== you have reference to will change 
        - Unix: Process / OS kernel
    2. protected control transfer
        - syscall
        - a user level process cannot jump into arbitarily into random part of the code in OS, it can only transfer control at a very well-defined points, the points are the beginnings of all of the syscall entries in OS. That is, a user level process can only start executing inside the kernel at the start of any of the system calls, but no other place -> protected: OS decides where a user level process gets to enter it instead of the process
    3. rights augmentation
        - ==caller has fewer rights== with an object ==than the callee== that is implementing it. (OS has more rights than the process who calls syscall)

> OS as a collection of subsystems

- Hydra Abstractions
    - resources -> objects
    - execution domains -> local name space (LNS), like stack frame, context
    - protection mechanism -> ==capabilities: pointer + right bits==
        - If callee add rights to the capability, when the caller resumes, the rights added are abandoned -> right augmentation
        - make procedure call back into the subsystem, passing capabilities, to amplify the rights
        - procedure call is equivalent of system call in monolithic system like Unix
        - happens at run-time
        - check by Hydra

- Reject hierarchal structure
    - Reject the OS being the code that has full privileged everything in the system and can do whatever it wants, and everything else just sits on top of the OS. 
    - Have lots of protected subsystems out there. One protected subsystem is no more special than other protected subsystems. They take advantage of the Hydra mechanism. 
    - During execution, making procedure calls on stack, kind of hierarchy. But no special for one subsystem than another.

# Structure
Protection
---
- Goal
    - Abstract model: unify protection approaches



|        | Objects  | Domains  |
| ------ | -------- | -------- |
| Hydra  | resource objects | LNS (logical name spaces)     |
| Multics| segments     | principle identifier (user)     |


- Access control list is associated with file, not a specific domain
- capability example: file descriptor
![](https://i.imgur.com/5zKQsaZ.png)


==Multics==
---
- Goal
    - allow users to be able to create their own subsystems, not something tied by the OS
- Protection
    - outline the structure of the protection system both in terms of what goes on in the file system and what happens during execution
    - uniform protection model and mechanism applying to everything in the system
    - ==segmented memory model==
        - each segment has a descriptor associated with it
        - the representation of address space is the collection of descriptors for all its segments, an array called descriptor table
        - virtual address: segment number, offset. (seg# is the index of the segment/descriptor table)

![](https://i.imgur.com/wZOWiDv.png)

- Principles
    - least privileges
    - design not secret
    - usability
    - check every access (every operation is a protection check)

- Mechanism
    -  procetion expressed/derives from the file system
    - capabilities are much easier to check than ACL
    - all the capability that the system uses are all derived from the ACL specified in the file system
    1. login
        every protection check relates to that user
    2. ACL (access control list) + capabilities

![](https://i.imgur.com/VvGn1aG.png)

- Read ("filename", ...) v.s. Read(fd, ...)
    - OS has to check the access
    - why not read by name: efficiency. look up the ACL every time call read. (read disk and fs for ACL to check)
    - opening file: purely a protection check. 
        - convert ACL to capa: bootstrap from the ACL of the protection (specified in fs of a given file or segment, associated with the file). During runtime when use it, when start protection check, bootstrp ACL to capa, then do protection checks from capa
        - does process has right to write a file: process associated user ID (came from login) -> OS read ACL, take userID, see if the process has the right -> if yes, return file descriptor
        - fd encodes the rights we have on the file. check efficiently
        - easy pass fd (file descriptor, capability) to another process

- Use bootstrapping to enforce protection check on every instruction, every memory reference
    - how they relate: for segment for page table entry, they are capa. Each time start a process, use capa for the addr space, efficient protection check. (TLB do protection check besides addr mapping)
    ![](https://i.imgur.com/ZrbD0cq.png)
    
- protection for subsystem
    - entry for subsystem is syscall
    - goal: not only OS implement syscall, each protected subsys to implement syscall
    - one method: transfer control into privileged code (today from user to kernel)
    - multics: fs not executing in OS, but in subsys above it. each user-defined subsys defines its entry point. (like library, library is a protected subsys, and app can only jump into the entry points of the library)
        - Use GATE (entry points, "call" in the graph) and Ring number to do this
        - when transfering control, executed in the new ring number
        - subsys protected from the app, but not protected from the kernel
        - protection checks are only done with respect to segments that are mapped at lower privileged levels

Unix
---
- unifying abstruction: **files**
    - easy to use
    - protection
    - general
    - model
    - also used for devices
    - streaming I/O: many devices map well into streaming I/O interface
    - single namespace

- Protection: set user ID
    - the process is run under the previliege of its owner, no matter who runs it

Plan 9
---
- everything is distributed
- cost
- admin/management cost
![](https://i.imgur.com/fgtqvwx.png)




# Distribution

V Kernel
---
- Goals
    - diskless = local disk
        - the performance of doing file I/O to a remote server is not that much slower than your local disk
    - message IPC -> high perf file I/O
        - no need to add/customize protocol
- V IPC
    - synchronous (not streaming)
    ![](https://i.imgur.com/7ImeJ6w.png)
    - small messages for control (32 bits)
        - separate API for data transfer (move to and from)
        - at least 4 messages for a network round trip -> solution: 2 extra routines, receive/reply with segment. When send control message, also send data. The minimum round-trip is 2.
- perf improvement: use raw Ethernet packets
- network penalty
    - kernel to kernel communication (client/server), no context switches
- kernel IPC primitives
    - same machine v.s. network
    - remote adds miliseconds operation overhead 
        - request time on server is large. Server does disk operations 20ms
        - cache. The network overhead then is large
- IPC synchronous: no need for streaming
    - streaming for network layer (above graph)
        - worst case: program loading. Solution: one req, multiple response, client then sends ACK for the last packet
    - disk I/O streaming: read ahead, write behind 
    - prefetch disk data? I/O bound? whether or not, for remote, it doesn't matter
        

Sprite
---
- Technology trends
    - network: sys admin sharing -> single global namespace (for file and device); process migration to use idle machines
    - large memories: cache file data, reduce I/O disk ops, improve app perf
    - multi-processes: sharing or parallelism? OS be multiprocessor aware

- Cache
    - caching clients
        - multiple clients, single server
        - store file data in multiple machine
        - consistency issue
        - two types of sharing: concurrent, consecutive
        - for consecutive sharing: Check version
        - concurrent: disable sharing. No longer cache file data at the client. Every time write a block, send it to the server, server cache it in memory. If another process at another client wants to read that data, need to fetch it via network. Not as fast as caching data at client locally, but no need to go to disk


# OS/Architecture Interaction
![](https://i.imgur.com/qpWZOdZ.png)

- **Mach**: to reduce overhead for app of communicating with subsystems, and overhead of isolation into different protection domains and communication, decompose OS into a bunch subsys running as a user level process on the top of microkernel, apps running as processes at user level as well, interact with OS with messages. Once inside the OS, the subsys inside the OS just do procedure call to each other, no messages are used to lower the overhead.
- everything inside the OS is doing procedure calls, but app interacting with OS uses IPC messages.
- co-locate: compile the user-level OS and move it back down into the privilege level to link together the OS together with the microkernel code. It's both a monolithic OS and a microkernel all running with privilege. -> cut down messgae traffic

==L4==
---
- Goal
    - Mach performance is slow, fundamental? 68%
    - Co-location needed? No
    - Specialize (customize existing service to run faster, pipe even can pass fd) & extend system code (add new features, cache partitioning to a process) -> increase app perf, evn faster than a monolithic system(too general)
- Abstraction
    - threads (ability to execute in CPU)
    - address space (processes)
        - L4 manages the page table for all processes
        - grant pages to different processes 
    - IPC (messages)
- Architecture
    - syscall
        - run app and OS at user level
        - read file: the thread call libC read function, the libC read function will be modified to send a message to the L4, then to the Linux server. The linux server invoke read syscall, then send response back along the path. (linux->L4->app) The thread return from the read routine in libC back to the app.
        - Linux server does not interact with L4 to access the memory in the app processes. It has a mapping to physical memory of the process.
    ![](https://i.imgur.com/zgP0OGv.png)
    - Page fault
        - same for interrupts and all hardware events
        - virtual page miss -> TLB miss. Hardware MMU/TLB transfers control to L4 -> L4 packages up the fage fault in a message and send it to the OS server -> linux allocates a physical page, updates a page table entry, and respond back to L4
    - Shadow paging
        > cannot trust linux server to change PT
        - L4 page table is shadowing the changed which are being made to the PT in Linux addr space.
        - allow Linux to create page table, but these page tables are not installed in MMU. When Linux modified a page table, make syscall down to L4 microkernel, the L4 verifies the modification, and install the page table entry if yes.
        - one page table in Linux for it to accesss its data structures and minimize the changing code.
        - one page table in L4 is used by the hardware. When Linux makes a change to PT, need to trap to do a syscall to L4, L4 verifies, then apply the change to L4 page table. The change is propagated.
- keep most of Linux unmodified
    - Treat L4, the micro kernel as another CPU architecture. Only need to modift the machine dependent file to interact with L4. The rest of Linux keeps unchanged


Exokernel
---
Push L4 further

- Goal
    - push everything up to the user level -> hw management
    - expose hardware
        - can use user level page tables directly and install page table in MMU
- Structure
    - Not run as a separate server at user level, the OS is linked to the app as a library
    - Rather than have it be in a separate process, the OS lives inside of each running process. cut out all the fat
    - every app has an OS linked to it
    ![](https://i.imgur.com/lQ6U1hl.png)
- Benefits
    - can do everything in the library
        - customize/specialize OS for app (improve perf)
    - ==syscall -> procedure call== (into the OS code)
        - Syscall is expensive, this reduces syscall
- Security
    - trust user level and keeps verification
    - process can only visit the pages given to it by Exo
- Exo functions
    - track resource
        - has a table to manage resource for each process
    - protection
        - process want to create PT, map its virtual page into a new physical page -> evoke Exokernel, say install PT entry -> exo checks if the physical page is given to that process -> return
    - revocation
- upcall
    - if TLB miss, MMU (hardware) do upcall into the library OS, upcall changes the protection domain, overhead of syscall. To reduce it, Exo caches TLB entries and software TLB (array of recent PT entries by user level OS). When TLB miss, exo looks software TLB, if finds, can quickly install it in TLB without upcall.
    - network package arrives, an interrupt, trap to the Exo, Exo figures out who the packet is for and delivers it up into the user level. to reduce, use packet filter into Exo to figure out the destination, then call application-specific handlers(ASH, app code verified and installed in Exo kernel. Do it within Exo without upcall.


# Hardware Virtualization
Two appoaches: 
1. VMWare workstation (hosted). Install a drive in OS to get info like pages, most of it runs as an app.
2. VMWare server / Xen (bare metal). Boot into the virtualization software (Xen) which runs in privilege.
![](https://i.imgur.com/DcrTMss.png)

VM370
---
- Why VMs?
    - hw access
    - isolation
    - continuous operation
    - emulate new hw/CPU instructions
    - os development
- Key to approach
    - trap & emulate
        - run OS in user level, when the CPU hit a privileged instruction, it generates an exception and trap to the underlying software(hypervisor), the underlying sw emulates the execution of that privileged instruction.

Xen
---
- Why VMs?
    - servers
- Requirements
    - isolation (security, performance)
    - performance (minimize overhead)
    - multiple OSes
    - scalability
- Challenge
    - x86 pain
- Approach
    - paravirtualization
        - modify OS (like L4, port the OS)
- Key Resources
    - CPU: privilege instructions -> "hyper" call into Xen 
        - trap in hypervisor, not OS. Remove privilege instruction and call in
        - VMWare uses binary writing. OS unmodified. The instruction are rewritten for CPU to jump back to VMWare software
        - OS -> ring 0; Hyper -> root ring 0
    - MEM
        - add 3rd layer of hardware/machine memory
        - ==virtual --OS--> physical --Xen--> hw/machine memory (DRAM)==
        ![](https://i.imgur.com/CFh7Izr.png)
        - page tables installed in MMU: virtual -> hardware
            - Xen: same Exo, give machine pages to OSes. If OS gonna update, call Xen and Xen verifies and updates PT. Need to modify OS source.
            - VMWare: same as L4, shadow page tables. Both OS and VMWare have page tables (only MMU installed vmware PT). Rewrites binaries and change code where OS update PT. OS makes the change and goes back to call VMWare for vmware to propagate the change shadow pages. No need to modify OS.
    - I/O
        - use domain0 to run all control sw. Normal Linux with normal device drivers. Xen allow drivers loaded in domain0 to access devices locally. Domain0 is trusted. (reuse device drivers in Linux)
        - for other untrusted domains, write an emulation layer maps from the device driver of that OS to emulate Ethernet card.
        - IP packet -> Ethernet device -> emulated Ethernet card -> domain0 -> use real Ethernet card
        - add PT/TLB to each devices to reduce copy overhead
        ![](https://i.imgur.com/xsCKYi0.png)


# Scheduling
- user level threads: enable functional parallelism. The number of threads is not tied to the number of cores, but tied to the workload
- kernel level threads: in charge of physical parallelism, decides which kernel thread runs on which core. Increase the actual parallelism by using multiple cores at the same time
- all language threads are user level threads, be managed and scheduled by the language runtime
- for physical parallelism, the language runtime uses the OS interface or a library interface like pthreads to create kernel threads to multiplex user level threads
- OS uses its scheduler to schedule on all of the physical cores
![](https://i.imgur.com/VegvhkH.png)
- advantage

| user          | kernel               |
| ------------- | -------------------- |
| less overhead | system integrated    |
| customizible  | more protection      |
| tailored      | hardware parallelism |

For system integration, when it comes to page faults and I/O, kernel threads perform better. When user level thread calls syscall read I/O, the kernel thread which executing the user level thread is blocked waiting for I/O to complete. Then the user level process will lose a core, the core may run another process. 
The kernel scheduler has no insight on user level threads, which may be in its critical section


Scheduler Activations
---
- Goal
    - remove kernel level scheduler (it ignores information of what user level scheduler knows)
    - every time when need to make a scheduler decision, make an upcall into the user level scheduler. The user level scheduler makes the decision

- Structure
    - the number of SA = the number of physical core / processes
    - when SA blocks, OS does not decide which other kernel level threads run on that core, instead, OS makes an upcall into the user level shceduler and let it decides which of its user-level threads it wants to run on the core that is freed up by the thread that is doing I/O or page fault.
    - OS just do resource allocation, allocate cores among processes
    - user level scheduler knows most about threads 
    - OS adds a core to a process: OS creates a new SA and that SA makes an upcall into the user level shceduler to notify the process 


Lottery Scheduling
---
- What: ==proportional share scheduling==
    - tickets (process with more tickets are more likely to be chosen during lottery / a dice rolling)
- Why?
    - abstract: tickets represent CPU time
    - relative
    - uniform
    - modular
    - no startvation
- transfer -> RPC server
- inflation -> make a process run faster
- currencies -> convenients modularizatoin, hand out tickets in base currency 


# I/O and File System
FFS
---
- problems in old FS
  - random data block placement for older FS -> Related data close (cylinder groups)
  - inodes allocated far from data block -> inodes and data close
  - small max file size -> larger block size
  - waste from larger block size -> use fragments
  - slow recovery -> replicated superblocks
  - device oblivious -> parameterize fs device characteristics
- need to understand physical disk architecture
    - disk partitions are split into one or more cylinder groups. 
    - cylinder group comprised of one or more consecutive cylinders on disk
- two-level allocator
  - Global: localize related data within a cylinder group
  - local: optimal local placement within a cylinder group
- Performance
  - increase locality to minimize seek latency
  - improve layout to make large transfers possible
  - disk accesses for ls dir cut in half


LFS
---
- A log structured file system 
- writes all modifications to disk sequentially in a log-like structure
- speed up both file writing and crash recovery
- The log is the only structure on disk; it contains indexing information so that files can be read back from the log efficiently
- FFS v.s. LFS
    - Main point: fs designed to exploit hw and workload trend(disk bw is scaling, but latency not; large main memory, buffer, absorb read, merge small writes into large); workload(small file access)
    - Problems of FFS: layout(inode separate from file, dir separate from inode); sync write(metadata need sync write)
- How LFS work 
    - Buffer small write into large to maximum use of disk bw; All info written to disk is appended to log
- Challenge: How to find data 
    - Index tables(inode map, file number -> inode location), written into log, ckpt, keep inode map into memory
- How to manage free space
    - Divide log into segs, thread segs in disk; do free space management at segment level(copy live data to the end, have free seg). Need to identify which is live.
- Cleaning policy
    - Which to clean? Cost-benefit, combine potential free space with age of segment(cold seg are cleaned at higher utilization)
- Crash recovery
    - Checkpoint
    - roll-forward


Soft Updates
---
- Write metadata
- tracks and enforces metadata update dependencies to ensure that the disk image is always kept consistent
- Obtain and track dependencies for its “real physical blocks”.
- Principle: never have pointer to null on disk, ok to have inode wasted(can be gc)
- When delete a file: Remove entry -> remove inode
- When create a file: create inode -> add entry
- consistency issue
    - fs use sync write to ensure consistency of update of metadata, solve "update dependency" problem.
    - Soft update: enable write-back caching of metadata(already have it for data)
- Why? Performance, integrity(handle all potential corruption), recovery(instant mounting).
- Roll back metadata in A(any one) to an earlier, safe state


Rio
---
- Goal
    - write-back performance with write-through reliability.
- How? 
    - Eliminate reliability-induced writes to the disk
    - protect file cache memory during normal operation, restore file cache contents on "warm" reboot.
- How? 
    - VM support, file cache page read-only, all access use VM protection(unprotected when write)
- Registry: tell kernel to find cache page on reboot, keep track on them, need to be stored in protected, recovered area
- protection domains separate from address space; separation of file buffer cache into separate protected module.

# Review

Memory map version
---
1. open
2. map file -> pointer in addr space
3. if access file data, dereference the pointer 
4. close

Streaming I/O
---
- traditional way which I/O is thought
- Unix treats everything as file, includes devices, streaming I/O maps onto devices as well as files. Run program both in file and device
- hard to map device I/O into a memory map model


paravirtualization v.s. shadow page tables
---

OS under VM has hierarchical page tables representing the address spaces of these processes.

virtual & physical & machine memory
Xen allows the OS to maintain its page tables



GMS & Sprite
---
- GMS
    - from user perspective, if use a page and faults out, it is not like the OS exposes what the pages are to them. If the user fault, the OS is performed global local page
- Sprite
    - user has task to do, not manually move to an idle machine, 
    - abstracts the process for them, instead of make availbe to them the info they choose
    - virtualize what that others don't?
        - the process does not care which machine they are running on. still run exactly the same way
        - virtualize a location where a process executes
        - virtualizes/makes files and devices transparent (every process is in the same namespace)

