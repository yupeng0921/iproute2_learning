# ss option explains
ss output may have different format for difference protocol type, this
document explains the format base on the TCP protocol
## -m option
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
