# nstat usage

## data source
nstat get network statistics from several places:
* /proc/net/netstat
* /proc/net/snmp
* /proc/net/snmp6
* /proc/net/sctp/snmp

## history file location
by default, the history file is:

/tmp/.nstat.u<your_uid>

e.g., if the uid of current user is 1000, the history file would be:

/tmp/.nstat.u1000

You could overwrite it by setting the NSTAT_HISTORY environment
variable.

## daemon mode

The '-d <interval>' option could let nstat run in daemon mode. In daemon mode,
nstat doesn't generate any output, nor update history file. It just
get the statistics for every <interval> seconds.
Every time you run nstat command without '-d <interval>' option, nstat
will try to find whether there is a nstat daemon, if there is a nstat
daemon, instead of get statistics from /proc, the nstat will get
statistics from the nstat daemon. Actually, below is the nstat search
path:
1. if current user has a nstat daemon, try to get statistics from it
2. if the root use has a nstat daemon, try to get statistics from it
3. get statistics from /proc
When you use the  '-d <interval>' option, you could combine it with a
'-t <time_constant>' option. It will keep the average value of the
statistics for the last <time_constant> seconds. For example, if you
run:

nstat -d 10 -t 60

The nstat daemon will get statistics for every 10 seconds, and keep a
60 seconds average value. Then, every time you run 'nstat', the nstat
command will get the current value and the average value from the
nstat daemon. e.g.:

    $ nstat -d 10 -t 60
    $ sleep 60
    $ nstat
    nstat: history is stale, ignoring it.
    #2842.1804289383 sampling_interval=10 time_const=60
    IpInReceives                    2726               0.1
    IpInDelivers                    2703               0.1
    IpOutRequests                   2370               0.1
    IcmpInMsgs                      2                  0.0
    IcmpInErrors                    1                  0.0
    IcmpInDestUnreachs              2                  0.0
    ...

The last column is the average value of the last 60 seconds. You
should consider the average value as a weigithed average, the recent
value has a higher weighted, the algorithm is similar as linux
kernel's load average.

# nstat statistics meaning
nstat get lots of statistics from linku kernel, most of them are
defined by several RFCs, others are linux kernel extentions without
well documentation. Here I will performance several experiments, and
explain the meanging of the statistics depend on the experiments. Most
of the experiments run on one or two virtual machines in my local
computer. The os is ubuntu 18.04. Before every experiment, I run
'nstat -n' to update the nstat history, so the nstat outputs after the
experiments won't be impacted by previous network traffic.

## a simple ping
Run the ping command against the public dns server 8.8.8.8

    ubuntu@nstat-a:~$ ping 8.8.8.8 -c 1
    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
    64 bytes from 8.8.8.8: icmp_seq=1 ttl=119 time=17.8 ms
    
    --- 8.8.8.8 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 17.875/17.875/17.875/0.000 ms

Below is the nstat result:

    ubuntu@nstat-a:~$ nstat
    #kernel
    IpInReceives                    1                  0.0
    IpInDelivers                    1                  0.0
    IpOutRequests                   1                  0.0
    IcmpInMsgs                      1                  0.0
    IcmpInEchoReps                  1                  0.0
    IcmpOutMsgs                     1                  0.0
    IcmpOutEchos                    1                  0.0
    IcmpMsgInType0                  1                  0.0
    IcmpMsgOutType8                 1                  0.0
    IpExtInOctets                   84                 0.0
    IpExtOutOctets                  84                 0.0
    IpExtInNoECTPkts                1                  0.0

### defined by rfc1213:

* IpInReceives

  [rfc1213 page 26](https://tools.ietf.org/html/rfc1213#page-26)

  The total number of input datagrams received from
  interfaces, including those received in error.

* IpInDelivers

  [rfc1213 page 28](https://tools.ietf.org/html/rfc1213#page-28)

  The total number of input datagrams successfully
  delivered to IP user-protocols (including ICMP).
  
* IpOutRequests

  [rfc1213 page 28](https://tools.ietf.org/html/rfc1213#page-28)

  The total number of IP datagrams which local IP
  user-protocols (including ICMP) supplied to IP in
  requests for transmission.  Note that this counter
  does not include any datagrams counted in
  ipForwDatagrams.

* IcmpInMsgs

  [rfc1213 page 41](https://tools.ietf.org/html/rfc1213#page-41)

  The total number of ICMP messages which the
  entity received.  Note that this counter includes
  all those counted by icmpInErrors.

* IcmpInEchoReps

  [rfc1213 page 42](https://tools.ietf.org/html/rfc1213#page-42)

  The number of ICMP Echo Reply messages received.

* IcmpOutMsgs
  [rfc1213 page 43](https://tools.ietf.org/html/rfc1213#page-43)

  The total number of ICMP messages which this
  entity attempted to send.  Note that this counter
  includes all those counted by icmpOutErrors.

* IcmpOutEchos

  [rfc1213 page 45](https://tools.ietf.org/html/rfc1213#page-45)

  The number of ICMP Echo (request) messages sent.

IcmpMsgInType0 and IcmpMsgOutType8 are not defined by any snmp related
RFCs, but theyir meaning are quite straightforward, they count the
packet number of specific icmp packet types. We could find the icmp
types here:

<https://www.iana.org/assignments/icmp-parameters/icmp-parameters.xhtml>

Type 8 is echo, type 0 is echo reply.

Until now, we can easily explain these items of the nstat: We sent a
icmp echo request, so IpOutRequests, IcmpOutMsgs, IcmpOutEchos and
IcmpMsgOutType8 were increased 1. We got icmp echo reply from 8.8.8.8,
so IpInReceives, IcmpInMsgs, IcmpInEchoReps, IcmpMsgInType0 were
increased 1. The icmp echo reply was passed to icmp layer via ip
layer, so IpInDelivers was increased 1.

Please note, these metrics don't aware LRO/GRO, e.g., IpOutRequests
might count 1 packet, but hardware splites it to 2, and sends them
separately.

### IpExtInOctets and IpExtOutOctets

They are linux kernel extentions, no rfc definiations. Please note,
rfc1213 indeed defines
[ifInOctets](https://tools.ietf.org/html/rfc1213#page-20) and
[ifOutOctets](https://tools.ietf.org/html/rfc1213#page-22), but they
are not the same things. The ifInOctets and ifOutOctets are packets
size which include the mac layer. But IpExtInOctets and IpExtOutOctets
don't inlucde mac layer.

In our example, an ICMP echo request has four parts:
* 14 bytes mac header
* 20 bytes ip header
* 16 bytes icmp header
* 48 bytes data (default value of the ping command)

So IpExtInOctets value is 20+16+48=84. The IpExtOutOctets is similar.

### IpExtInNoECTPkts
We could find IpExtInNoECTPkts in the nstat output, but kernel provide
four similar statistics, we explain them together, they are:
* IpExtInNoECTPkts
* IpExtInECT1Pkts
* IpExtInECT0Pkts
* IpExtInCEPkts

They indicate four kind of ECN IP packets, they are defined here:

<https://tools.ietf.org/html/rfc3168#page-6>

These 4 statistics count how many packets received per ECN
status. They count the real frame number regardless the LRO/GRO. So
for a same packet, you might find that IpInReceives count 1, but
IpExtInNoECTPkts counts 2 or more.

### additional explain
The ip layer stastics are recorded by the ip layer code in
kernel. I mean, if you send packet to a lower layer directly, Linux
kernel won't recorded it. For exmaple, tcpreplay will open an
AF_PACKET socket, and send packet to layer 2, although it could send
an IP packet, you can't find it from the nstat output. Here is an
exmaple:

We capture the ping packet by tcpdump

    ubuntu@nstat-a:~$ sudo tcpdump -w /tmp/ping.pcap dst 8.8.8.8

Then run ping command:

    ubuntu@nstat-a:~$ ping 8.8.8.8 -c 1

Terminate tcpdump by Ctrl-C, and run 'nstat -n' to update the nstat
history. Then run tcpreplay:

    ubuntu@nstat-a:~$ sudo tcpreplay --intf1=ens3 /tmp/ping.pcap
    Actual: 1 packets (98 bytes) sent in 0.000278 seconds
    Rated: 352517.9 Bps, 2.82 Mbps, 3597.12 pps
    Flows: 1 flows, 3597.12 fps, 1 flow packets, 0 non-flow
    Statistics for network device: ens3
            Successful packets:        1
            Failed packets:            0
            Truncated packets:         0
            Retried packets (ENOBUFS): 0
            Retried packets (EAGAIN):  0

Check the nstat output:

    ubuntu@nstat-a:~$ nstat
    #kernel
    IpInReceives                    1                  0.0
    IpInDelivers                    1                  0.0
    IcmpInMsgs                      1                  0.0
    IcmpInEchoReps                  1                  0.0
    IcmpMsgInType0                  1                  0.0
    IpExtInOctets                   84                 0.0
    IpExtInNoECTPkts                1                  0.0

We can see, nstat only show the received packet, because the IP layer
of kernel only know the reply of 8.8.8.8, it doesn't know what
tcpreplay sent.

At the same time, when you use AF_INET socket, even you use the
SOCK_RAW option, the IP layer will still try to verify wether the
packet is an ICMP packet, if it is, kernel will still count it to its
stasticis and you can find it in the output of nstat.

## simple tcp examples

### tcp 3 way handshake
On server side, we run:

    ubuntu@nstat-b:~$ nc -lknv 0.0.0.0 9000
    Listening on [0.0.0.0] (family 0, port 9000)

On client side, we run:

    ubuntu@nstat-a:~$ nc -nv 192.168.122.251 9000
    Connection to 192.168.122.251 9000 port [tcp/*] succeeded!

On server side, we can find below nstat output:

    ubuntu@nstat-b:~$ nstat | grep -i tcp
    TcpPassiveOpens                 1                  0.0
    TcpInSegs                       2                  0.0
    TcpOutSegs                      1                  0.0
    TcpExtTCPPureAcks               1                  0.0

On client side, we can find below nstat output:

    ubuntu@nstat-a:~$ nstat | grep -i tcp
    TcpActiveOpens                  1                  0.0
    TcpInSegs                       1                  0.0
    TcpOutSegs                      2                  0.0
