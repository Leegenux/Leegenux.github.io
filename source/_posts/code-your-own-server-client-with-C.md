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

This is its declaration:
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

Before introducing this structure, you should scan through declaration below:
```C
struct sockaddr {
    unsigned  short  sa_family;     /* address family, AF_xxx */
    char  sa_data[14];                 /* 14 bytes of protocol address */
}
```

Structure above usually serves as parameter of functions `bind`, `connect`, `recvfrom` and `sendto`, etc. Its second member is not much readable for human, so developers introduced its equivalent structure. Cast between their pointers is secure.

Let's look at the equivalent structure's declaration:
```C
struct sockaddr_in {
    short int sin_family; /* Address family */
    unsigned short int sin_port; /* Port number */
    struct in_addr sin_addr; /* Internet address */
    unsigned char sin_zero[8]; /* Same size as struct sockaddr */
}
```

Code above suggests that `struct sockaddr` in fact contains information about address-family, port and address. 

The header for `sockaddr` is `<sys/socket.h>`, `<netinet/in.h>` for `sockaddr_in`.

### Commonly Seen Functions

#### One Parameter Functions

- `gethostbyname` : Literally understood.
- `inet_ntoa`: Transfer `in_addr` into human-readable format.
- `inet_addr`: Do reverse transformation of `inet_ntoa`.
  -`htonl`: Host-to-Net Long conversion.
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

If you are struggling figuring out the meanings of terms like "send", "listen" and so on, I recommend you to first learn some basics about [IP/TCP](https://www.studytonight.com/computer-networks/tcp-ip-reference-model) and [socket](https://medium.com/@chathuranga94/introduction-to-socket-io-600025322cd2).





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

`gethostbyname` takes a C-string parameter, returns the information in the format of `struct hostent`. The member `h_addr_list` stores all results we want. To make things simpler, we just fetch and print the first IP we get. If no result is available, `gethostbyname` returns NULL.



### tcp-client







**...To Be Continued**