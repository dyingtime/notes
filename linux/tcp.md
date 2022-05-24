## Linux TCP Tunning

![tcp status](../assets/927655-20161215133843776-605308204.png)

- `net.ipv4.ip_local_port_range`
  - 临时端口（ephemeral ports）的范围
  - 默认值为 `net.ipv4.ip_local_port_range = 32768 60999`
  - 调大此参数可以一定程度上缓解 `TIME_WAIT`，可以修改为 `net.ipv4.ip_local_port_range = "4096 60999"`
  - `TIME_WAIT` 的时间是 `2MSL`，无法修改
- `net.ipv4.tcp_rmem`
  - 值是3维数组 `[min, default, max]`，收包缓冲（`receive buffer`）的大小，`TCP`会根据此值动态调整收包缓冲的大小
  - 默认值是 `net.ipv4.tcp_rmem = 4096 87380 6291456`
  - 第一列表示每个 `TCP` socket 的最小收包缓冲，这个值用于系统内存紧张时保证最低限度的连接建立，注意指定过 `SO_RCVBUF` 的 socket 不受此参数限制。
  - 第二列表示每个 `TCP` socket 的默认收包缓冲，此数值将会覆盖全局参数 `net.core.rmem_default`
  - 三列表示每个 TCP socket 最大收包缓冲，注意指定过 `SO_RCVBUF` 的 socket 不受此参数限制。此数值 **不覆盖** 全局参数 `net.core.rmem_max`
  - 可根据实际情况调整，不是越大越好
- `net.ipv4.tcp_wmem`
  - 值是3维数组 `[min, default, max]`，发包缓冲（`send buffer`）的大小，`TCP`会根据此值动态调整发包缓冲的大小
  - 默认值是 `net.ipv4.tcp_wmem = 4096 16384 4194304`
  - 第一列表示每个 `TCP` socket 的最小发包缓冲，这个值用于系统内存紧张时保证最低限度的连接建立，注意指定过 `SO_SNDBUF` 的 socket 不受此参数限制。
  - 第二列表示每个 `TCP` socket 的默认收包缓冲，此数值将会覆盖全局参数 `net.core.wmem_default`
  - 三列表示每个 TCP socket 最大收包缓冲，注意指定过 `SO_SNDBUF` 的 socket 不受此参数限制。此数值 **不覆盖** 全局参数 `net.core.wmem_max`
  - 可根据实际情况调整，不是越大越好
- `net.core.rmem` & `net.core.wmem`
  - 和 `net.ipv4.tcp_rmem` `net.ipv4.tcp_wmem` 基本相同，但作用于所有协议
  - `tcp_wmem` 和 `tcp_rmem` 的最大值不能超过该值的最大值
- `net.ipv4.tcp_tw_reuse`
  - 依赖 TCP 时间戳 `net.ipv4.tcp_timestamps = 1`，是否复用 `TIME_WAIT` 状态的 socket
  - 默认值 `net.ipv4.tcp_tw_reuse = 0`
  - 开启可以复用 `TIME_WAIT` 状态的连接，官方文档不建议修改
  - 官方文档说明：It should not be changed without advice/request of technical experts
- `net.ipv4.tcp_tw_recycle`
  - 依赖 TCP 时间戳 `net.ipv4.tcp_timestamps = 1`， 是否回收 `TIME_WAIT` 状态的 socket
  - 默认值是 `net.ipv4.tcp_tw_recycle = 0`
  - NAT环境下会有问题，一般不建议开
  - Enabling this option is not recommended as the remote IP may not use monotonically increasing timestamps (devices behind NAT, devices with per-connection timestamp offsets)
- `net.ipv4.fin_timeout`
  - `FIN_WAIT2` 状态的 socket 存活时间
  - 默认值是 `net.ipv4.fin_timeout = 60`
  - 可以修改为 `net.ipv4.fin_timeout = 30`
  - `FIN_WAIT2` 只存在于主动关闭端，`HTTP` 由服务端关闭 `TCP` 连接，所以客户端修改此值没用？
- `net.core.somaxconn`
  - `ESTABLISHED` 状态，等待 `accept` 的最大队列长度
  - 默认值是 `net.core.somaxconn = 128`
  - 影响服务端，客户端调整意义不大
  - 在一个 socket 进行 `listen(int sockfd, int backlog)` 时需要指定 `backlog` 值作为参数，如果这个 `backlog` 值大于 `somaxconn` 的值，最大队列长度将以 `somaxconn` 为准，多余的连接请求将被放弃，此时客户端可能收到一个 `ECONNREFUSED` 或忽略此连接并重传。此 `backlog` 指的是状态为 `ESTABLISHED` 但还未被应用程序 `accept()` 的连接数
- `net.ipv4.tcp_max_syn_backlog`
  - 未被 `ACK` 的 `SYN` 包的最大队列长度，超过这个数值后，多余的请求会被丢弃
  - 默认值是 `net.ipv4.tcp_max_syn_backlog = 128`
  - 影响服务端，客户端调整意义不大

## 参考文章

- [Linux 网络调优：内核网络栈参数篇](https://www.starduster.me/2020/03/02/linux-network-tuning-kernel-parameter/#netipv4tcp_max_syn_backlog_netipv4tcp_syncookies)
- [increasing the maximum number of tcp p connections in linux](https://stackoverflow.com/questions/410616/increasing-the-maximum-number-of-tcp-ip-connections-in-linux)
- [TCP - Linux manual page](https://man7.org/linux/man-pages/man7/tcp.7.html)
- [TCP SOCKET中backlog参数的用途是什么](http://www.cnxct.com/something-about-phpfpm-s-backlog/)
- [TCP/IP协议中backlog参数](https://www.cnblogs.com/Orgliny/p/5780796.html)](https://www.cnblogs.com/orgliny/p/5780796.html)