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

Some of them are defined by rfc1213:

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

