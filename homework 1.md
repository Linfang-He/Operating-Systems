# CS 221 hw1

###### tags: `UCSD`


## Question 1

(a) the protection domain that they support, 
(b) the mechanism for crossing protection domains, 
(c) how rights are represented, 
(d) how rights are amplified crossing domains,
(e) how the OS determines whether to allow the domain crossing.


### Hydra

(a) The protection domain that HYDRA supports is the "local name space" (LNS). LNS is the record of the execution environment (domain) of a procedure when that procedure is invoked. Those objects and rights change into new ones as the LNS moves to another.

(b) An LNS is uniquely associated with an invocation of a procedure. For crossing protection domains,  the LNS (execution domains) change precisely when a procedure is entered or exited (operated by the kernel functions: call and return), creating a new LNS or return to the caller's LNS.

(c) Right is represented by capability, which consists of: a reference to an object and a collection of "access rights" to that object. The capability, an object which can only be operated by the kernel, includes information about operations may be performed by the referenced objects. 

(d) When creating a new LNS, a capability is constructed by merging the caller's rights with the rights specified in the template. Thus, a callee may have greater freedom to operate on an object than the caller.

(e) "If a type mismatch occurs, the call is not permitted and an error code is returned to the caller. If the types agree, the rights are then checked, using a special field present only in templates called the "check- rights field." The rights contained in the actual parameter capability must include the rights specified in the check-rights field of the template; otherwise, a protection failure occurs and the call is not permitted. If the caller's rights are adequate, a capability is constructed in the (new) LNS which references the object passed by the caller, but which contains a rights list specified by the template." (Hydra p. 341)
The kernel examines the actual parameter capabilities supplied by the caller and determines whether all protection requirements are met when a procedure wishes to call another. If so, the kernel creates a new LNS (domain) which defines the new environment, and the caller's LNS is superceded by this new LNS for the duration of the called procedure's execution. It may also make use of "template", which defines the checking to be performed when a procedure is called.

### Multics
(a) The protection domain of Multics is the "protected subsystems", which are collection of procedures and data that can only be used via designated entry points. Also, "virtual address space" is protection domain for processes, which is the set of processor addresses that the process can use to reference information in memory.

(b) Only through the designed entry points (gate) can a protected subsystem enter.

(c) Rights are represented by "access control list" (ACL) associated with each segment, which is "an open-ended list of names of users who are permitted to reference the segment". (Multics p.390) The ACL provides protection for stored information.

(d) Rights are amplified crossing domains by borrowing a program, and use the protection ring mechanism to run the borrowed one with lower access privileges.

(e) The protection ring mechanism. Each subsystem is assigned a number between 0 and 7, and the hardware permits a subsystem to use all of those descriptors containing protected subsystem numbers greater than or equal to its own.


### Pilot

(a) The Pilot is a single-user system, and everything runs in a single address space. Therefore, it can be regarded that the Pilot does not have a protection domain.

(b) There is only one user (domain) in Pilot so no crossing protection domain will ever happen.

(c) The rights are represented by interfaces defining services provided to clients. For example, the interfaces *File* and *Volumn* define the basic facilities for permanent storage of data.

(d) Again, there is only one domain, so no rights crossing domains at all.

(e) There is only one domain, so no domain crossing at all.


## Question 2

1. Why must a traditional operating system like Unix explicitly provide support (e.g., system calls) for process debugging?
Debugging requires the execution of a process has to be paused, which can only be supported by the operating systems. If not, the debugger has no way of obtaining such a privilege.

2. List two distinct operations that a debugger must perform that require support from the operating system.
First, pause the execution of the debugged process.
Second, be able to access and inspect the variable (data) of the debugged process.

3. Because processes are protected and isolated from each other, operating systems must also provide support for communication and coordination among processes. Why can't debuggers just use the support that operating systems already provide for process communication and coordination?
If an operating system provides supports for the debugger to debug another process (pause the execution or inspect the data), then other processes can do the same as well, which definitely violates the isolation and protection among processes. Therefore, debuggers should be treated differently with extra support from operating systems.

4. Do language runtime environments like Java and Perl require operating system support for debugging programs in those languages? Why or why not?
They do not require support from operating system, because they have their own platforms for debugging, like Java Virtual Machine (JVM) for running and debugging Java programs. However, if it needs deeper debugging under the virtual machine layer(for example, thread), system call is also needed. 

5. When working on an operating system, developers also need to use a debugger on the operating system itself. Why is debugging the kernel of an operating system more challenging than debugging a user-level process? What is one option for where to run a kernel debugger?
Debugging the kernel of an operating system is more challenging because the kernel of an OS cannot break and debug itself. 
One solution is that the kernel can connect to another operating system especially for debugging. Then, the current kernel process can be treated as the proxy (debugged process) of the new connected debugging OS.


## Question 3

### First argue why a file system should incorporate versioning as an explicit feature, and justify your argument.

Versioning file system means when a file is modified, the system will automatically trigger the creation of a new version that users can directly access afterwords. For end-users, especially for applications in need of version control, it would be pretty simple and handy if incorporating versioning.
For major operations on files, the versioning file system does not need extra effort from users. The system will automatically assign a higher version number when creating files, and will automatically use the latest version when opening files. Besides, version numbers are user-visible, and people can specify a version as needed, just remember to purge down old versions if you no longer need them.

### Then argue why a file system should not incorporate versioning, also justifying your argument.
A versioning file system keeps the older versions of files, which will store large redundant information (even only allocate data blocks of a file that are changed). It is a waste of resource. For majority applications or users, only the latest file is needed. If users do need to keep the older ones, he can explicitly backup the file in storage. In conclusion, not incorporating versioning makes the most of storage resource, and does not introduce much inconvenience.

### For you personally, in terms of your computing needs, which argument do you find more compelling?
I am a CS student writing codes a lot. However, I prefer not incorporating versioning. Maybe it is just a habit due to using Linux or MacOS for a long time.
I prefer to control the versions by committing the one I needed, instead of depending on the system to keep every version of code automatically. The version number is also not so easy to understand as committing messages or different backup file names. In addition, I cannot afford large storage, so I prefer to save resource for other files.

Thus, I find not incorporating versioning more compelling to me.
