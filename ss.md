# ss option explains
ss output may have different format for difference protocol type, this
document explains the format base on the TCP protocol
## -m option
show memory information
format:

    skmem:(r<rmem_alloc>,rb<rcv_buf>,t<wmem_alloc>,tb<snd_buf>,f<fwd_alloc>,w<wmem_queued>,o<opt_mem>,bl<backlog>)

* <rmem_alloc>:
  the memory allocated for receiving package
* <rcv_buf>
  the total memory can be allocated for receive package
* <wmem_alloc>
  the memory allocated for sending package, the package has
  been sent to IP layer, but not been acked.
* <snd_buf>
  the total memory can be allocated for send package
* <fwd_alloc>
  the memory allocated by the socket as cache, but not used for
  receive/send pacakge yet. If need memory to send/receive package,
  the memory in this cache will be used before allocate additional
  memory.
* <wmem_queued>
  The memory allocated for sending package, the package has not yet
  been sent to IP layer, which means the package is queued in the
  socket.
* <opt_mem>
  The memory used for store socket option, e.g., the key for TCP MD5
  signature.
* <backlog>
  The memory used for the sk backlog queue. On a process context, if
  the process is receving package, and a new package comes, it will be
  put into the sk backlog queue, so it can be received by the process
  immediately.

## -o option
show timer information
format:

    timer:(<timer_name>,<expire_time>,<retrans>)

* <timer_name>
  the name of the timer, there are five kinds of timer names:
  on: means one of these timers: tcp retrans timer, tcp early retrans timer and tail loss probe timer
  keepalive: tcp keep alive timer
  timewait: timewait stage timer
  persist: zero window probe timer
  unknown: none of the above timers
* <expire_time>
  how long time the timer will expire
* retrans
  how may times the retran occurs

## -e option
show additional socket information
format:

    uid:<uid_number> ino:<inode_number> sk:<cookie>

* <uid_number>
  the user id if the socket belongs to a user
* <inode_number>
  the socket's inode number in VFS
* cookie
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
  show string "ecnseen" if the saw enc flag in received package
* fastopen
  show string "fastopen" if the fastopen option is set
* cong_alg
  the congestion algorithm name, the default congestion algorithm is
  "cubic"
* wscale:<snd_wscale>:<rcv_wscale>
  if window scale option is used, will show the send scale factory and
  receive scale factory
* rto:<icsk_rto>
  tcp retransmission timeout value, the unit is usecond
* backoff:<icsk_backoff>
  used for exponential backoff retranmission, the actual retranmission
  timeout vaule should be icsk_rto << icsk_backoff
