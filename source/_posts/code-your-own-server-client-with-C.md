---
title: Code Your Own Server/Client Software With C socket APIs
date: 2019-02-15 23:39:07
tags: [linux, socket, C]
---

In this article I'm gonna show you how to code a simple echo server and corresponding client from scratch. 

## Prerequisite

To achieve the goal, we should first learn about some basic C APIs.

### Commonly Seen Structures

#### 1. `struct hostent`

`hostent` is abbreviation of `host entry`. Its instance is usally used to store the return value of `gethostbyname()` function. It's imported by including `<netdb.h>`

This is its definition:
```C
struct hostent{
    char * h_name;
    char ** h_aliases;
    short h_addrtype;
    short h_length;
    char ** h_addr_list;
    #define h_addr h_addr_list[0];
};
```

#### 2. `struct in_addr`

This data structure contains just one member `in_addr_t s_addr;`. Its header file is `<arpa/inet.h>`

Its declaration is as follows:
```C
struct in_addr {
    in_addr_t s_addr;
};
```
The type `in_addr_t` is in most cases alias of 32 bits `unsigned int`.

#### 3. `struct sockaddr_in`

Before introducing this structure, you should scan through definition below:
```C
struct sockaddr {
    unsigned  short  sa_family;     /* address family, AF_xxx */
    char  sa_data[14];                 /* 14 bytes of protocol address */
}
```

Structure above usually serves as parameter of functions `bind`, `connect`, `recvfrom` and `sendto`, etc. Its second member is not much readable for human, so developers introduced its equivalent structure. Cast between their pointers is secure.

Let's look at the equivalent structure's definition:
```C
struct sockaddr_in {
    short int sin_family; /* Address family */
    unsigned short int sin_port; /* Port number */
    struct in_addr sin_addr; /* Internet address */
    unsigned char sin_zero[8]; /* Same size as struct sockaddr */
}
```

Code above suggests that `struct sockaddr` in fact contains information about address-family, port and address. 

The header for `sockaddr` is `<sys/socket.h>`, and `<netinet/in.h>` for structure `sockaddr_in`.

### Commonly Seen Functions

#### One Parameter Functions

- `gethostbyname` : Literally understood.
- `inet_ntoa`: Transfer `in_addr` into human-readable format.
- `inet_addr`: Do reverse transformation of `inet_ntoa`.
- `htonl`: Host-to-Net Long conversion.
- `htons`: Host-to-Net Short conversion.
- `close`: Close the socket handle.


`gethostbyname`, `inet_ntoa`, `inet_addr`, `htonl` and `htons` all take just one parameter. 

Here `htox` functions mainly focus on adapting endianness, while `inet_xxxx` functions on transformation between machine presentation and human-readable format.

#### Multi-parameter Functions

- `socket`: Create socket instance.
- `connect`
- `send`
- `recv`
- `listen`
- `accept`

These functions take multiple parameters. If you compare each one with another, you can reveal a pattern of parameters for them all. Here I won't talk too much about this.

If you are struggling figuring out the meanings of terms like "send", "listen" and so on, I recommend you to first learn some basics about [TCP/IP](https://searchnetworking.techtarget.com/definition/TCP-IP) and socket.





## Hands-on Coding

[Here](https://github.com/Leegenux/C_socket_client_server) you can preview the result we are going to produce.



### nslookup

First, let's start with retrieving basic information using system-provided APIs. Say getting IP of a given URL. 

In Unix-like systems, there is tool named `nslookup` which does the job mentioned above. Here is an example:

```
$ nslookup google.com
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
Name:   google.com
Address: 172.217.160.110
Name:   google.com
Address: 2404:6800:4008:802::200e
```

We can code our own simpler version using APIs provided by operating system.

According to the prerequisite section, we can use the `gethostbyname` to finish the code.



First part. The includes :

```C
#include <stdio.h>          /* stderr, stdout */
#include <netdb.h>          /* hostent struct, gethostbyname() */
#include <arpa/inet.h>      /* inet_ntoa() to format IP address */
#include <netinet/in.h>     /* in_addr structure *
```

Then here comes the main body :

```C
int main(int argc, char **argv) {
    struct hostent *host;     /* host information */
    struct in_addr h_addr;    /* Internet address */
    if (argc != 2) {
        fprintf(stderr, "USAGE: nslookup <inet_address>\n");
        return 1;
    }
    if ((host = gethostbyname(argv[1])) == NULL) {
        fprintf(stderr, "(mini) nslookup failed on '%s'\n", argv[1]);
        return 1;
    }
    h_addr.s_addr = *((unsigned long *) host->h_addr_list[0]);
    fprintf(stdout, "%s\n", inet_ntoa(h_addr));
    return 0;
}
```

`gethostbyname` takes a C-string parameter, and returns the information in the format of `struct hostent`. The member `h_addr_list` stores all results we want. To make things simpler, we just fetch and print the first IP we get. If no result is available, `gethostbyname` returns `NULL`.



### tcp-client

First part is still includes:

```C
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
```

Then the main function:

```C
#define BUFFSIZE 32

int main(int argc, char *argv[]) {
```

Check the command line parameters and print the help-string if necessary: 

```C
// Check the command line parameters
if (argc != 4) {
    fprint(stderr, "Usage: tcp-client <server-ip> <port> <data>\n4)
}
```

Then create the socket, the first parameter of `socket` is often set to `PF_INET` or `AF_INET`. Note that in Linux implementation, `AF_INET` is totally same as `PF_INET`. The second and third parameters are type and protocol. In this condition, we set `type = SOCK_STREAM` and `protocol = IPPROTO_TCP`. With these setups, we create a tcp socket. And the function returns a socket handle for later use, which normally should be greater than 0.

```C
// Create the socket
int sock;
if ((sock = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP)) < 0) {
    fprintf(stderr, "Failed to create socket\n");
    return 1;
}
```

After that, let's construct the server address object with command line parameters. The member `sin_family` is usually either set to `AF_INET` or ` PF_INET` . And the member `sin_addr.s_addr` is actually integer as inner presentation. But you can easily get the presentation using function `inet_addr`. The last member `sin_zero` is intended for main memory alignment optimization, and should be set 0 manually.

```C
// Construct server address object
struct sockaddr_in echoServer;
echoServer.sin_family = AF_INET;
echoServer.sin_addr.s_addr = inet_addr(argv[1]);
echoServer.sin_port = htons(atoi(argv[2]));
memset(&(echoServer.sin_zero), 0, sizeof(echoServer.sin_zero));
```

Then, we shall try to establish the connection and send the data. Look closely at the parameters of two functions `connect` and `send`, as well as how the socket is related.

```C
// Establish connection
if (connect(sock, 
    (struct sockaddr*) &echoServer,
    sizeof(echoServer)) < 0) {
    fprintf(stderr, "Failed to connect.\n");
    return 1;
}

// Send the data
unsigned char dataLen = strlen(argv[3]);
if (send(sock, argv[3], dataLen, 0) != dataLen) {
    fprintf(stderr, "Data check failure.\n");
    return 1;
}
```

At last, if everything goes fine, we are now at the final step. The return value of the `recv` function is the number of the bytes received. It is not guaranteed that `recv` function can get the whole echo result through one time execution. What is guaranteed is that normally when there are data to receive, `packSize` will not be 0. 

You should answer yourself why `BUFFSIZE - 1` in the third parameter position.

```C
// Receive the echo
short received = 0;
char buffer[BUFFSIZE];

printf("Echo: \n");
while (received < dataLen) {
    short packSize = 0;
    if ((packSize = recv(sock, buffer, BUFFSIZE-1, 0)) < 1) {
        fprintf(stderr, "Failed to receive byte.");
        return 1;
    }
    received += packSize;
}

// Print the echo
buffer[received] = '\0';
printf("%s\n", buffer);

return 0;
}
```



### tcp-server

Still the old rules --- the includes:

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#include <netinet/in.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
```

And in the `main` function, check the arguments:

```c
#define BUFFSIZE 32
#define MAXPENDING 5

int main(int argc, char *argv[]) {
    // Check the arguments
    if (argc != 2) {
        fprintf(stderr, "Usage: tcp-server <port>\n");
        return 1;
    }
```

Create the socket. The arguments of function `socket` and setup of `echoServer` have been explained in section `tcp-client`. 

```c
    // Create main socket
    int serverSock;

    // Create TCP socket
    if ((serverSock = socket(PF_INET, SOCK_STREAM, IPPROTO_TCP)) < 0) {
        fprintf(stderr, "Failed to create socket\n");
        return 1;
    }

    // Set main socket address
    struct sockaddr_in echoServer;
    echoServer.sin_family = AF_INET;
    echoServer.sin_addr.s_addr = htonl(INADDR_ANY);
    echoServer.sin_port = htons(atoi(argv[1]));
    memset(&(echoServer.sin_zero), 0, sizeof(echoServer.sin_zero));
```

After the socket is created, we should bind it to some port of the machine and listen to it so as to work as server. Note that the `bind` function has same parameters as `connect` function. There is an important attribute for socket when listening. That is the number of pending requests. Here the second parameter of `listen` stands for that attribute. 

```c
    // Bind the main socket
    if (bind(serverSock,
        (struct sockaddr*) &echoServer,
        sizeof(echoServer)) < 0) {
        fprintf(stderr, "Failed to bind.\n");
        return 1;
    }
    
    // Listen to the main socket
    if (listen(serverSock, MAXPENDING) < 0) {
        fprintf(stderr, "Failed to listen on server socket.\n");
        return 1;
    }
```

At last, after work done above, we are truly "serving" when executing code following. `accept` function fetches the connection on the top of the pending list out and starts a conversation. It returns the handle of the client socket. After that, we ought to do echo. It's just a loop of `receive` and `send`.

Answer yourself why `BUFFSIZE` and not `BUFFSIZE - 1`.

```c
    // Handle the requests
    int clientSock;
    struct sockaddr_in echoClient;
    unsigned int clientLen = sizeof(echoClient);

    while (1) {
        if ((clientSock = accept(serverSock,
                            (struct sockaddr*) &echoClient,
                            &clientLen)) < 0) {
            fprintf(stderr, "Failed to accept the connection.\n");
            return 1;
        }
        printf("Client connected: %s\n", inet_ntoa(echoClient.sin_addr));

        // Do the echo
        char buffer[BUFFSIZE];
        int received = -1;
        if ((received = recv(clientSock, buffer, BUFFSIZE, 0)) < 0) {
            fprintf(stderr, "Failed to receive data.\n");
            return 1;
        }

        while (received > 0) {
            if (send(clientSock, buffer, received, 0) != received) {
                fprintf(stderr, "Failed to echo back.\n");
                return 1;
            }
            if ((received = recv(clientSock, buffer, BUFFSIZE, 0)) < 0) {
                fprintf(stderr, "Failed to receive data.\n");
            }
        }
        close(clientSock);
    }

    return 0; // Never get here
}
```



If you have finished the code above, you can build 3 executables. Given that you name them after the sources, you can test your server and client code like this:

open one terminal and type `./tcp-server 12334`. 

In another terminal, type `./tcp-client 127.0.0.1 12334 'Hello world' && ./tcp-client 127.0.0.1 12334 'I like you'`

```
$ ./tcp-server 12334
Client connected: 127.0.0.1
Client connected: 127.0.0.1
```

```
$ ./tcp-client 127.0.0.1 12334 'Hello world' && ./tcp-client 127.0.0.1 12334 'I like you'                                 
Echo:
Hello world
Echo:
I like you
```

Above shows the result of execution on my laptop, and yours should be alike.
