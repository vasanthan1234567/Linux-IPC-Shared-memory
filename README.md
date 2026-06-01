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

## Write a C program that illustrates two processes communicating using shared memory.
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <sys/wait.h>

#define TEXT_SZ 2048  // Shared memory size

struct shared_use_st {
    int written;  
    char some_text[TEXT_SZ];
};

int main() {
    int shmid;
    void *shared_memory = (void *)0;
    struct shared_use_st *shared_stuff;

    // Create shared memory
    shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666 | IPC_CREAT);
    if (shmid == -1) {
        perror("shmget");
        exit(EXIT_FAILURE);
    }

    printf("Shared memory ID = %d\n", shmid);

    // Attach to shared memory
    shared_memory = shmat(shmid, NULL, 0);
    if (shared_memory == (void *)-1) {
        perror("shmat");
        exit(EXIT_FAILURE);
    }

    printf("Memory attached at %p\n", shared_memory);

    shared_stuff = (struct shared_use_st *)shared_memory;
    shared_stuff->written = 0;

    pid_t pid = fork();

    if (pid < 0) {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) {
        // Child process: Consumer
        while (1) {
            while (shared_stuff->written == 0) {
                sleep(1); // Wait for data
            }

            printf("Consumer received: %s\n", shared_stuff->some_text);

            if (strncmp(shared_stuff->some_text, "end", 3) == 0) {
                break;
            }

            shared_stuff->written = 0;
        }

        if (shmdt(shared_memory) == -1) {
            perror("shmdt (consumer)");
            exit(EXIT_FAILURE);
        }

        exit(EXIT_SUCCESS);
    } else {
        // Parent process: Producer
        char buffer[TEXT_SZ];

        while (1) {
            printf("Enter some text: ");
            if (fgets(buffer, TEXT_SZ, stdin) == NULL) {
                perror("fgets");
                break;
            }

            // Remove newline character
            buffer[strcspn(buffer, "\n")] = '\0';

            strncpy(shared_stuff->some_text, buffer, TEXT_SZ);
            shared_stuff->written = 1;

            printf("Producer sent: %s\n", shared_stuff->some_text);

            if (strncmp(buffer, "end", 3) == 0) {
                break;
            }

            while (shared_stuff->written == 1) {
                sleep(1); // Wait for consumer
            }
        }

        // Wait for consumer to finish
        wait(NULL);

        // Detach from shared memory but do NOT remove it
        if (shmdt(shared_memory) == -1) {
            perror("shmdt (producer)");
            exit(EXIT_FAILURE);
        }

        printf("Detached from shared memory. Shared memory NOT removed.\n");
        exit(EXIT_SUCCESS);
    }
}


```




## OUTPUT


![Screenshot from 2025-05-24 08-57-49](https://github.com/user-attachments/assets/983a145f-8326-4259-8411-ad519458a502)


# RESULT:
The program is executed successfully.
