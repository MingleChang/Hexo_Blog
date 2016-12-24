---
title: Socket套接字API
date: 2016-12-24 11:52:12
tags: [Socket]
---

###IPv4套接字地址结构  
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

###IPv6套接字地址结构  
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

###通用套接字地址结构  
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

###字节排序函数  
```
#include <sys/_endian.h>
返回：网络字节序的值
#define htons(x)	__DARWIN_OSSwapInt16(x)
#define htonl(x)	__DARWIN_OSSwapInt32(x)
返回：主机字节序的值
#define ntohs(x)	__DARWIN_OSSwapInt16(x)
#define ntohl(x)	__DARWIN_OSSwapInt32(x)
```

###字节操纵函数  
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

###inet_aton、inet_addr和inet_ntoa函数  
```
#include <arpa/inet.h>
int		 inet_aton(const char *strptr, struct in_addr *addrptr);
in_addr_t	 inet_addr(const char *strptr);
char		*inet_ntoa(struct in_addr inaddr);
```

inet_aton将strptr所指C字符串转换成一个32位的网络字节序二进制值，并通过指针addrptr来存储，若成功则返回1，否这返回0。  
inet_addr进行相同的转换，返回的值为32位的网络字节序二进制值，出错时返回INADDR_NONE常值。该函数存在一个问题：  255.255.255.255不能有该函数处理，因为它的二进制值用来指示该函数失败。  
inet_ntoa函数将一个32位的网络二进制IPv4地址转换成相应的点分十进制数串。

###inet_pton和inet_ntop函数  
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

