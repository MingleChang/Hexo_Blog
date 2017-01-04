---
title: Socket套接字API
date: 2016-12-24 11:52:12
tags: [Socket]
---

**IPv4套接字地址结构**

```
#include <netinet/in.h>
/*
 * Internet address (a structure for historical reasons)
 */
struct in_addr {
    in_addr_t s_addr;
};
/*
 * Socket address, internet style.
 */
struct sockaddr_in {
    __uint8_t	sin_len;
    sa_family_t	sin_family;
    in_port_t	sin_port;
    struct	in_addr sin_addr;
    char		sin_zero[8];
};
```

**IPv6套接字地址结构**
  
```
#include <netinet6/in6.h>
/*
 * IPv6 address
 */
struct in6_addr {
    union {
        __uint8_t   __u6_addr8[16];
        __uint16_t  __u6_addr16[8];
        __uint32_t  __u6_addr32[4];
    } __u6_addr;			/* 128-bit IP6 address */
};
struct sockaddr_in6 {
    __uint8_t	sin6_len;	/* length of this struct(sa_family_t) */
    sa_family_t	sin6_family;	/* AF_INET6 (sa_family_t) */
    in_port_t	sin6_port;	/* Transport layer port # (in_port_t) */
    __uint32_t	sin6_flowinfo;	/* IP6 flow information */
    struct in6_addr	sin6_addr;	/* IP6 address */
    __uint32_t	sin6_scope_id;	/* scope zone index */
};
```

**通用套接字地址结构**  

```
#include <sys/socket.h>
/*
 * [XSI] Structure used by kernel to store most addresses.
 */
struct sockaddr {
    __uint8_t	sa_len;		/* total length */
    sa_family_t	sa_family;	/* [XSI] address family */
    char		sa_data[14];	/* [XSI] addr value (actually larger) */
};
```

**字节排序函数**  

```
#include <sys/_endian.h>
返回：网络字节序的值
#define htons(x)	__DARWIN_OSSwapInt16(x)
#define htonl(x)	__DARWIN_OSSwapInt32(x)
返回：主机字节序的值
#define ntohs(x)	__DARWIN_OSSwapInt16(x)
#define ntohl(x)	__DARWIN_OSSwapInt32(x)
```

**字节操纵函数**  

```
#include <strings.h>
void	 bzero(void *, size_t)
void	 bcopy(const void *, void *, size_t)
int	 bcmp(const void *, const void *, size_t)
```

bzero把目标字节串中指定数目的字节置为0，我们经常使用该函数来把一个套接字地址结构初始化为0。  
bcopy将指定数目的字节从源字节串复制到目标字节串。  
bcmp比较两个任意的字节串，若相同则返回值为0，否则返回值为非0。  

```
#include <string.h>
void	*memset(void *__b, int __c, size_t __len);
void	*memcpy(void *__dst, const void *__src, size_t __n);
int	 memcmp(const void *__s1, const void *__s2, size_t __n);
```

memset把目标字节串指定数目的字节置为__c。  
memcpy类似bcopy，不过两个指针参数的顺序是相反的，当源字节串与目标字节串重叠时，bcopy能够正确处理，但是memcpy的操作结果却不可知。  
memcmp比较两个任意的字节串，若相同则返回0，否则返回一个非0，是大于0还是小于0则取决于第一个不等的字节。

**inet_aton、inet_addr和inet_ntoa函数** 
 
```
#include <arpa/inet.h>
int		 inet_aton(const char *strptr, struct in_addr *addrptr);
in_addr_t	 inet_addr(const char *strptr);
char		*inet_ntoa(struct in_addr inaddr);
```

inet_aton将strptr所指C字符串转换成一个32位的网络字节序二进制值，并通过指针addrptr来存储，若成功则返回1，否这返回0。  
inet_addr进行相同的转换，返回的值为32位的网络字节序二进制值，出错时返回INADDR_NONE常值。该函数存在一个问题：  255.255.255.255不能有该函数处理，因为它的二进制值用来指示该函数失败。  
inet_ntoa函数将一个32位的网络二进制IPv4地址转换成相应的点分十进制数串。

**inet_pton和inet_ntop函数**  

```
#include <arpa/inet.h>
int		 inet_pton(int family, const char *strptr, void *addrptr);
const char	*inet_ntop(int family, const void *addrptr, char *strptr, socklen_t len);
```

inet_pton尝试转换由strptr指针所指的字符串，并通过addrptr指针存放二进制结果。若成功则返回值为1，若输入不是有效的表达格式则为0，若出错则为-1  
inet_ntop是进行相反的转换，从数值格式（addrptr）转换到表达式（strptr）。len参数是目标存储单元的大小，以免该函数溢出其调用者的缓冲区。为有助于指定这个大小，在头文件中有如下定义：
`#define INET_ADDRSTRLEN 16`
`#define	INET6_ADDRSTRLEN	46`  
如果len太小，不足以容纳表达格式结果（包括结尾的空字符），那么返回一个空指针，并置errno为ENOSPC。  
inet_ntop函数的strptr参数不可以是一个空指针。调用者必须为目标存储单元分配内存并指定其大小。调用成功时，这个指针就是该函数的返回值。

**socket函数**  
为了执行网络I/O，一个进程必须做的第一件事就是调用socket函数，指定期望的通信协议类型。

```
#include <sys/socket.h>
int	socket(int family, int type, int protocol);
```

family参数指名协议族，该参数也往往被称为协议域。  

|family|说明|
|:---:|:---:|
|AF_INET|IPv4协议|
|AF_INET6|IPv6协议|
|AF_LOCAL|Unix域协议|
|AF_ROUTE|路由套接字|
|AF_KEY|密钥套接字|

type参数指名套接字类型。

|type|说明|
|:---:|:---:|
|SOCK_STREAM|字节流套接字|
|SOCK_DGRAM|数据报套接字|
|SOCK_SEQPACKET|有序分组套接字|
|SOCK_RAW|原始套接字|

protocol参数应设为某个协议类型常值，或者设为0，以选择给定的family和type组合的系统默认值；

|protocol|说明|
|:---:|:---:|
|IPPROTO_TCP|TCP传输协议|
|IPPROTO_UDP|UDP传输协议|
|IPPROTO_SCTP|SCTP传输协议|

并非所有套接字family与type组合都是有效的，下表给出了一些有效的组合和对应的真正协议。其中标为“是”的项也是有效的，但还没有找到便捷的缩略词。而空白项则是无效组合。

||AF_INET|AF_INET6|AF_LOCAL|AF_ROUTE|AF_KEY|
|:---:|:---:|:---:|:---:|:---:|:---:|
|SOCK_STREAM|TCP/SCTP|TCP/SCTP|是|||
|SOCK_DGRAM|UDP|UDP|是|||
|SOCK_SEQPACKET|SCTP|SCTP|是|||
|SOCK_RAW|IPv4|IPv6||是|是|

socket函数在成功时返回一个小的非负整数值，它与文件描述符类似，我们把它称为套接字描述符(socket descrptor)，简称sockfd。为了得到这个套接字描述符，我们只是指定了协议族(IPv4、IPv6和Unix)和套接字类型(字节流、数据报或原始套接字)。我们并没有指定本地协议地址和远程协议地址。

**connect函数**  
TCP客户端用connect函数来建立与TCP服务器的连接。

```
#include <sys/socket.h>
int	connect(int sockfd, const struct sockaddr *servaddr, socklen_t addrlen);
```

sockfd是由socket函数返回的套接字描述符，第二个、第三个参数分别是一个指向套接字地址结构的指针和该结构的大小，套接字地址结构必须含有服务器的IP地址和端口号。

**bind函数**  
bind函数把一个本地协议地址赋予一个套接字，对于网际网协议，协议地址是32位的IPv4地址或128位的IPv6地址与16位的TCP或UDP端口号的组合。

```
#include <sys/socket.h>
int	bind(int sockfd, const struct sockaddr *myaddr, socklen_t addrlen)
```

第二个参数是一个指向特定协议的地址结构的指针，第三个参数是该地址结构的长度。对于TCP，调用bind函数可以指定一个端口号，或指定一个IP地址，也可以两者都指定，还可以都不指定。 
 
|IP地址|断开|结果|
|:---:|:---:|:---|
|通配地址|0|内核选择IP地址和端口|
|通配地址|非0|内核选择IP地址，进程指定端口|
|本地IP地址|0|进程指定IP地址，内核选择端口|
|本地IP地址|非0|进程指定IP地址和端口|

**listen函数**

```
#include <sys/socket.h>
int	listen(int sockfd, int backlog);//返回：若成功则为0，若出错则为-1
```

listen函数仅由TCP服务器调用，他做两件事情：  
1. 当socket函数创建一个套接字时，它被假设为一个主动套接字，也就是说，它是一个将调用connect发起连接的客户套接字。listen函数把一个未连接的套接字转换成一个被动套接字，指示内核应接受指向该套接字的连接请求。调用listen导致套接字从CLOSED状态转喊到LISTEN状态。  
2. 第二个参数规定了内核应该为相应套接字排队的最大连接个数。
  
本函数通常应该在调用socket和bind这两个函数之后，并在调用accept函数之前调用。

**accept函数**  
accept函数由TCP服务器调用，用于从已完成连接队列队头返回下一个已完成连接。如果已完成连接队列为空，那么进程被投入睡眠。

```
#include <sys/socket.h>
int	accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);
```

返回值：如果成功返回非负描述符，若出错则为-1  
参数cliaddr和addrlen用来返回已连接的对端进程的协议地址。如果accept成功，那么其返回值是由内核自动生成的一个全新描述符，代表与所返回客户的TCP链接。

**fork和exec函数**

```
#include <unistd.h>
pid_t	 fork(void);
```
返回值：在子进程中为0，在父进程中为子进程ID，若出错则为-1  
fork在子进程返回0而不是父进程的进程ID的原因在于：任何子进程只有一个父进程，而且子进程总是可以通过调用`getppid`取得父进程的进程ID。相反，父进程可以有许多子进程，而且无法获取各个子进程的进程ID。如果父进程想要跟踪所有子进程的进程ID，那么它必须记录每次调用fork的返回值。  
父进程中调用fork之前打开的所有描述符在fork返回之后由子进程分享。  
fork有两个典型用法：  
1） 一个进程创建一个自身的副本，这样每个副本都可以在另一个副本执行其他任务的同时处理各自的某个操作。这是网络服务器的典型用法。  
2) 一个进程想要执行另一个程序。既然创建新进程的唯一办法是调用fork，该进程于是首先调用fork创建一个自身的副本，然后其中一个副本（通常为子进程）调用`exec`把自身替换成新的程序。这是诸如shell之类程序的典型用法。  

```
#include <unistd.h>
int	 execl(const char * __path, const char * __arg0, ...);
int	 execv(const char * __path, char * const * __argv);
int	 execle(const char * __path, const char * __arg0, ...);

int	 execve(const char * __file, char * const * __argv, char * const * __envp);
int	 execlp(const char * __file, const char * __arg0, ...);
int	 execvp(const char * __file, char * const * __argv);
```
返回值：若成功则不返回，若出错则为-1  
这些函数只在出错时才返回到调用者。否则，控制将被传递给新程序的起始点，通常就是main函数。  

**close函数**  

```
#include <unistd.h>
int	 close(int sockfd);
```
返回值：若成功则为0，若出错则为-1  
close函数用来关闭套接字，并终止TCP连接。  
close一个TCP套接字的默认行为是把该套接字标记成已关闭，然后立即返回到调用进程。

**getsockname和getpeername函数**  
这两个函数返回与某个套接字关联的本地协议地址(getsockname)，或者返回与某个套接字关联的外地协议地址(getpeername)。  

```
#include <sys/socket.h>
int	getsockname(int sockfd, struct sockaddr * localaddr, socklen_t * addrlen);
int	getpeername(int sockfd, struct sockaddr * localaddr, socklen_t * addrlen);
```
