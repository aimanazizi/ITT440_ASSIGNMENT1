#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

#define MAX_CHILDREN 5

int children[MAX_CHILDREN];
int num_children = 0;
int pipes[MAX_CHILDREN][2];

void parent_handler(int sig) {
    char msg[100];
    printf("Enter a message to send to the child processes: ");
    fgets(msg, 100, stdin);
    for (int i = 0; i < num_children; i++) {
        write(pipes[i][1], msg, sizeof(msg));
    }
}

void child_handler(int i) {
    char msg[100];
    while (1) {
        read(pipes[i][0], msg, sizeof(msg));
        printf("Child %d received message: %s", i, msg);
    }
}

void interrupt_handler(int sig) {
    printf("\nInterrupt received.Thankyou for wishes. Exiting program.\n");
    for (int i = 0; i < num_children; i++) {
        kill(children[i], SIGKILL);
    }
    exit(0);
}

int main() {
    signal(SIGINT, interrupt_handler);
    signal(SIGUSR1, parent_handler);
    for (int i = 0; i < MAX_CHILDREN; i++) {
        if (num_children >= MAX_CHILDREN) break;
        pid_t pid = fork();
        if (pid == -1) {
            perror("fork");
            exit(1);
        }
        if (pid == 0) {
            child_handler(i);
            exit(0);
        }
        children[num_children] = pid;
        pipe(pipes[num_children]);
        num_children++;
    }
    printf("Enter HARI RAYA message to 5 of your friend.\n", num_children);
    printf("Press Ctrl+C to exit when you reach FRIEND 4.\n");
    while (1) {
        pause();
    }
    return 0;
}
