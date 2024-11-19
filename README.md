**RCU** : RCU stands for **Read Copy Update**


1) It is used when there is a shared data structures (like linked list/trees/hash tables).
2) It is used when there are more readers.

When a thread/writer is inserting/deleting elements from the data-structures(shared memory), all the readers are guranteed to see either the older value or the newer value. Therefore, it will avoid the inconsistencies (like deferencing a NULL pointer)

**Eg 1:**
struct shared_data {
int a;
int b;
int c;
}

old value of a,b,c: 1,2,3
New value of a,b,c: 4,5,6

Suppose you change the value of a,b,c from 1,2,3 to 4,5,6. Then the readers will see the value either 1,2,3 or 4,5,6 but not the NULL values (in between state of 1,2,3 and 4,5,6)

**Eg 2:**

Suppose you have linked list as below.
a=>b=>c

And you want to delete element 'b'.

**States:**

1) a=>b=>c
2) a=>b=>c 
   |_____|
  i.e You have a link from b to c and a to c.
This is the intermediate state when deleting the element 'b'.
Here few readers will be accessing 'b' from 'a' and can traverse to 'c' . And few readers will be accessing 'c' from 'a'.
In either case, the readers will see a vaild data i.e, either a,b,c or a,c. 
   Do synchronize_rcu(), it will wait till all the readers are have completed the execution of the critical section. Now 'b' can be removed from the list/deleted.
   Now on, the readers cannot get 'b'.
5) a=>c
   
**NOTE:**
In the second state, different readers will see the different versions of the linked list.
version 1: a=>b=>c
version 2: a=>c

**Difference between traditional locking vs RCU:**

**RCU:** 
a) Reader side critical section can be simultaneously executed by many readers.
b) While updating the shared_memory, the reader can still read the shared_data, it is guranteed to get the old_version/new_version but never the inconsistent version of the data.
c) Only while deleting the shared_data, we ensure that all the readers have left the critical section.
d) Saves lot of CPU cyces, thus increases performance.
e) Suitable when there are many readers.
**f) readers can see the 2 versions of the shared_data. Therefore do not use when all readers should see the same data.**

**Traditional locking:**
a) b) c) Either reader or writer will execute the lock/mutually exclusive.
d) Lot of CPU cycles are used. Reduced performace.
e) Suitable when there is single reader/few readers and a writer.
**f) All the readers see the single updated version of the data.**

**RCU APIs**

**rcu_read_lock():** Inform others the reader is entering the critical section, does not actually block anyone from entering the critical section.
e)
**rcu_read_unlock():** Inform others the reader has exit critical section.

**rcu_assign_pointer():** It is same as '=' operator. But it will avoid compiler optimization to arg1, since it will use the loop as hown below.
#define RCU_INIT_POINTER(p, v)	do { (p) = (v); } while (0)
#define rcu_assign_pointer(p, v)	do { (p) = (v); } while (0)

**syncronize_rcu():** Wait for readers who are already in the critical section and haven't left yet. Used after updating the data and before the old data has been released.

**call_rcu():** this function is the asyncronized version of syncronize_rcu(), it will return right after it schedules a handler (clean up the old data) to be performed after all previous readers left C.S.(critical section)

**syncronize_rcu():** Wait for readers who are already in the critical section and haven't left yet.
**Used after updating the data and before the old data has been released.**

**call_rcu():** this function is the asyncronized version of syncronize_rcu(), it will return right after it schedules a handler (clean up the old data) which will be called after all previous readers left C.S.(critical section).

void Reader(){
    rcu_read_lock();    //enter C.S. (critical section)
    item *data = shared_data;
    access_data(data);
    rcu_read_unlock();  //leave C.S.
    return;
}

void Updater(){
    item *old_data = shared_data;  //save the old location
    item *new_data = new(item);    //make a new item
    update_data(new_data);   //modify the data in new item
    rcu_assign_pointer(shared_data, new_data);   //now the global pointer points to the new item
    syncronize_rcu();   //wait for all readers using old_data to leave C.S. so that you can free the old_data.
    kfree(old_data);  //now we can safely release the old_data
    return;
}

void item_free(item *data){
    kfree(data);
    return;
}
void Updater(){
    item *old_data = shared_data;  //save the old location
    item *new_data = new(item);    //make a new item
    update_data(new_data);   //modify the data in new item
    rcu_assign_pointer(shared_data, new_data);   //now the global pointer points to the new item
    call_rcu(old_data, item_free);   //schedule item_free(old_data) to be performed after all previous readers left C.S. and **return immediately**
    return;
}

**References:**
https://free5gc.org/blog/20231129/20231129/#another-version-of-updater-with-call_rcu


