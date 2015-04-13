# OpenWrt NOTICES

## What's the "input device grab"

    By default, X input events are delivered to all clients that registered for a specific event on a window.
    The basic premise of a grab is that it affects event delivery to deliver events exclusively to one client only.
    There are three types of grabs, two classes of grabs, and two modes for a grab. The three types are:
    * active grabs
    * passive grabs
    * implicit passive grab
    The two classes are core grabs and device grabs and the two modes are synchronous and asynchronous.
    A grab comes in a combination of type + class + mode, so a grab may be an "active synchronous device grab".

## Something about the "Blcontrol problem"

    Since the input device (here is a touch screen) can not be grab by two clients with the block mode.
    And it is hard to catch the events by realtime in user space, so I processes the events in kernel space.
    Some ioctl are usefull.
    see -> [modou fb driver]

## How to make code simple

    When I feel that it is hard to manage a project, and the project may be not good in the beginning.
    Simple things are not necessarily the best, but the best one must be simple.
    Every function must be used by many others.
    Every file must be used by many others.
    If not, it is failure.
    See -> [app-ss-vpn](https://gitcafe.com/Modou/app-ss-vpn/blob/master/sbin/ss.sh)

## About Lua coroutine

    luci cgi run function:
``````````````````````````````lua
    function run()
      local r = luci.http.Request(
        luci.sys.getenv(),
        limitsource(io.stdin, tonumber(luci.sys.getenv("CONTENT_LENGTH"))),
        ltn12.sink.file(io.stderr)
      )
      local x = coroutine.create(luci.dispatcher.httpdispatch)
      local active = true
      -- as a sheduler
      while coroutine.status(x) ~= "dead" do
        local res, id, data1, data2 = coroutine.resume(x, r)
        if not res then
          -- return error
          break;
        end
        if active then
          if id == 1 then
            ...
          elseif id == 2 then
            active = false
          elseif id == 3 then
            data1:close()
          end
        end
      end
    end
``````````````````````````````

      See -> [Lua corotine demo] (http://blog.csdn.net/wzzfeitian/article/details/8832017)

## iptables modules

    If want to support some match/TARGET,  it must to install a user space module and its kernel module.

user space:

| module name | module describe |
| ----------- | --------------- |
| iptables-mod-hashlimit | iptables extensions for hashlimit matching Includes: - libipt_hashlimit |

kernel space:

| module name | module describe |
| ----------- | --------------- |
| kmod-ipt-hashlimit | Kernel modules support for the hashlimit bucket match module |

    see -> [netfilter modules](http://wiki.openwrt.org/doc/howto/netfilter)

## simple shell string

`````````````````````````````shell
DEBUGDIR="/data/modoudebug"
[ ! -d  "$DEBUGDIR" ] && exit 0
for path in `find $DEBUGDIR -type f`
do
        cp $path ${path#$DEBUGDIR} 2>/dev/null
done
`````````````````````````````

## shell awk cal

     total=`awk 'BEGIN{printf "%.2f", ($used+$free) '/' 1024 '/' 1024 }'`

## redirect traffic to ifb

     * redirect input traffic to ifb0
        tc qdisc  del dev $WAN_IFACE ingress 2>/dev/null
        tc qdisc  add dev $WAN_IFACE ingress
        tc filter add dev $WAN_IFACE parent ffff: protocol ip u32 match u32 0 0 action mirred egress redirect dev $DEVICE0

     * redirect output traffic to ifb1
        tc qdisc  del dev $WAN_IFACE root handle 1: htb default 2 2>/dev/null
        tc qdisc  add dev $WAN_IFACE root handle 1: htb default 2
        tc filter add dev $WAN_IFACE parent 1: protocol ip u32 match u32 0 0 action mirred egress redirect dev $DEVICE1
## crontabs

    0 * * * * /system/sbin/pingback.sh

## tc filter

     * filte OUTPUT icmp
        tc filter add dev eth0.2 parent 1: prio 1 protocol ip u32 match ip protocol 1 0xff flowid 1:1
     * filte OUTPUT src ip address from eth0.2(it's ip is 10.241.52.166)
        tc filter add dev eth0.2 parent 1: prio 1 protocol ip u32 match ip src 10.241.52.166/32 flowid 1:1
     * filte OUTPUT mark 6 package
        tc filter add dev eth0.2 parent 1: prio 1 handle 6 fw flowid 1:1

## iptables insert

     * head add
        iptables -I
     * tail add
     *  iptables -A

## ifb and iptables

     You'll need to filter aswell - on egress you can use iptables + marks (I don't think classify will work).
     But on ingress you can't use iptables because ifb is before netfilter.

## DNAT and SNAT

     Source NAT is when you alter the source address of the first packet: i.e. you are changing where the connection
     is coming from. Source NAT is always done post-routing, just before the packet goes out onto the wire.
     Masquerading is a specialized form of SNAT.
     Destination NAT is when you alter the destination address of the first packet: i.e. you are changing where the
     connection is going to. Destination NAT is always done before routing, when the packet first comes off the wire.
     Port forwarding, load sharing, and transparent proxying are all forms of DNAT.

     full NAT with SNAT and DNAT:
     iptables -t nat -A PREROUTING -d 205.254.211.17 -j DNAT --to-destination 192.168.100.17
     iptables -t nat -A POSTROUTING -s 192.168.100.17 -j SNAT --to-destination 205.254.211.17


## 关于dns劫持和访问速度

    [15/3/16 21:16:13] 军建: 我这看也没区别
    [15/3/16 21:16:44] 军建: 比你那的还快
    [15/3/16 21:16:45] 蚂蚁-刘维: 从理论上来说也应改没区别，就是几个udp包的交互
    [15/3/16 21:16:45] 军建: 。。
    [15/3/16 21:17:04] 蚂蚁-刘维: 关键的问题不在于解析
    [15/3/16 21:17:08] 军建: 那为啥在freewifi上就慢得这么明显呢
    [15/3/16 21:17:12] 蚂蚁-刘维: 在于解析出来的结果
    [15/3/16 21:17:49] 蚂蚁-刘维: 你是上海用户，用本地dns解析后，网站给你一个上海附近的服务器伺候你
    [15/3/16 21:18:17] 蚂蚁-刘维: 而你上海用户用贵州电信的dns解析，网站就给你一个贵州附近的服务器伺候你
    [15/3/16 21:18:29] 军建: 哦。。
    [15/3/16 21:18:35] 蚂蚁-刘维: 你开始进行大数据通讯的时候从贵州取就很悲剧了
    [15/3/16 21:19:00] 蚂蚁-刘维: 几乎国内所有的静态服务器都是通过dns地址来判断用户的区域来调度的
    [15/3/16 21:19:07] 蚂蚁-刘维: 所以问题在这里
    [15/3/16 21:19:18] 蚂蚁-刘维: 不劫持用户的使用效果才不受影响
    [15/3/16 21:19:25] 蚂蚁-刘维: 对于小网站没有影响
    [15/3/16 21:19:26] 军建: 理解了，，
    [15/3/16 21:19:45] 蚂蚁-刘维: 小网站就一个ip地址，比如我们的官网，官论坛
    [15/3/16 21:20:00] 蚂蚁-刘维: 你用全球哪个dns解析都是这个地址
    [15/3/16 21:20:12] 军建: 所以就没有差异
    [15/3/16 21:20:19] 军建: 嗯。
    [15/3/16 21:20:23] 蚂蚁-刘维: 访问的html速度一致，但是我们的图片用了cdn
    [15/3/16 21:20:36] 蚂蚁-刘维: 你访问我们网站的图片时，就不同了
    [15/3/16 21:20:46] 蚂蚁-刘维: 明白了吧？
    [15/3/16 21:20:51] 军建: 明白了


## shell字符串提取ip地址

    ip_1=${master_ipaddr%%.*}
    ip_tmp=${master_ipaddr#*.}
    ip_2=${ip_tmp%%.*}
    ip_tmp=${ip_tmp#*.}
    ip_3=${ip_tmp%%.*}
    ip_4=${master_ipaddr##*.}



