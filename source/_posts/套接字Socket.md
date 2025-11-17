---
abbrlink: '0'
---
# 一、套接字
## 1.1 认识套接字
所谓套接字，它是通过标准的UNIX文件描述符和其他的程序通讯的一个方法。
## 1.2 Socket描述符
一个文件描述符只是一个简单的整型数值，代表一个被打开的文件（这里的文件是广义的文件，并不止代表不同的磁盘文件，他可以代表一个网络上的连接，一个先进先出队列，一个终端显示屏幕，以及其他一切）。在UNIX系统中任何·东西都是文件！所以你想通过Internet和另外一个程序通讯的话，将会是通过一个文件来描述符实现的。
首先调用系统函数socket(),它返回一个套接字描述符，你可以通过则个描述符进行一些操作：send() & recv()。write()和read()是可以对套接字描述符进行操作的，但是，通过使用用 send() 和 recv() 函数，你可以对网络数据的传输进行更好的控制！
# 二、套接字结构
## 2.1基本结构
struct sockaddr 这个结构用来存储套接字地址
```
struct sockaddr {
unsigned short sa_family; /* address族, AF_xxx */
char sa_data[14]; /* 14 bytes的协议地址 */
};
```
sa_family 一般来说，都是 “AFINET”。
sa_data 包含了一些远程电脑的地址、端口和套接字的数目，它里面的数据是杂溶在一切的。
为了处理 struct sockaddr， 程序员建立了另外一个相似的结构 struct sockaddr_in：
```
struct sockaddr_in (“in” 代表 “Internet”)
struct sockaddr_in {
short int sin_family; /* Internet地址族 */
unsigned short int sin_port;/* 端口号 */
struct in_addr sin_addr;/* Internet地址 */
unsigned char sin_zero[8];/* 添0（和struct sockaddr一样大小）*/

};
``` 
注意 sin_zero[8] 是为了是两个结构在内存中具有相同的尺寸，使用 sockaddr_in 的时
候要把 sin_zero 全部设成零值（使用 bzero()或 memset()函数）。而且，有一点很重要，就是一个指向 struct sockaddr_in 的指针可以声明指向一个 sturct sockaddr 的结构。所以虽然socket() 函数需要一个 structaddr * ，你也可以给他一个 sockaddr_in * 。注意在 structsockaddr_in 中，sin_family 相当于 在 struct sockaddr 中的 sa_family，需要设成“AF_INET”。
最后一定要保证 sin_port 和 sin_addr 必须是网络字节顺序.
## 2.2转换函数
如果你想将一个短型数据从主机字节顺序转换到网络字节顺序的话，有这样一个函
数：它是以“h”开头的（代表“主机”）；紧跟着它的是“to”，代表“转换到”；然后是“n”
代表“网络”；最后是“s”，代表“短型数据”。H-to-n-s，就是 htons() 函数（可以使用 Host to Network Short 来助记）
```
l htons()——“Host to Network Short //主机字节顺序转换为网络字节顺序（对无符号
短型进行操作 4 bytes）

l htonl()——“Host to Network Long //主机字节顺序转换为网络字节顺序（对无符

号长型进行操作 8 bytes）

l ntohs()——“Network to Host Short //网络字节顺序转换为主机字节顺序（对无符

号短型进行操作 4 bytes）

l ntohl()——“Network to Host Long //网络字节顺序转换为主机字节顺序（对无符

号长型进行操作 8 bytes）
```
## 2.3 ip地址转换
假设你有一个 struct sockaddr_in sock，并且你的 IP 是 166.111.69.52 ，你想
把你的 IP 存储到 ina 中。你可以使用的函数： inet_addr() ，它能够把一个用数字和点表
示 IP 地址的字符串转换成一个无符号长整型。你可以像下面这样使用它：
```
sock.sin_addr.s_addr = inet_addr（“166.111.69.52”）;
```
# 三、套接字调用
## 3.1 socket函数
声明如下：
```
#include <sys/types.h>
#include <sys/socket.h>

int socket（int domain , int type , int protocol）;
```
domain 需要被设置为 “AF_INET”，就像上面的 struct sockaddr_in。type
参数告诉内核这个 socket 是什么类型，“SOCK_STREAM”或是“SOCK_DGRAM”。最后
只需要把 protocol 设置为 0 。
## 3.2bind函数
```
#include <sys/types.h>
#include <sys/socket.h>

int bind (int sockfd , struct sockaddr *my_addr , int addrlen)
```
sockfd 是由 socket()函数返回的套接字描述符。
my_addr 是一个指向 struct sockaddr 的指针，包含有关你的地址的信息：名称、
端口和 IP 地址。
addrlen 可以设置为 sizeof(struct sockaddr)


最后注意有关 bind()的是：有时候你并不一定要调用 bind()来建立网络连接。比如你只是想连接到一个远程主机上面进行通讯，你并不在乎你究竟是用的自己机器上的哪个端口进行通讯（比如 Telnet），那么你可以简单的直接调用 connect()函数，connect()将自动寻找出本地机器上的一个未使用的端口，然后调用 bind()来将其 socket 绑定到那个端口上。
## 3.3connect()函数

```
#include <sys/types.h>

#include <sys/socket.h>

int connect(int sockfd, struct sockaddr *serv_addr, int addrlen);
```
sockfd ：套接字文件描述符，由 socket()函数返回的。
serv_addr 是一个存储远程计算机的 IP 地址和端口信息的结构。
addrlen 应该是 sizeof(struct sockaddr)。
看如下代码片段：
```
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>

#define DEST_IP “166.111.69.52”
#define DEST_PORT 23

int main()
{
int sockfd ;

/* 将用来存储远程信息 */
struct sockaddr_in dest_addr ;

/* 注意在你自己的程序中进行错误检查！！ */
sockfd = socket（AF_INET, SOCK_STREAM, 0）;

/* 主机字节顺序 */
dest_addr.sin_family = AF_INET ;

/* 网络字节顺序，短整型 */
dest_addr.sin_port = htons（DEST_PORT）;
dest_addr.sin_addr.s_addr = inet_addr（DEST_IP）;

/* 将剩下的结构中的空间置 0 */
bzero(&(dest_addr.sin_zero), 8）;

/* 不要忘记在你的代码中对 connect()进行错误检查！！ */
connect(sockfd, (struct sockaddr *)&dest_addr, sizeof(struct sockaddr));

```

在面向连接的协议的程序中,服务器执行以下函数：
* 调用 socket()函数创建一个套接字。
* 调用 bind()函数把自己绑定在一个地址上。
* 调用 listen()函数侦听连接。
* 调用 accept()函数接受所有引入的请求。

调用 recv()函数获取引入的信息然后调用 send()回答。
## 3.4 listen函数
listen()函数是等待别人连接，进行系统侦听请求的函数。当有人连接你的时候，你有
两步需要做：通过 listen()函数等待连接请求，然后使用 accept()函数来处理。
```
#include <sys/socket.h>

int listen(int sockfd, int backlog);
```
sockfd 是一个套接字描述符，由 socket()系统调用获得。
backlog 是未经过处理的连接请求队列可以容纳的最大数目。
backlog 具体一些是什么意思呢？每一个连入请求都要进入一个连入请求队列，等待
listen 的程序调用 accept()（accept()函数下面有介绍）函数来接受这个连接。当系统还没有调用 accept()函数的时候，如果有很多连接，那么本地能够等待的最大数目就是 backlog 的数值。你可以将其设成 5 到 10 之间的数值（推荐）。
我们需要指定本地端口，因为我们是等待别人连接。所以，在listen()函数调用之前
，我们需要使用bind()函数来指定使用本地的哪一个端口数值。
## 3.5 accept()函数
调用该函数时，了解下大致过程：
>从很远的地方调用connect()来连接你的机器上某个端口
>

```
#include <sys/socket.h>

int accept(int sockfd, void *addr, int *addrlen);
```
* sockfd 是正在 listen() 的一个套接字描述符。
* addr 一般是一个指向 struct sockaddr_in 结构的指针；里面存储着远程连接过来的计算机的信息（比如远程计算机的 IP 地址和端口）。
* addrlen 是一个本地的整型数值，在它的地址传给 accept() 前它的值应该是
sizeof(struct sockaddr_in)；accept()不会在 addr 中存储多余 addrlen bytes 大小的数据。如果accept()函数在 addr 中存储的数据量不足 addrlen，则 accept()函数会改变 addrlen 的值来反应这个情况。
## 3.6 send()、recv()函数
```
#include <sys/types.h>
#include <sys/socket.h>

int send(int sockfd, const void *msg, int len, int flags);
```
* sockfd 是代表你与远程程序连接的套接字描述符。
* msg 是一个指针，指向你想发送的信息的地址。
* len 是你想发送信息的长度。
* flags 发送标记。一般都设为 0（你可以查看 send 的 man pages 来获得其他的参数值并且明白各个参数所代表的含义）
```
#include <sys/types.h>
#include <sys/socket.h>第 6 章 berkeley 套接字

int recv(int sockfd, void *buf, int len, unsigned int flags）
```
* sockfd 是你要读取数据的套接字描述符。
* buf 是一个指针，指向你能存储数据的内存缓存区域。
* len 是缓存区的最大尺寸。
* flags 是 recv() 函数的一个标志，一般都为 0 
recv(返回它真正收到的数据长度。（也就是存到buf中数据长度）如果返回-1则代表发生了错误（比如网络以外中断、对方关闭了套接字连接等），全局变量errno里面存储了错误代码。
## 3.7 sendto() recvfrom()
```
#include <sys/types.h>

#include <sys/socket.h>

int sendto（int sockfd, const void *msg, int len, unsigned int flags,

const struct sockaddr *to, int tolen）
```
**参数如下**：
* sockfd 是代表你与远程程序连接的套接字描述符。
* msg 是一个指针，指向你想发送的信息的地址。
* len 是你想发送信息的长度。
* flags 发送标记。一般都设为 0。（你可以查看 send 的 man pages 来获得其他的参
数值并且明白各个参数所代表的含义）
* to 是一个指向 struct sockaddr 结构的指针，里面包含了远程主机的 IP 地址和端口
数据。
* tolen 只是指出了 struct sockaddr 在内存中的大小 sizeof(struct sockaddr)。

```
#include <sys/types.h>
#include <sys/socket.h>

int recvfrom(int sockfd, void *buf, int len, unsigned int flags
struct sockaddr *from, int *fromlen);
```
sockfd 是你要读取数据的套接字描述符。
* buf 是一个指针，指向你能存储数据的内存缓存区域。
* len 是缓存区的最大尺寸。
* flags 是 recv() 函数的一个标志，一般都为 0 （具体的其他数值和含义请参考 recv()的 man pages）。
* from 是一个本地指针，指向一个 struct sockaddr 的结构（里面存有源 IP 地址和端口数）．
* fromlen 是一个指向一个 int 型数据的指针，它的大小应该是 sizeof（struct
sockaddr）．当函数返回的时候，formlen 指向的数据是 form 指向的 struct sockaddr 的实际
大小
## 3.8 shutdown()函数

```
#include <sys/socket.h>
int shutdown（int sockfd, int how）
```
  * how 可以取下面的值。0 表示不允许以后数据的接收操；1 表示不允许以后数据第 6 章 berkeley 套接字的发送操作；2 表示和 close()一样，不允许以后的任何操作（包括接收，发送数据）shutdown() 如果执行成功将返回 0，如果在调用过程中发生了错误，它将返回–1，全局变量 errno 中存储了错误代码
## 3.9 gethostname()函数

```
#include <unistd.h>
int gethostname(char *hostname, size_t size);
```
参数说明如下：
* hostname 是一个指向字符数组的指针，当函数返回的时候，它里面的数据就是本
地的主机的名字．
* size 是 hostname 指向的数组的长度．
* 函数如果成功执行，它返回 0，如果出现错误，则返回–1，全局变量 errno 中存储着错
误代码。

# 四、DNS
## 4.1 概念
DNS 是“Domain Name Service”（域名服务）的缩写。有了它，你就可以通过一个可读性非常强的因特网名字得到这个名字所代表的 IP 地址。转换为 IP地址后，你就可以使用标准的套接字函数（bind()，connect()，sendto()，或是其他任何需要使用的函数）。在这里，如果你输入命令：
```
telnet bbs.tsinghua.edu.cn
```
==Telnet 可以知道它需要连往 202.112.58.200。这就是通过 DNS 来实现的==
## 4.2 相关函数
```
#include <netdb.h>
struct hostent *gethostbyname(const char *name);
```
它返回了一个指向 struct hostent 的指针．Struct hostent 是这样定义的：

```
struct hostent {
	char *h_name;
	char **h_aliases;	
	int h_addrtype;
	int h_length;
	char **h_addr_list;
};
```
* h_name 是这个主机的正式名称。
* h_aliases 是一个以 NULL（空字符）结尾的数组，里面存储了主机的备用名称。
* h_addrtype 是返回地址的类型，一般来说是“AF_INET”。
* h_length 是地址的字节长度。
* h_addr_list 是一个以 0 结尾的数组，存储了主机的网络地址。
* h_addr - h_addr_list 数组的第一个成员．
gethostbyname() 返回的指针指向结构 struct hostent ，如果发生错误，它将会返回 NULL
## 4.3 DNS例程
```

#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <netdb.h>
#include <sys/types.h>
#include <netinet/in.h>

int main (int argc, char *argv[])
{
	struct hostent *h;
	/* 检测命令行中的参数是否存在 */
	if (argc != 2){
		/* 如果没有参数，给出使用方法 */
		fprintf (stderr “usage: getip address\n”);
		/* 然后退出 */
		exit(1);
	}
	/* 取得主机信息 */
	if（(h=gethostbyname(argv[1])) == NULL）
	{
		/* 如果 gethostbyname 失败，则给出错误信息 */
		herror(“gethostbyname”);
		/* 然后退出 */
		exit(1);
	}
	/* 列印程序取得的信息 */
	printf(“Host name : %s\n”, h->h_name);
	printf(“IP Address : %s\n”, inet_ntoa (*((struct in_addr *)h->h_addr))）;
		/* 返回 */
	return 0;
}
```

# 五、套接字Client/Server实现的例子
现在是一个服务器／客户端的世界．几乎网络上的所有工作都是由客户端向服务器端
发送请求来实现的．比如 Telnet ，当你向一个远程主机的 23 端口发出连接请求的时候，
远程主机上的服务程序（Telnetd）就会接受这个远程连接请求。允许你进行 login 操作。
等等。
## 5.1 简单的流服务器
这个服务器所有的工作就是给远程的终端发送一个字符串：“Hello,World!”你所需要
做的就是在命令行上启动这个服务器，然后在另外一台机器上使用 telnet 连接到这台我们
自己写的服务器上：
```
telnet 127.0.0.1 4000
```
 就是你运行我们自己写的服务器的那台机器名。
```
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <sys/wait.h>

/* 服务器要监听的本地端口 */
#define MYPORT 4000

/* 能够同时接受多少没有 accept 的连接 */
#define BACKLOG 10

int main()
{
	/* 在 sock_fd 上进行监听，new_fd 接受新的连接 */
	int sock_fd, new_fd ;
	/* 自己的地址信息 */
	struct sockaddr_in my_addr;
	/* 连接者的地址信息*/
	struct sockaddr_in their_addr;
	int sin_size;
	/* 这里就是我们一直强调的错误检查．如果调用 socket() 出错，则返回 */ 
	
	if(socket(AF_INET, SOCK_STREAM, 0)) == -1)
	{
		/* 输出错误提示并退出 */
		perror(“socket”);
		exit(1);
	}
	/* 主机字节顺序 */
	my_addr.sin_family = AF_INET;
	/* 网络字节顺序，短整型 */
	my_addr.sin_port = htons(MYPORT);
	/* 将运行程序机器的 IP 填充入 s_addr */
	my_addr.sin_addr.s_addr = INADDR_ANY;
	/* 将此结构的其余空间清零 */
	bzero(&(my_addr.sin_zero), 8);
	/* 这里是我们一直强调的错误检查！！ */ 
	if (bind(sockfd, (struct sockaddr *)&my_addr,sizeof(struct sockaddr)) == -1)
	{
		/* 如果调用 bind()失败，则给出错误提示，退出 */
		perror(“bind”);
		exit(1);
	}
	/* 这里是我们一直强调的错误检查！！ */
	if (listen(sockfd, BACKLOG) == -1)
	{
		/* 如果调用 listen 失败，则给出错误提示，退出 */
		perror(“listen”);
		exit(1);
	}
	
	while(1)
	{
		/* 这里是主 accept()循环 */
		sin_size = sizeof(struct sockaddr_in);
		/* 这里是我们一直强调的错误检查！！ */
		
		if ((new_fd = accept(sockfd, (struct sockaddr *)&their_addr, &sin_size)) == -1)
		{
		/* 如果调用 accept()出现错误，则给出错误提示，进入下一个循环 */
			perror(“accept”);
			continue;
		}
	}
	/* 服务器给出出现连接的信息 */
	printf（“server: got connection from s\n”,inet_ntoa(their_addr.sin_addr)）;
	/* 这里将建立一个子进程来和刚刚建立的套接字进行通讯 */
	if (!fork()){
		/* 这里是子进程 */
		/* 这里就是我们说的错误检查！ */
		if (send(new_fd, “Hello, world!\n”, 14, 0) == -1)
		{
			/* 如果错误，则给出错误提示，然后关闭这个新连接，退出 */
			perror(“send”);
			close(new_fd);
			exit(0);
		}
		/* 关闭 new_fd 代表的这个套接字连接 */
		close(new_fd);
	}
	/* 等待所有的子进程都退出 */

	while(waitpid(-1,NULL,WNOHANG) > 0);
}
```
## 5.2 简单的流式套接字客户端程序
这个程序比起服务器端程序要简单一些。它所做的工作就是 connect()到服务器的 4000
端口，然后把服务器发送的字符串给显示出来

```
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <sys/types.h>
#include <netinet/in.h>
#include <sys/socket.h>

/* 服务器程序监听的端口号 */

#define PORT 4000

/* 我们一次所能够接收的最大字节数 */

#define MAXDATASIZE 100

int main(int argc, char *argv[])
{
	
	/* 套接字描述符 */
	
	int sockfd, numbytes;
	
	char buf[MAXDATASIZE];
	
	struct hostent *he;
	
	/* 连接者的主机信息 */
	
	struct sockaddr_in their_addr;
	
	/* 检查参数信息 */
	
	if (argc != 2)
	
	{
	
		/* 如果没有参数，则给出使用方法后退出 */
		
		fprintf(stderr,“usage: client hostname\n”);
		
		exit(1);
	
	}
	
	/* 取得主机信息 */
	
	if ((he=gethostbyname(argv[1])) == NULL)
	
		/* 如果 gethostbyname()发生错误，则显示错误信息并退出 */
		
		herror(“gethostbyname”);
		
		exit(1);
	
	}
	
	if ((sockfd = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
	
		/* 如果 socket()调用出现错误则显示错误信息并退出 */
		
		perror(“socket”);
		
		exit(1);
	
	}
	/* 主机字节顺序 */
	
	their_addr.sin_family = AF_INET;
	
	/* 网络字节顺序，短整型 */
	
	their_addr.sin_port = htons(PORT);
	
	their_addr.sin_addr = *((struct in_addr *)he->h_addr);
	
	/* 将结构剩下的部分清零*/
	
	bzero(&(their_addr.sin_zero), 8);
	
	if（connect(sockfd, (struct sockaddr *)&their_addr, sizeof(struct sockaddr)) == -1）
	
	{
	
		/* 如果 connect()建立连接错误，则显示出错误信息，退出 */
		
		perror(“connect”);
		
		exit(1);
	
	}
	
	if((numbytes=recv(sockfd, buf, MAXDATASIZE, 0)) == -1）
	{
	
		/* 如果接收数据错误，则显示错误信息并退出 */
		
		perror(“recv”);
		
		exit(1);
	
	}
	
	buf[numbytes] = ‘\0’;
	
	printf(“Received: %s”,buf);
	
	close(sockfd);
	
	return 0;

}
```
先启动server 再client。
使用以下命令，查看建立连接的端口号：
```
netstat -anp| more
```


