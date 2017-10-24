# ss option explains
ss output may have different format for difference protocol type, this
document explains the format base on the TCP protocol
## -m option
Show memory information. Format:

    skmem:(r<rmem_alloc>,rb<rcv_buf>,t<wmem_alloc>,tb<snd_buf>,f<fwd_alloc>,w<wmem_queued>,o<opt_mem>,bl<back_log>)

* <rmem_alloc>  
  the memory allocated for receiving package
* <rcv_buf>  
  the total memory can be allocated for receiving package
* <wmem_alloc>  
  the memory used for sending package (which has been sent to layer 3).
* <snd_buf>  
  the total memory can be allocated for sending package
* <fwd_alloc>  
  the memory allocated by the socket as cache, but not used for
  receiving/sending pacakge yet. If need memory to send/receive package,
  the memory in this cache will be used before allocate additional
  memory.
* <wmem_queued>  
  The memory allocated for sending package (which has not been sent to
  layer 3).
* <opt_mem>  
  The memory used for storing socket option, e.g., the key for TCP MD5
  signature.
* <back_log>  
  The memory used for the sk backlog queue. On a process context, if
  the process is receving package, and a new package is received, it will be
  put into the sk backlog queue, so it can be received by the process
  immediately.

## -o option
Show timer information. Format:

    timer:(<timer_name>,<expire_time>,<retrans>)

* <timer_name>  
  the name of the timer, there are five kinds of timer names:
  * on: means one of these timers: tcp retrans timer, tcp early retrans timer and tail loss probe timer
  * keepalive: tcp keep alive timer
  * timewait: timewait stage timer
  * persist: zero window probe timer
  * unknown: none of the above timers
* <expire_time>  
  how long time the timer will expire
* retrans  
  how many times the retran occurs

## -e option
Show additional socket information. Format:

    uid:<uid_number> ino:<inode_number> sk:<cookie>

* <uid_number>  
  the user id the socket belongs to
* <inode_number>  
  the socket's inode number in VFS
* <cookie>  
  an uuid of the socket

## -i option
show tcp internal information

* ts  
  show string "ts" if the timestamp option is set
* sack  
  show string "sack" if the sack option is set
* ecn  
  show string "ecn" if the explicit congestion notification option is
  set
* ecnseen  
  show string "ecnseen" if the saw ecn flag is found in received packages
* fastopen  
  show string "fastopen" if the fastopen option is set
* cong_alg  
  the congestion algorithm name, the default congestion algorithm is
  "cubic"
* wscale:<snd_wscale>:<rcv_wscale>  
  if window scale option is used, this field shows the send scale factory and
  receive scale factory
* rto:<icsk_rto>  
  tcp retransmission timeout value, the unit is millisecond
* backoff:<icsk_backoff>  
  used for exponential backoff retransmission, the actual retransmission
  timeout vaule is icsk_rto << icsk_backoff
* rtt:<rtt>/<rttvar>  
  rtt is the average round trip time, rttvar is the mean deviation of
  rtt, their units are millisecond
* ato:<ato>  
  ack timeout, unit is millisecond, used for delay ack mode
* mss:<mss>  
  max segment size
* cwnd:<cwnd>  
  congestion window size
* ssthresh:<ssthresh>  
  tcp congestion window slow start threshold
* bytes_acked:<bytes_acked>  
  bytes acked
* bytes_received:<bytes_received>  
  bytes received
* segs_out:<segs_out>  
  segments sent out
* segs_in:<segs_in>  
  segments received
* send <send_bps>bps  
  egress bps
* lastsnd:<lastsnd>  
  how long time since the last package sent, the unit is millisecond
* lastrcv  
  how long time since the last package received, the unit is millisecond
* lastack:<lastack>  
  how long time since the last ack received, the unit is millisecond
* pacing_rate <pacing_rate>bps/<max_pacing_rate>bps  
  the pacing rate and max pacing rate
* rcv_space:<rcv_space>  
  a helper variable for TCP internal auto tunning socket receive buffer
