# Exploring POSIX Threads in C

This project explores POSIX threads with three example programs. It demonstrates multi-threaded programming with shared resources, synchronization mechanisms, and passing arguments to threads.

---

## Contents

### Example 1: Incrementing a Global Counter
- **Objective**: Create a program that spawns a thread to increment a global counter every second while the main thread waits for user input.
- **Key Points**:
  - The counter continues running while waiting for user input.
  - Measures the elapsed time during user input.

### Example 2: Synchronizing with Mutex
- **Objective**: Modify the above program to protect the global counter with a mutex.
- **Key Points**:
  - Ensures thread safety when accessing the global counter.
  - Discusses the necessity of the mutex and the timer's accuracy.

### Example 3: Passing Arguments to Threads
- **Objective**: Implement a second thread function that takes a `double` argument and returns a `double`.

---

## Example Code

```c
#include <stdio.h>      // printf()
#include <stdlib.h>     // exit()
#include <pthread.h>    // POSIX threads

// Function declarations
static void *threadFunc(void *arg);
static void *threadFunc2(void *arg);

// Static global variables
static volatile int glob1 = 0; // Updated with a mutex
static volatile int glob2 = 0; // Updated without a mutex

// Initialized a mutex
static pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;

int main(int argc, char *argv[]) {
    // Variables for thread IDs
    pthread_t t1, t2, t3, t4;

    // Number of times each thread will increment the global variables
    int loops = 1000000;

    // Create two threads to increment glob1 (with a mutex for safety)
    pthread_create(&t1, NULL, threadFunc, &loops);
    pthread_create(&t2, NULL, threadFunc, &loops);

    // Create two threads to increment glob2 (without a mutex)
    pthread_create(&t3, NULL, threadFunc2, &loops);
    pthread_create(&t4, NULL, threadFunc2, &loops);

    // Join the threads (this waits for each thread to finish)
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_join(t3, NULL);
    pthread_join(t4, NULL);

    // Access glob1 and glob2 safely (no mutex needed as threads are finished)
    printf("\nglob1 (updated with mutex) = %d\n", glob1);
    printf("\nglob2 (updated without mutex) = %d\n", glob2);

    // Destroy the mutex
    pthread_mutex_destroy(&mtx);

    return 0;
}

// Function to increment glob1 using a mutex
static void *threadFunc(void *arg) {
    int loops = *(int *)arg; // Cast void* to int* and dereference
    int loc;

    for (int j = 0; j < loops; j++) {
        pthread_mutex_lock(&mtx); // Lock the mutex
        loc = glob1;             // Read global variable
        loc++;                   // Increment locally
        glob1 = loc;             // Update global variable
        pthread_mutex_unlock(&mtx); // Unlock the mutex
    }

    return NULL; // No return value needed
}

// Function to increment glob2 without using a mutex
static void *threadFunc2(void *arg) {
    int loops = *(int *)arg; // Cast void* to int* and dereference
    int loc;

    for (int j = 0; j < loops; j++) {
        loc = glob2;         // Read global variable
        loc++;               // Increment locally
        glob2 = loc;         // Update global variable
    }

    return NULL; // No return value needed
}
```

---

## How to Run
1. Clone the repository.
2. Compile the program with `gcc`:
   ```bash
   gcc -pthread -o posix_threads posix_threads.c
   ```
3. Run the compiled executable:
   ```bash
   ./posix_threads
   ```

---

## Discussion Points
- **Thread Safety**: The use of `pthread_mutex_t` ensures mutual exclusion for shared resources, avoiding race conditions.
- **Timer Accuracy**: The accuracy of the timer thread depends on system scheduling and the sleep duration.
- **Thread Arguments**: Demonstrates how to pass and return data using `void*` pointers.
