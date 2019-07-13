# 说明

比 netstat 好用的socket统计信息，iproute2 包附带的另一个工具，允许你查询 socket 的有关统计信息

**ss命令** 用来显示处于活动状态的套接字信息。ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。

当服务器的socket连接数量变得非常大时，无论是使用netstat命令还是直接`cat /proc/net/tcp`，执行速度都会很慢。可能你不会有切身的感受，但请相信我，当服务器维持的连接达到上万个的时候，使用netstat等于浪费 生命，而用ss才是节省时间。

天下武功唯快不破。ss快的秘诀在于，它利用到了TCP协议栈中tcp_diag。tcp_diag是一个用于分析统计的模块，可以获得Linux 内核中第一手的信息，这就确保了ss的快捷高效。当然，如果你的系统中没有tcp_diag，ss也可以正常运行，只是效率会变得稍慢。

# 命令格式

ss [参数]

ss [参数] [过滤]

# 输入参数：

h, --help	帮助信息

-V, --version	程序版本信息

-n, --numeric	不解析服务名称

-r, --resolve        解析主机名

-a, --all	显示所有套接字（sockets）

-l, --listening	显示监听状态的套接字（sockets）

-o, --options        显示计时器信息

-e, --extended       显示详细的套接字（sockets）信息

-m, --memory         显示套接字（socket）的内存使用情况

-p, --processes	显示使用套接字（socket）的进程

-i, --info	显示 TCP内部信息

-s, --summary	显示套接字（socket）使用概况

-4, --ipv4           仅显示IPv4的套接字（sockets）

-6, --ipv6           仅显示IPv6的套接字（sockets）

-0, --packet	        显示 PACKET 套接字（socket）

-t, --tcp	仅显示 TCP套接字（sockets）

-u, --udp	仅显示 UCP套接字（sockets）

-d, --dccp	仅显示 DCCP套接字（sockets）

-w, --raw	仅显示 RAW套接字（sockets）

-x, --unix	仅显示 Unix套接字（sockets）

-f, --family=FAMILY  显示 FAMILY类型的套接字（sockets），FAMILY可选，支持  unix, inet, inet6, link, netlink

-A, --query=QUERY, --socket=QUERY

​      QUERY := {all|inet|tcp|udp|raw|unix|packet|netlink}[,QUERY]

-D, --diag=FILE     将原始TCP套接字（sockets）信息转储到文件

 -F, --filter=FILE   从文件中都去过滤器信息

​       FILTER := [ state TCP-STATE ] [ EXPRESSION ]

# 输出参数：

# 实例

ss -lp | grep 3306

找出打开套接字/端口应用程序