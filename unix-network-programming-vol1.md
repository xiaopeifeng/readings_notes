section 2.7

page  44

​	TIME_WAIT State

​	the way in which a packet get "lost" in a network is usually the result of routing anomalies. A router crashes or a link between two routers goes down and it takes the routing protocols seconds or minutes to stablize and find an alternate path. During that time period, routing loops can occur (router A send packets to router B, and B sends them back to A) and packets can get caught in these loops. In the meantime, assuming the lost packet is a TCP segment, the sending TCP times out and retransmits the  packet, and the retransmitted packet gets to the final destination by some alternate path. But sometime later (up to MSL seconds after the lost packet started on its journey), the routing loop is corrected an the packet that was lost in the loop is sent to the final destination. This original packet is called a lost duplicate or a wandering duplicate. TCP must handle these duplicates.

​	There are two reasons for the TIME_WAIT state.

​	1 To implement TCP's full-duplex connection termination reliably

​	2 To allow old duplicate segments to expire in the network



section 2.11

page 57

​	The goal of the MSS is to tell the peer the actual value of the reassembly buffer size and to try to avoid fragmentation.  



page 58

​	Therefore, the successful return from write to a TCP socket only tells us that we can reuse our application buffer. It does not tell us that either the peer TCP has received the data or that the peer application has received the data.

​	TCP sends the data to IP in MSS-sized or smaller chunks, prepending its TCP header to each segment, where the MSS is the value announced by the peer, or 536 if the peer did not send an MSS option. IP prepends its header, searches the routing table for destination IP address (the matching routing entry specifies the outgoing interface), and passes the datagram to the appropriate datalink. IP might perform framentation before passing the datagram to the datalink, but as we said earlier, one goal of the MSS option is to try to avoid fragmentation and newer implementations alse use path MTU discovery. Each datalink has an output queue, and if this queue is full, the packet is discarded and an error is returned up the protocol stack.

page 59

we show the socket send buffer as a dashed box because it doesn't really exist. A UDP socket has a send buffer size (which we can change with the SO_SNDBUF socket option), but this is simply an upper limit on the maximum-sized datagram that can be written to the socket. If an application writes a datagram larger than the socket send buffer size, EMSGSIZE is returned. Since UDP is unreliable, it does not need to keep a copy of the application's data and does not need an actual send buffer. (the application data is normally copied into a kernel buffer of some form as it passes down the protocol stack, but this copy is discarded by the datalink layere after the data is transmitted.)



section 3.5

page 80

Byte Manipulation Functions

 #include <strings.h>

void bzero(void *dest, size_t nbytes);

void bcopy(const void *src, void *dest, size_t nbytes);



 #include <string.h>

  void *memset()



the first group of functions, whose names begin with b(for byte), are from 4.2BSD and are still provided by almost any system that supports the socket functions. The second group of functions, whose names begin with mem(for memory), are from the ANSIC standard and are provided with any system that supports an ANSI C library.



section 3.7 page 83

inet_pton and inet_ntop. the letters "p" and "n" stand for *presentation* and *numeric*



section 4.2 page 98

The "AF_" prefix stands for "address family" and the "PF_" prefix stands for "protocol family". Historically, the intent was that a single protocol family might support multiple address families and that the PF_ value was used to create the socket and the AF_ value was used in socket address structures. But in actuality, a protocol family support-header defines the PF_ value for a given protocol to be equal to the AF_ value for that protocol. WHile there's no guarantee that this equality between the two will always be true, should anyone change this for existing protocols, lots of existing code would break. To conform to existing coding practice, we use only the AF_ constants.



section 4.3 page 99

In the case of a TCP socket, the connect function initiates TCP's three-way handshake. The function returns only when the connection is established or an error occurs. There are several different error returns possible.

1 syn timeout, ETIMEDOUT

2 RST, ECONNREFUSED

3 ICMP "destination unreachable", EHOSTUNREACH



section 4.5 page 104

listen(int fd, int backlog)

To understand the backlog argument, we must realize that for a given listening socket, the kernel maintains two queues:

1 an incomplete connection queue, which contains an entry for each SYN that has arrived from a client for which the server is awaiting completion of the TCP three-way handshake

2 a completed connection queue, which contains an entry for each client with whom the TCP three-way handshake has completed.

sum of both queues cannot exceed backlog



section 4.7 page 112

there are two typical uses of fork:

1 a process makes a copy of itself so that one copy can handle one operation while the other copy does another task. This is typical for network servers.

2 a process wants to execute another program. Since the only way to create a new process is by calling fork, the process first calls fork to make a copy of itself, and then one of the copies calls exec to replace itself with new program. This is typical for programs such as shells.

six exec functions:

​	the differences in the six exec functions are: a) whether the program file to execute is specified by a filename or pathname (b) whether the arguments to the new program are listed one by one or referenced through an array of pointers; and (c) whether the environment of the calling process is passed to the new program or whether a new environment is specified



section 4.9 page 117

the default action of close with a TCP socket is to mark the socket as closed and return to the process immediately. The socket descriptor is no longer usable by the process. But, TCP will try to send any data that is already queued to be sent to the other end, and after this occurs, the normal TCP connection termination sequence takes place.



section 5.8 page 132

the purpose of the zombie state is to maintain information about the child for the parent to fetch at some later time. This information includes the process ID of the child, its termination status, and information on the resource utilization of the child (CPU time, memory, etc). 



section 5.9 page 134

when writing network programs that catch signals, we must be cognizant of interrupted system calls, and we must handle them.  cause: SA_RESTART flag



**Handling Interrupted System Calls**

 We used the term "slow system call" to describe accept, and we use this term for any system call that can block forever. That is, the system call need never return. Most networking functions fall into this category. For example, there is no guarantee that a server's call to accept will ever return, if there are no clients that will connect to the server. Similary, our server's call to read will never return if the client never sends a line for the server to echo. Other examples of slow system calls are reads and writes of pipes and terminal devices.  A notable exception is disk I/O, which usually returns to the caller.

​	the basic rule that applies here is that when a process is blocked in a slow system call and the process catches a signal and the signal handler returns, the system call can return an error of EINTR. some kernels automatically restart some interrupted system calls. For portablity, when we write a program that catches signals (most concurrent servers catch SIGCHLD), we must be prepared for slow system calls to return EINTR. Protablity problems are caused by the qualifiers "can" and "some", which were used earlier, and the fact that support the POSIX SA_RESTART flag is optional. Even if an implementation supports the SA_RESTART flag, not all interrupted system calls may automaically be restarted.



section 5.13 page 142

The rule that applies is:  When a process writes to a socket that has received an RST, the SIGPIPE signal is sent to the process.



**RST**

When un expectd TCP packet arrives at a host, that host usually responds by sending a reset packet back on the same connection. there are a few circumstances in which a tcp packet might not be expected: the two most common are:

​	1 the packet is an initial SYN packet trying to establish a connection to a server port on which no process is listening.

​	2 the packet arrives on a tcp connection that was previously established, but the local application already closed its socket or exited and the OS closed the socket.



section 6.2 page 160

POSIX defines these two terms as follows:

- A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes.
- An asynchronous I/O operation does not cause the requesting process to be blocked.

using these definitions, the first four I/O modes - blocking, nonblocking, I/O multiplexing, and signal-driven I/O -are all synchronous because the actual I/O operation(recvfrom) blocks the process. Only the asynchronous I/O model matches the asynchronous I/O definition.



section 6.3 page 164

select arguments

  these three arguments are value-result arguments. any descriptor that is not ready on return will have its corresponding bit cleared in the descriptor set. to handle this, we turn on all the bits in which we are interested in all the descriptor sets each time we call select.



section 6.6 page 172

the normal way to terminate a network connection is to call the close function. But, there are two limitations with close that can be avoided with shutdown:

- ***close*** decrements the descriptor's reference count and closes the socket only if the count reaches 0. With shutdown, we can initiate TCP's normal connection termination sequence( the four segments beginning with a FIN), regardless of the reference count.
- ***close*** terminates both directions of data transfer, reading and writing. Since a TCP connection is full-duplex, there are times when we want to tell the other end that we have finished sending, even though that end might have more data to send  us. 





section 6.8 page 180

the basic concept here is that when a server is handling multiple clients, the server can ***never*** block in a function call related to a single client. Doing so can hang the server and deny service to all other clients. This is called a denial-of-service attack. It does something to the server that prevents it from servicing other legitimate clients. Possible solutions are to: use nonblocking; thread per client; or timeout limit



section 7.5 page 198

Socket States

​	the following socket options are **inherited** by a connected TCP socket from the listening socket: SO_DEBUG, SO_DONTROUTE, SO_KEEPALIVE, SO_LINGER, SO_OOBINLINE, SO_RECVBUF, SO_RECVLOWAT, SO_SNDBUF, SO_SNDLOWAT, TCP_MAXSEG, TCP_NODELAY. this is import with TCP because the connected socket is not returned to a server by accept until the three-way handshake is completed by the TCP layer. To ensure that one of these socket options is set for the connected socket when the three-way handshake completes, we must set that option for the listening socket.



SO_KEEPALIVE option

This opeiton is normally used by servers, although clients can also use the option. Severs use the  option because they spend most of their time blocked waiting for input across the TCP connection, that is, waiting for a client request. But if the client host's connection drops, is powered off, or crashes, the server process will never know about it, and the server will continually wait for input that can never arrive. this is called a *half-open connection* . The keep-alive option will dectec these half-open connections and terminate them.



page 209

SO_SNDLOWAT

with udp, the low-water mark is used, as we described, but since the number of bytes of available space in the send buffer for a UDP socket never changes (since UDP does not keep a copy of the datagrams sent by the application), as long as the UDP socket send buffer size is greater than the socket's slow-water mark, the UDP socket is always writable.



TCP_NODELAY

​	TCP delayed acknowledgment

​         the additional wait time introduced by the delayed ACK can cause further delays when interacting with certain applications and configurations. If Nagle's algorithm is being used by the sending party, data will be queued by the sender until an ACK is received. If the sender does not send enough data to fill the maximum segment size (for example, if it performs two small writes followed by a blocking read) then the transfer will pause up to the ACK delay timeout.

​	the purpose of the Nagle algorithm is to reduce the number of small packets on a WAN. The algorithm states that if a given connection has outstanding data(i.e., data that our TCP has sent, and for which it is currently awaiting an acknowledgment), then no smalll packets will be sent on the connection in response to a user write operation until the existing data is acknowledged. The definition of a "small" packet is any packet smaller than the MSS. TCP will always send a full-sized packet if possible;



the *nagle* algorithm often interacts with the *delayed ACK* alrorithm.



SO_REUSEADDR

this flag indicates that the rules used in validating addresses supplied in a bind call should allow reuse of local address. For AF_INET sockets this means that a socket may bind, **except when there is an active listening socket bound to the address.**





section 8.2 page 241

Writing a datagram of **length 0** is **acceptable**. In the case of UDP, this results in an IP datagram containing an IP header (normally 20 bytes for IPV4), an 8-byte UDP header, and no data. This also means that a return value of 0 from recvfrom is acceptable for a datagram protocol: **It does not mean that the peer has closed the connection**, as does a return value of 0 from read on a TCP socket. Since UDP is connectionless, there is no such thing as closing a UDP connection.



page 243

Since UDP is a connectionless protocol, there is nothing like an EOF as we have with TCP.



we call this ICMP error an asynchronous error. The ICMP error is not returned until later which is why it is called asynchronous.

the basic rule is that an asynchronous error is not returned for a UDP socket unless the socket has been connected.



page 253

In summary, we can say that a UDP client or server can call connect only if that process uses the UDP socket to communicate with exactly one peer.



when an applicationi calls sendto on an unconnected UDP socket, Berkeley-derived kernels temporarily connect the socket, send the datagram, then unconnect the socket. Calling sendto for two datagrams on an unconnected UDP socket then involves the following six steps by the kernel:

- connect the socket
- output the first datagram
- unconnect the socket
- connect the socket
- output the second datagram
- unconnect the socket




secion 14.3 page 387

recv flags

**MSG_WAITALL** this flag was introduced with 4.3 BSD. it tells the kernel not to return from a read operation until the requested number of bytes have been read. 



section 15.7 page 420

**Passing Descriptors**

current unix systems provide a way to pass any open descriptor from one process to any other process. That is, there is no need for the processes to be releated, such as a parent and its child. the technique requires us to first establish a Unix domain socket between the two processes and then use sendmsg to send a special message across the Unix domain socket. This message is handled specially by the kernel, passing th eopen descriptor from the sender to the receiver.



section 16.2 page 436

th establishment of a TCP connection involves a three-way handshake and the connect function does not return until the client receives the ACK of its SYN. this means that a TCP connect always blocks th ecalling porocess for at least the RTT to the server.

traditionally, System V has returned the error EAGAIN for a nonblocking I/O operation that cannot be satisfied, while Berkeley-derived implementations have returned the error EWOULDBLOCK. Because of this history, the POSIX specification says either may be returned for this case. 



which use select, still uses blocking IO, if a line is available on standard input, we read it with read and then send it to the server with writen. But the call to writen can block if the socket buffer is full, while we are blocked in the call to writen, data could be avaiable for reading from the socket receive buffer.



section 16.4 page 448

when a tcp socket is set to nonblocking and then connect is called, connect returns immediately with an error of EINPROGRESS but the TCP three-way handshake continues. there are three uses for a nonblocking connect:

1. we can overlap over processing with the three way handshake. A connect takes one RTT to complete, and this can be anywhere from a few millseconds on a LAN to hundreds of milliseconds or a few seconds on a WAN. there might be other processing we wish to perform during this time
2. we can establish multiple connections at the same time using this tech. this has become popular with web browers.
3. since we wait for the connection to be established using select, we can specify a time limit for select.

page 451

nonblocking connects are one of the most nonportable areas of network programming.





section 22.4 page 595

When to use UDP instead of TCP

- UDP supports broadcasting and multicasting. indeed, UDP must be used if the application uses broadcasting or multicasting.
- UDP has no connection setup or teardown. UDP requires only two packets to exchange a request and a reply (assuming the size of each is less than the minimum MTU between the two end-systems). TCP requires about 10 packets, assuming that a new TCP connection is established for each request-reply exchage.

also important in this number-of-packet analysis is the number of packet round trips required to obtain the reply. the minimum tranasaction time for a UDP request-reply is RTT+server processing time (SPT). with TCP, however, if a new TCP connection is used for the request-reply, the minimum transaction time is 2*RTT+SPT, one RTT greater than the UDP time



page 596

in summary, we can state the following recommendations:

- UDP must be used for broadcast or multicast applications.
- UDP can be used for simple request-reply applications.
- UDP should not be used for bulk data transfer.



section 24.5

tcp does not have true out-of-band data. it provides an urgent pointer that is sent in the TCP header to the peer as soon as the sender goes into urgent mode. the receipt of this pointer by the other end of the connection tells that process that the sender has gone into urgent mode, and the pointer points to the final byte of urgent data. but all the data is still subject to tcp's normal flow control.