# Linux-IPC--Pipes
Linux-IPC-Pipes


# Ex03-Linux IPC - Pipes

# AIM:
To write a C program that illustrate communication between two process using unnamed and named pipes

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - pipe(), fifo()

### Step 3:

Testing the C Program for the desired output. 

# PROGRAM:

## C Program that illustrate communication between two process using unnamed pipes using Linux API system calls


#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/wait.h>

void server(int rfd, int wfd);
void client(int wfd, int rfd);

int main() {
    int p1[2], p2[2];
    pid_t pid;

    if (pipe(p1) == -1 || pipe(p2) == -1) {
        perror("pipe");
        exit(1);
    }

    pid = fork();

    if (pid < 0) {
        perror("fork");
        exit(1);
    }

    if (pid == 0) {
        // Child process - Server
        close(p1[1]);
        close(p2[0]);

        server(p1[0], p2[1]);

        close(p1[0]);
        close(p2[1]);
        exit(0);
    } else {
        // Parent process - Client
        close(p1[0]);
        close(p2[1]);

        client(p1[1], p2[0]);

        close(p1[1]);
        close(p2[0]);
        wait(NULL);
    }

    return 0;
}

void server(int rfd, int wfd) {
    char fname[2000];
    char buff[2000];
    int n, fd;

    n = read(rfd, fname, sizeof(fname) - 1);
    if (n <= 0) {
        write(wfd, "Error reading filename\n", 23);
        return;
    }

    fname[n] = '\0';

    fd = open(fname, O_RDONLY);

    if (fd < 0) {
        write(wfd, "Can't open file\n", 16);
        return;
    }

    while ((n = read(fd, buff, sizeof(buff))) > 0) {
        write(wfd, buff, n);
    }

    close(fd);
}

void client(int wfd, int rfd) {
    char fname[2000];
    char buff[2000];
    int n;

    printf("Enter file name: ");
    scanf("%s", fname);

    write(wfd, fname, strlen(fname));
    close(wfd);

    while ((n = read(rfd, buff, sizeof(buff))) > 0) {
        write(1, buff, n);
    }
}


## OUTPUT

<img width="828" height="375" alt="image" src="https://github.com/user-attachments/assets/655242c7-c917-4476-82bd-120e76924da2" />

## C Program that illustrate communication between two process using named pipes using Linux API system calls

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <string.h>
#include <sys/wait.h>

#define FIFO_FILE "/tmp/my_fifo"
#define FILE_NAME "hello.txt"

void server();
void client();

int main() {
    pid_t pid;

    if (mkfifo(FIFO_FILE, 0666) == -1) {
        perror("FIFO already exists or cannot create");
    }

    pid = fork();

    if (pid > 0) {
        server();
        wait(NULL);
        unlink(FIFO_FILE);
    } 
    else if (pid == 0) {
        client();
    } 
    else {
        perror("Fork failed");
        exit(EXIT_FAILURE);
    }

    return 0;
}

void server() {
    int fifo_fd, file_fd;
    char buffer[1024];
    ssize_t bytes_read;

    file_fd = open(FILE_NAME, O_RDONLY);
    if (file_fd == -1) {
        perror("Error opening hello.txt");
        exit(EXIT_FAILURE);
    }

    fifo_fd = open(FIFO_FILE, O_WRONLY);
    if (fifo_fd == -1) {
        perror("Error opening FIFO");
        close(file_fd);
        exit(EXIT_FAILURE);
    }

    while ((bytes_read = read(file_fd, buffer, sizeof(buffer))) > 0) {
        write(fifo_fd, buffer, bytes_read);
    }

    close(file_fd);
    close(fifo_fd);
}

void client() {
    int fifo_fd;
    char buffer[1024];
    ssize_t bytes_read;

    fifo_fd = open(FIFO_FILE, O_RDONLY);
    if (fifo_fd == -1) {
        perror("Error opening FIFO");
        exit(EXIT_FAILURE);
    }

    while ((bytes_read = read(fifo_fd, buffer, sizeof(buffer))) > 0) {
        write(STDOUT_FILENO, buffer, bytes_read);
    }

    close(fifo_fd);
}



## OUTPUT
<img width="597" height="337" alt="WhatsApp Image 2026-05-21 at 08 22 37" src="https://github.com/user-attachments/assets/77022dd5-52a6-46ef-ac1a-e7a8356ef480" />


# RESULT:
The program is executed successfully.
