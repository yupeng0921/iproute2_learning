# nstat usage

## data source
nstat get network statistics from several places:
/proc/net/netstat
/proc/net/snmp
/proc/net/snmp6
/proc/net/sctp/snmp
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
