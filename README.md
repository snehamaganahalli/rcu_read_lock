**RCU** : RCU stands for **Read Copy Update**


1) It is used when there is a shared data structures (like linked list/trees/hash tables).
2) It is used when there are more readers.

When a thread/writer is inserting/deleting elements from the data-structures(shared memory), all the readers are guranteed to see either the older value or the newer value. Therefore, it will avoid the inconsistencies (like deferencing a NULL pointer)

