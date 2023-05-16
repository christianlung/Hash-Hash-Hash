# Hash Hash Hash
This lab xxx

## Building
```shell
make clean && make
```

## Running
```shell
./hash-table-tester -t 4 -s 50000
Generation: 41,286 usec
Hash table base: 350,728 usec
  - 0 missing
```

## First Implementation
In the `hash_table_v1_add_entry` function, I initialized a `statically allocated mutex` on line 11.
I added the lock on line 96 and unlocked it on line 114. I blocked lines 100-113, guaranteeing correctness because the fetching of the list head and list entry and the insertion of the entry are all protected. No other thread is able to insert any elements in the hashtable while the critical section is locked, ensuring proper functionality without race conditions. The mutex is deleted in the destroy() function. If the list entry exists, unlock the mutex and return.

### Performance
```shell
./hash-table-tester -t 4 -s 50000
Generation: 41,286 usec
Hash table base: 350,728 usec
  - 0 missing
Hash table v1: 636,209 usec
  - 0 missing
```

This time version 1 is slower than the base version. This is because when one thread is inserting into the hash table, no other threads can insert into the hash table concurrently. Each thread must wait for the previous thread to unlock its mutex first. As a result, this implementation is slow since it processes only one thread at a time, allowing for no parallelism when inserting. This does not capitalize on the modularity provided by the different hash table entries.

## Second Implementation
In the `hash_table_v2_add_entry` function, I implemented a mutex in the hash table entry struct. In total, there is HASH_TABLE_CAPACITY amount of mutexes. The mutex for each hash table entry is locked when retrieving the list head and list entry and inserting the list entry. It is unlocked after critical section and is also conditionally unlocked if the list entry already exists. Each one is initialized in the create() function and deleted in the destroy() function. 

### Performance
```shell
./hash-table-tester -t 4 -s 50000
Generation: 41,286 usec
Hash table base: 350,728 usec
  - 0 missing
Hash table v1: 636,209 usec
  - 0 missing
Hash table v2: 59,456 usec
  - 0 missing
```

This time the speed up is 5.9 times faster, reflective of the quad core processor the program is running on. This speed-up owes its performance to the parallelism achieved through the multiple fine-grained mutexes, since there is a mutex for each hash table entry. When a thread locks the critical section, it only locks the corresponding hash table entry. As a result, insertions into other entries can happen concurrently since there is no dependency between entries. Threads no longer have to wait one at a time to insert elements.

## Cleaning up
```shell
Make clean
```