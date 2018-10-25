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

Except TcpExtTCPPureAcks, all other statistics are defined by rfc1213

* TcpActiveOpens

  [rfc1213 page 47](https://tools.ietf.org/html/rfc1213#page-47)

  The number of times TCP connections have made a
  direct transition to the SYN-SENT state from the
  CLOSED state.

* TcpPassiveOpens

  [rfc1213 page 47](https://tools.ietf.org/html/rfc1213#page-47)

  The number of times TCP connections have made a
  direct transition to the SYN-RCVD state from the
  LISTEN state.

* TcpInSegs

  [rfc1213 page 48](https://tools.ietf.org/html/rfc1213#page-48)

  The total number of segments received, including
  those received in error.  This count includes
  segments received on currently established
  connections.

* TcpOutSegs

  [rfc1213 page 48](https://tools.ietf.org/html/rfc1213#page-48)

  The total number of segments sent, including
  those on current connections but excluding those
  containing only retransmitted octets.

The TcpExtTCPPureAcks is an extention in linux kernel. When kernel
receives a TCP packet which set ACK flag and with no data,
TcpExtTCPPureAcks will increase 1.

Now we can easily explain the nstat outputs on server side and client
side.

When server received the first syn, it reply a syn+ack, and came into
SYN-RCVD state, so TcpPassiveOpens increased 1. The server received
syn, sent syn+ack, received ack, so server sent 1 packet, received 2
packets, TcpInSegs increased 2, TcpOutSegs increased 1. The last ack
of the 3 way handshake is a pure ack without data, so
TcpExtTCPPureAcks increased 1.

When client sent syn, the client came into SYN-SENT state, so
TcpActiveOpens increased 1, client sent syn, received syn+ack, sent
ack, so client sent 2 packets, received 1 packet, TcpInSegs increased
1, TcpOutSegs increased 2.

Note: about TcpInSegs and TcpOutSegs, rfc1213 doens't define the
behaviors when gso/gro/tso are enabled on the NIC (network interface
card). On current linux implementation, TcpOutSegs awares gso/tso, but
TcpInSegs doesn't aware gro. So TcpOutSegs will count the actual
packet number even only 1 packet is sent via tcp layer. If mutiple
packets arrived at a NIC, and they are merged to 1 packet, TcpInSegs
will only count 1.


### tcp disconnect

Continue our previous example, on the server side, we run

    ubuntu@nstat-b:~$ nc -lknv 0.0.0.0 9000
    Listening on [0.0.0.0] (family 0, port 9000)

On client side, we run

    ubuntu@nstat-a:~$ nc -nv 192.168.122.251 9000
    Connection to 192.168.122.251 9000 port [tcp/*] succeeded!

Now we type Ctrl-C on client side, stop the tcp connection between the
two nc command. Then we check the nstat output.

On server side:

    ubuntu@nstat-b:~$ nstat | grep -i tcp
    TcpInSegs                       2                  0.0
    TcpOutSegs                      1                  0.0
    TcpExtTCPPureAcks               1                  0.0
    TcpExtTCPOrigDataSent           1                  0.0

On client side:

    ubuntu@nstat-b:~$ nstat | grep -i tcp
    TcpInSegs                       2                  0.0
    TcpOutSegs                      1                  0.0
    TcpExtTCPPureAcks               1                  0.0
    TcpExtTCPOrigDataSent           1                  0.0

Wait for more than 1 minute, run nstat on client again:

    ubuntu@nstat-a:~$ nstat | grep -i tcp
    TcpExtTW                        1                  0.0

Most of the statistics are explained in the previous section except
two: TcpExtTCPOrigDataSent and TcpExtTW. Both of them are linux kernel
extentions.

TcpExtTW means a tcp connection is closed normally via
time wait stage, not via tcp reuse process.

About TcpExtTCPOrigDataSent, the [kernel patch
comment](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=f19c29e3e391a66a273e9afebaf01917245148cd)
has a good explain for it, I pasted it here:

TCPOrigDataSent: number of outgoing packets with original data (excluding retransmission but including data-in-SYN). This counter is different from TcpOutSegs because TcpOutSegs also tracks pure ACKs. TCPOrigDataSent is more useful to track the TCP retransmission rate.

## the effect of gso and gro

The Generic Segmentation Offload (GSO) and Generic Receive Offload
would affect the metrics of packet in/out on both ip and tcp
layer. Here is an iperf example. Before the test, run below command to
make sure both gso and gro are enbaled on the NIC:

    $ sudo ethtool -k ens3 | egrep '(generic-segmentation-offload|generic-receive-offload)'
    generic-segmentation-offload: on
    generic-receive-offload: on

On server side, run:

    iperf3 -s -p 9000

On client side, run:

    iperf3 -c 192.168.122.251 -p 9000 -t 5 -P 10

The server listened on tcp port 9000, client connected to the server,
created 10 threads parallel, run 5 seconds. After the pierf3 stopped, we
run nstat on both the server and cleint.

On server side:

    ubuntu@nstat-b:~$ nstat
    #kernel
    IpInReceives                    36346              0.0
    IpInDelivers                    36346              0.0
    IpOutRequests                   33836              0.0
    TcpPassiveOpens                 11                 0.0
    TcpEstabResets                  2                  0.0
    TcpInSegs                       36346              0.0
    TcpOutSegs                      33836              0.0
    TcpOutRsts                      20                 0.0
    TcpExtDelayedACKs               26                 0.0
    TcpExtTCPHPHits                 32120              0.0
    TcpExtTCPPureAcks               16                 0.0
    TcpExtTCPHPAcks                 5                  0.0
    TcpExtTCPAbortOnData            5                  0.0
    TcpExtTCPAbortOnClose           2                  0.0
    TcpExtTCPRcvCoalesce            7306               0.0
    TcpExtTCPOFOQueue               1354               0.0
    TcpExtTCPOrigDataSent           15                 0.0
    IpExtInOctets                   311732432          0.0
    IpExtOutOctets                  1785119            0.0
    IpExtInNoECTPkts                214032             0.0

Client side:

    ubuntu@nstat-a:~$ nstat
    #kernel
    IpInReceives                    33836              0.0
    IpInDelivers                    33836              0.0
    IpOutRequests                   43786              0.0
    TcpActiveOpens                  11                 0.0
    TcpEstabResets                  10                 0.0
    TcpInSegs                       33836              0.0
    TcpOutSegs                      214072             0.0
    TcpRetransSegs                  3876               0.0
    TcpExtDelayedACKs               7                  0.0
    TcpExtTCPHPHits                 5                  0.0
    TcpExtTCPPureAcks               2719               0.0
    TcpExtTCPHPAcks                 31071              0.0
    TcpExtTCPSackRecovery           607                0.0
    TcpExtTCPSACKReorder            61                 0.0
    TcpExtTCPLostRetransmit         90                 0.0
    TcpExtTCPFastRetrans            3806               0.0
    TcpExtTCPSlowStartRetrans       62                 0.0
    TcpExtTCPLossProbes             38                 0.0
    TcpExtTCPSackRecoveryFail       8                  0.0
    TcpExtTCPSackShifted            203                0.0
    TcpExtTCPSackMerged             778                0.0
    TcpExtTCPSackShiftFallback      700                0.0
    TcpExtTCPSpuriousRtxHostQueues  4                  0.0
    TcpExtTCPAutoCorking            14                 0.0
    TcpExtTCPOrigDataSent           214038             0.0
    TcpExtTCPHystartTrainDetect     8                  0.0
    TcpExtTCPHystartTrainCwnd       172                0.0
    IpExtInOctets                   1785227            0.0
    IpExtOutOctets                  317789680          0.0
    IpExtInNoECTPkts                33836              0.0

The TcpOutSegs and IpOutRequests on the server are 33836, exactly the
same as IpExtInNoECTPkts, IpInReceives, IpInDelivers and TcpInSegs on
the client side. Durng iperf3 test, server only reply very short
packets, so gso and gro has no effect on the server's reply.

On the client side, TcpOutSegs is 214072, IpOutRequests is 43786, the
tcp layer packet out is larger than ip layer packet out, because
TcpOutSegs count the packet number after gso, but IpOutRequests
doesn't. On the server side, IpExtInNoECTPkts is 214032, this number
is smaller a little than the TcpOutSegs on client side (214072), it
might cause by the packet loss. The IpInReceives, IpInDelivers and
TcpInSegs are obviously smaller than the TcpOutSegs on client side,
because these statistics count the packet after gro.
