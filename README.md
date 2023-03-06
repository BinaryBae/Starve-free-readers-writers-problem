# Starve-free-readers-writers-problem

The Reader-Writer Problem is a synchronization problem that involves multiple processes accessing a shared resource. The problem arises when some processes only need to read the shared resource, while others need to write to it.

The goal is to ensure that the concurrent access to the shared resource by multiple processes is coordinated in a way that avoids conflicts and maintains data consistency. The key challenge is to provide a solution that ensures the integrity of the shared resource while minimizing the time that processes spend waiting for access.

In general, the Reader-Writer Problem has two types of processes: readers and writers. Multiple readers can access the shared resource simultaneously, while only one writer can access the resource at a time. When a writer accesses the resource, no other process, whether it is a reader or a writer, can access it until the writer is finished writing.

The problem is to find a solution that prevents race conditions, where two or more processes access the shared resource at the same time, leading to unpredictable and potentially incorrect behavior. Additionally, the solution must avoid starvation, where a process is continually blocked and unable to access the shared resource due to other processes holding it for extended periods.

Many different algorithms have been developed to solve the Reader-Writer Problem, such as the First-Come-First-Served (FCFS), Readers-Writer Locks (RWL), and Priority Readers-Writer Locks (PRWL). These algorithms differ in their complexity, fairness, and ability to prevent starvation.

### Pseudocode for starve free Reader-Writter problem

#### Global Variables

Global variables shared across all the processes. They are as follows:

```cpp
Semaphore* mutex = new Semaphore(1);
Semaphore* rw_mutex = new Semaphore(1);
Semaphore* write_sem = new Semaphore(0);
int read_count = 0;
bool writer_waiting = false;
```

#### Implementation: Reader

```cpp

// Reader code

do{
    // Entry section
    wait(mutex);
    read_count++;
    signal(mutex);
    
    // Critical section
    // Reading happens here

    // Exit section
    wait(rw_mutex);
    read_count--;
    if(read_count == 0 && writer_waiting){
        signal(write_sem);
    }
    signal(rw_mutex);

    // Remainder section

} while (true);
```

The reader part of the code is executed by multiple reader processes concurrently. The reader code is enclosed within an infinite loop. In the entry section, the process acquires a mutual exclusion lock called `mutex` before incrementing the `read_count` variable, which keeps track of the number of readers that are currently accessing the shared resource. The mutual exclusion lock ensures that multiple readers don't access the shared resource simultaneously, which can result in race conditions.

In the critical section, the process performs the reading operation, i.e., it accesses the shared resource. Multiple readers can access the shared resource simultaneously in this section.

In the exit section, the process acquires a lock called `rw_mutex` before decrementing the `read_count` variable. If the number of readers accessing the shared resource becomes zero after decrementing `read_count`, it means that all readers have completed accessing the shared resource, and the writer can now access the resource. In this case, the `writer_waiting` variable is checked, and if it is true, the `writer_sem` is signaled to allow the waiting writer to proceed. The `rw_mutex` lock is then released.

#### Implementation: Writter

```cpp

// Writer code

do{
  // Entry section
  wait(mutex);
  wait(rw_mutex);
  if(read_count == 0){
    // Release the rw_mutex
    signal(rw_mutex);
  }
  else{
    // Reader(s) is/are present
    writer_waiting = true;
    signal(rw_mutex);
    wait(write_sem);
    writer_waiting = false;
  }

  // Critical section
  // Writing happens here

  // Exit section
  signal(mutex);

  // Remainder section
} while (true);
```

The writer part of the code is executed by multiple writer processes concurrently. The writer code is enclosed within an infinite loop. In the entry section, the process first acquires the mutual exclusion lock called `mutex`, which ensures that only one writer or reader can execute the critical section at any time. Then the process acquires a lock called `rw_mutex`, which is used to ensure that no other reader or writer is currently accessing the shared resource.

If there are no readers accessing the shared resource, the writer releases the `rw_mutex` lock and proceeds with the critical section, i.e., it accesses the shared resource exclusively. Otherwise, if there are readers accessing the shared resource, the writer sets the `writer_waiting` variable to true, signals the `rw_mutex` lock, and waits on the `writer_sem`, which is initialized to 0.

Once the `writer_sem` is signaled by the last reader leaving the critical section, the writer proceeds with the critical section, i.e., it accesses the shared resource exclusively. Then the process releases the mutex lock and proceeds to the remainder section.

#### Mutual Exclusion:

Mutual exclusion is ensured in the code by using two semaphores:
- `mutex`
- `rw_mutex` 
The `mutex` semaphore ensures mutual exclusion for the readers, and the `rw_mutex` semaphore ensures mutual exclusion for the writers. The `write_sem` semaphore is used to signal the waiting writer when all readers have completed accessing the shared resource.

#### Progress:

Progress is ensured by checking the `writer_waiting` variable before signaling the `write_sem` semaphore. If there are no waiting writers, the `read_count` variable will be incremented, and the process will enter the critical section. Similarly, if there are no readers, the writer will release the `rw_mutex` lock and proceed with the critical section.

The given code is a solution to the classic Reader-Writer problem where multiple processes (readers and writers) access the same shared resource. The goal of this problem is to ensure that multiple readers can access the resource simultaneously, while exclusive access is provided to the writers.
