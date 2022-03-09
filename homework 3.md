# CS 221 hw3

###### tags: `UCSD`

Question 1
---
Why do scheduler activations have to worry about this deadlock issue, but standard kernel threads implementations do not have to?
- For standard kernel threads, the thread ready list is managed and available by the kernel. The kernel can avoid deadlocks by the information from the ready list if some thread holds a lock. 
- For scheduler activations, however, the thread ready list is in user space, and the kernel has no idea on which thread holds a lock on it. In this case, the preempted thread could be holding a lock on the user-level thread ready list, and deadlock would occur if the upcall attempted to place the preempted thread onto the ready list. So, scheduler activations have to worry about deadlock issues.


Question 2
---
a. Let f be a new file created in a directory d. The file system will issue at least three disk operations to complete this operation. Ignoring any data blocks allocated for the directory or new file, what are these three disk operations for?
- Create file inode
- Find directory inode
- Write directory entry

b. In Unix FFS, at least two of these writes will be issued synchronously. Which are they, and what order should they be performed in? Briefly explain why.
- Creating file inode and writing directory entry are issued synchronously.
- File inode must be created before writing directory ("an inode must be initialized before a directory entry references it").
- The order is to ensure metadata consistency. If the order is reversed and the system goes down after the new directory entry has been written to disk but before the initialized inode is written, consistency may be compromised, since the contents of the on-disk inode are unknown. To ensure metadata consistency, the initialized inode must reach stable storage before the new directory entry.

c. Consider the Soft Updates solution to this problem. Does it do any reliability-induced synchronous writes? If so, how does it differ from FFS? If not, why can it avoid doing so? Explain.
- No. Soft Updates do not do synchronous writes by allowing safe use of delayed writes for metadata updates. Synchronous metadata updates have been replaced with delayed writes. The disk write routines have been modified to perform the appropriate undo/redo actions on source memory blocks.
- "It combines multiple metadata updates into a much smaller quantity of background disk writes. However, to maintain integrity in the face of unpredictable failures, sequencing constraints must be upheld as dirty blocks are propagated to stable storage. To address this requirement, the soft updates mechanism maintains dependency information, associated with any dirty in-memory copies of metadata, to keep track of sequencing requirements." (SU p.133)

d) Consider the same operation in LFS. Does LFS generate any reliability-induced synchronous writes? Explain.
- No. The LFS converts the synchronous random writes into large asynchronous sequential transfers. It buffers a sequence of file system changes in the file cache and then writing all the changes to disk sequentially in a single disk write operation. So no synchronous disk writes are used.

e) Consider the same operation with the Rio file cache. Does Rio generate any reliability-induced synchronous writes? Explain.
- No. Because Rio eliminates all reliability-induced writes to disk, and achieves write-through reliability by protecting memory during a crash.