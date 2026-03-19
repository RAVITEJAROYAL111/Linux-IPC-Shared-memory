# Linux-IPC-Shared-memory
Ex06-Linux IPC-Shared-memory

# AIM:
To Write a C program that illustrates two processes communicating using shared memory.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Shared Memory

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/wait.h>

#define TEXT_SZ 2048   // Shared memory size

// Shared structure
struct shared_use_st {
    int written;
    char some_text[TEXT_SZ];
};

int main() {
    int shmid;
    void *shared_memory = NULL;
    struct shared_use_st *shared_stuff;

    // Create shared memory
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    if (shmid == -1) {
        perror("shmget failed");
        exit(EXIT_FAILURE);
    }

    printf("Shared memory ID = %d\n", shmid);

    // Attach shared memory
    shared_memory = shmat(shmid, NULL, 0);
    if (shared_memory == (void *)-1) {
        perror("shmat failed");
        exit(EXIT_FAILURE);
    }

    printf("Memory attached at %p\n", shared_memory);

    shared_stuff = (struct shared_use_st *)shared_memory;
    shared_stuff->written = 0;

    pid_t pid = fork();

    if (pid < 0) {
        perror("fork failed");
        exit(EXIT_FAILURE);
    }

    // ================= CHILD (CONSUMER) =================
    if (pid == 0) {
        while (1) {
            while (shared_stuff->written == 0) {
                sleep(1);   // wait for producer
            }

            printf("Consumer received: %s", shared_stuff->some_text);

            if (strncmp(shared_stuff->some_text, "end", 3) == 0) {
                break;
            }

            shared_stuff->written = 0;
        }

        // Detach memory
        if (shmdt(shared_memory) == -1) {
            perror("shmdt failed");
            exit(EXIT_FAILURE);
        }

        exit(EXIT_SUCCESS);
    }

    // ================= PARENT (PRODUCER) =================
    else {
        char buffer[TEXT_SZ];

        while (1) {
            printf("Enter Some Text: ");
            fgets(buffer, TEXT_SZ, stdin);

            strncpy(shared_stuff->some_text, buffer, TEXT_SZ);
            shared_stuff->written = 1;

            if (strncmp(buffer, "end", 3) == 0) {
                break;
            }

            while (shared_stuff->written == 1) {
                sleep(1);   // wait for consumer
            }
        }

        wait(NULL);

        // Detach memory
        if (shmdt(shared_memory) == -1) {
            perror("shmdt failed");
            exit(EXIT_FAILURE);
        }

        // Remove shared memory
        if (shmctl(shmid, IPC_RMID, NULL) == -1) {
            perror("shmctl failed");
            exit(EXIT_FAILURE);
        }

        printf("Shared memory removed\n");
        exit(EXIT_SUCCESS);
    }
}

## Write a C program that illustrates two processes communicating using shared memory.





## OUTPUT:
<img width="832" height="185" alt="image" src="https://github.com/user-attachments/assets/e3da7714-644c-4a88-b3a6-8dc58d364533" />



# RESULT:
The program is executed successfully.
