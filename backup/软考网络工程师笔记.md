# OSPF 邻接关系建立过程与状态分析

## （一）邻居状态概述


 OSPF** **共有 8 种状态机，分别是： Down、 Attempt、 Init、 2-way、 Exstart、 ExchLoading、 Full。**

- Down状态: 刚刚开始启用ospf 邻居会话/邻居会话超时，初始阶段 发起新一轮hello包
- attempt状态: 仅在NBMA网络 在NBMA网络中邻居是手动指定的，在该状态下，路由器使用HelloInterval取代PollInterval来发送Hello包
- init状态：收到链路对端设备的hello包且hello信息满足建立邻居的“4+1”条件，变为init状态 发送hello包（带有邻居的R-ID）
- two-way状态:从处于init状态邻居收到带有自己路由器的ID的hello立刻从init状态上升 2-way状态 完成邻居建立进入DR BDR竞选流程。
- Exstart状态: 完成DR和BDR选举后，发送DD报文     协商LSA更新阶段的主从关系和序列号MTU
- Exchange状态:主从关系协商完毕     主设备开始发送带有LSDB的简略信息的DD 报文 LSR请求自身设备没有的LSA
- Loading状态: 载入状态     回应请求的LSA （LSA LSU LSACK 交互发送）最后一个LSACK发完上升为full 状态
- Full状态: 向对端发送了最后一个lsack及收到对端的最后一个lsack后 本地设备状态切换为full状态 LSA同步更新完成，完成OSPF邻居的最终状态邻接状态（Full）

 

## （二）OSPF状态分析

 

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

 

（ 1） one-way 是什么状态，如何进入 two-way 当收到一个 HELLO 包中，没有包含自己
 的 router id，这时为 one-way，当收到的 HELLO 包中包含自己的 router id 则为 two-way
 （ 2） DR 的选举过程及时间：首先所有的路由器都会成为 DR others，先选举 BDR，
 BDR 发现没有 DR，自动成为 DR， DR others 发现没有 BDR，再选举出来一个 BDR。 DR 的选举时间等于 Dead time，只有在广播网络中才会选举 DR。
 （ 3） first DD 与 DD 的区别：首先 first DD 也称为空 DD，作用是选举出主从，并且统一一个序列号，保证数据库同步过程的有序可靠。
 （ 4） 如何选举主从，选举主从的作用：选择 router id 大的为主，接下来交互的 DD报文统一序列号，保证同步数据库同步的有序与可靠。

 

## （三）OSPF状态停留解析

1----------OSPF邻居状态停留在int状态 ?

int状态是本端收到一个符合邻居条件（满足4+1）的hello包后会将其在本端设置为int状态；如果本端对端不发送带有本设备ID的hello包，会一直停留在int状态，收到对方hello包带有本端ID会将邻居状态上升到2-way

2--------------OSPF邻居状态停留在2-way状态：在MA（广播网，就是咱们的以太网中）每一个网段链路上只有DR和BDR和其他成员会建立邻接状态（FULL）；其他OSPF设备只建立2-way在OSPF设备中，只有处于Full状态的成员之间才会交互LSAAR和BDR的选举依靠OSPF设备的接口优先级决定，默认为1 ，最大255，为0表示本设备不参与DR及BDR的选举。

 

3--------------OSPF邻居状态停留在exstart状态

（1）状态解析： OSPF邻居之间在交互LSA之前需要通过交互DD报文协商交换LSA这期间的主从关系，另外还需要协商MTU值 如果MTU值协商失败（两端接口MTU不一致）就会使得OSPF状态停留在exstart状态

-GigabitEthernet0/0/0]mtu 1400--------------------------------修改此接口的IP MTU为1500字节

[AR-1-GigabitEthernet0/0/0]ospf mtu-enable------------------------启用此接口下OSPF对MTU的参考

[AR-2-GigabitEthernet0/0/0]ospf mtu-enable-----------------------启用接口OSPF的MTU的参考（按照接口MTU值走） [AR-2-GigabitEthernet0/0/0]quit 注意：华为设备对OSPF的DD报文中参考值为0，意为忽略DD报文协商阶段的MTU值，无论多端设备的DD报文中携带多少都可以协商成功。

 

 

 

 

 

#  二-----OSPF报文概述

（1）---OSPF5种数据包的类型和作用描述

 

 

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image003.png)

 

 

\1.  hello包 用来发现邻居 建立邻居 维持邻居 （缺省下10秒一发，4倍hello超时）

 

\1.  DD（数据链路描述包）:协商MTU值 协商主从关系 描述LSDB简略信息（重传时间 5s）

DBD 分为 firstDBD 和 DBD
 <1>firstDBD 不携带 LSA 头部信息。通过 first DBD 确认主从关系。主的作用只是为了控制序列号的同步。 Router-ID 高的将成为主。
 <2>DBD 只携带 LSA 的头部信息，没有携带 LSA 的具体信息。承载完整 LSA 是 LSAUpdate 包。

 

\1.  LSR（链路状态请求包)：向邻居请求本设备缺省的LSA（重传时间 5S）

是不携带 LSA 头部的，只通过（通告 ID， LSA 类型， linkstate-ID）来请求具体的条目

\1.  LSU链路状态更新包：用于回应链路状态请求包 LSR, 而发送的更新包

含有真正 LSA 完整信息的，用来回应 LSRequest。

 

5.LSAck（链路状态确认包）：本路由器回应邻居确切收到我的LSU链路状态更新包

 

# 三-----影响 OSPF 邻接关系建立的因素（ 10条）

 

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image004.png)

 


## （ 1） Route-ID（ Route-ID 冲突导致的问题）

在同一区域内：
 R1 和 R2 及 R2 和 R3 都可以正常建立邻居，同步数据库的时候就会出现问题， R2 的lsdb 中， adv 为 1.1.1.1 的 lsa（ LSA1 和 LSA2）只有一份， 路由计算会出现问题。假设 R1 宣告（ network）一条路由 10.10.10.0/24， R1 会把这条 LSA（ adv=1.1.1.1， type=1LS ID=1.1.1.1， seq=80000001）发送给 R2， R2 收到后会发给他的邻居 R3， R3 收到发现通告者是 1.1.1.1，但是自己又没有这个网段，于是会给 R3 发送一个自己的 LSA1（ age=1s，seq=80000002）， R2 收到后会与之前 adv=1.1.1.1 的 LSA1 进行比较，选择这条 seq 更大的 LSA1，然后也会转发给 R1， R1 收到后发现自己有这个网段，又会发送一条新的 LSA1（ seq=80000003），会一直出现这样重复的情况，而导致路由动荡。
 假设 R1 引入一条路由 10.10.10.0/24， R1 会把这条 LSA（ adv=1.1.1.1， type=5， LSID=1.1.1.1， seq=80000001）发送给 R2， R2 收到后会发给他的邻居 R3， R3 收到发现通告
 者是 1.1.1.1，但是自己又没有这个网段，于是会给 R2 发送一个（ age=3600s， seq=80000001）的 LSA5， R2 收到后，会与之前收到的 LSA5 进行比较，因为 seq 和 check sum 与之前的一样，所以会优选 age=3600s 的，然后也会转发给 R1， R1 收到后发现自己有这个网段，

又会发送一条新的 LSA5（ seq=80000002），会一直出现这样重复的情况，而导致路由动荡。
 实验现象： R2 有时候有路由，有时候没路由，在一段时间后，有一台会自己修改router-id。

在不同区域;

 

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image005.png)

 

 

邻居关系正常, 区域内及区域间路由能学到进路由表。 如果 R1 和 R3 不引入外部路由的话，是不会出现问题的。因为 ospf 在区域间使用 LSA3， LSA3 是由区域的 ABR 根据LSA1、 LSA2 产生的， adv 是 ABR 的 router-id，区域间路由只是被当成叶子挂在 ABR 上，本区域内的 spt 树上不会出现在有相同 router-id 的节点，也就不会出现问题。但是如果在相同 router-id 的设备上做引入的时候就会出现问题了，因为 asbr 的 router-id 是需要被 ospf 域内的所有路由器所知道的，如果发现 asbr 的 router-id 与本设备的 router-id一样时，会出现问题

分析： 假设 R1 引入一条路由 10.10.10.0/24， R1 会把这条 LSA（ adv=1.1.1.1， type=5， LS ID=1.1.1.1， seq=80000001）发送给 R2， R2 收到后会发给他的邻居 R3， R3 收到发现通告者是 1.1.1.1，但是自己又没有这个网段，于是会给 R2 发送一个（ age=3600s， seq=80000001）的 LSA5， R2 收到后，会与之前收到的 LSA5 进行比较，因为 seq 和 check sum 与之前的一样，所以会优选 age=3600s 的，然后也会转发给 R1，R1 收到后发现自己有这个网段，又会发送一条新的 LSA5（ seq=80000002），会一直出现这样重复的情况，而导致路由动荡。

 

 

## （ 2） 接口区域 ID：
 区域 ID 包含在 ospf 头部，双方不一致时无法建立邻居


 ## （ 3） 认证：
 认证类型分为不认证（ 00），明文认证（ 01）和 MD5 认证（ 02）， OSPF 的认证放在 OSPF头中，所以 OSPF 一边接口认证，一边区域认证可以认证成功。

 

 

## （ 4） MA 网络掩码（为什么 p2p 中掩码可以不一致）：
 MA 网路中掩码必须一致，因为 MA 网络中所有路由器共用一个网段，只有一个 2Lsa 的Network 来描述当前的网络拓扑和网络号，所以当掩码不一致时，无法通过一个 2LSA 描述不同的掩码。 P2P 网络中掩码之所以可以不一致是因为 P2P 中有 1LSA 的 stub 类型来描述每一个网络的掩码信息，并且在 PPP 链路中 NCP 阶段，两台路由器会互推自己的 IP地址，并且以 32 位主机路由的方式加入自己的路由表，所以 P2P 网络中建立邻居不需要掩码一致。


## （ 5） MA 网络中优先级不能为零， DR 选举不成功。

 


## （ 6） 区域类型（ option 字段中的 E 位与 N 位）：

## （ 7） hello-dead 间隔（区别网络类型）

 

## （ 8） MTU（默认不检查，不一致时会停留 在哪种状态）：
 如果开启了 MTU 检查，如果双方 MTU 不一致，则小的一方停留在 Exstart 状态，另一方停留在 Exchange 阶段。

 


## （ 9） 网络类型

（四种，当两边不一致是否一定建立不了邻居，如果能建立会不会有问题，哪种网络类型发送单播，哪种发送组播）

答：双方网络类型不一致，不能建立 FULL 的邻接关系，但如果修改 hello， dead 时间,可以建立 full 的邻居关系（除了 NBMA 这种网络类型， NBMA 即使修改时间也无法和其他网络类型建立邻居关系，因为其收发 hello 报文都是单播）， MA 与 P2P、 P2MP 修改时间可以建立 FULL 的邻居关系， 但不能计算路由。

 

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image007.jpg)

 

 

## （ 10） silence（特点）：OSPF的静默接口
 OSPF 的 silence 接口，不收不发任何OSPF报文。

 

 

# 四----LSA详解


## （ 一） 列举各种 LSA 的 link-state ID； ADV；泛洪范围

 

 

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image009.jpg)

 

 

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image011.jpg)

 

 

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image013.jpg)

 

 

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image015.jpg)

 

 

## （二）LSA 详细内容

（ 1） 描述 1， 2 类 LSA 的作用
 首先 1类LSA，运行 OSPF 的每台路由器都会产生并且只产生一份 1LSA，泛洪范围为本区域，路由器根据不同的网络类型会产生不同的 1LSA，其中分为四种类型， P2P 描述的是 P2P 网络中的拓扑信息， Stub 描述的是路由信息（ 要知道什么时候会产生 stub），Transit 描述的是 MA 网络中的拓扑信息， v-link 描述的是做了虚链路的路由器的拓扑信息。

 

2类LSA 中描述的就是 MA 网络中的拓扑信息，主要表示的就是 MA 网络中的伪节点连接着哪几台路由器（ attched router 字段）。 1LSA 和 2LSA 只在区域内泛洪。所以 OSPF 中1LSA 和 2LSA 的作用就是描述当前网络中的拓扑信息及网络号以及开销。


## （ 三） LSA 的传播机制---触发更新+周期泛洪

触发更新：

当OSPF网络中有拓扑变更（增减网段进入或者邻居变更等）都会触发相应链路的拓扑或者网段的变更，这时直接感知到这一变化的OSPF设备会向邻居泛洪对应的LSA信息。

 

周期泛洪：
 LSA 的泛洪就是向水一样流出去，除了接收端口外向其他所有运行了 OSPF 接口泛洪。
 泛洪周期为 1800 秒，老化时间为 3600 秒，所有的 OSPF 路由器每 1800 秒把自己数据库中所有的 LSA 向外泛洪。当收到 3600 秒的 LSA，则直接删除数据库中对应的 LSA。OSPF 中通过三要素标识一条 LSA： LSA 的 Type， Link state ID， ADV routerOSPF通过序列号， checksum， age time 来判断 LSA 的新旧。序列号最小为 80000001，最大为 7fffffff，序列号越大代表 LSA 越新， checksum 这条 lan 的检验值，只要数据包不损坏， 一般不是对比， age time 越小越新，但当一台路由器收到两条 LSA，两条 LSA 的 agetime 的相差时间小于 900 秒（ 15 分钟），则认为两份 LSA 是相同的，则会保留先收到的 LSA，不收后收到的 LSA，主要是为了保证网络的稳定性，如果收到两条 LSA 的相差时间大于900 秒则会选择 age time 小的那份 LSA。

 
## （四） LSA中Forarding Address

FA 的作用，及产生条件， 5 类 LSA 携带 FA 与不携带 FA 的区别。
 FA 为 forwarding address， FA 的作用：解决次优和放环
 5L 的 FA 产生的条件：
 （ 1）下一跳非 P2P 和 P2MP
 （ 2）下一跳接口所在网段必须宣告进 OSPF
 （ 3）下一跳接口不能被 silent
 5L 中如果携带 FA 地址，则直接选择通过 FA 的地址去往目标网段， 如果没有携带 FA地址，则选择通过 ASBR 去往目标网段
 （ 5） 7 类 LSA 中的 P 位的作用
 为了将 NSSA 区域引入的外部路由发布到其它区域，需要把 Type7LSA 转化为
 Type5LSA 以便在整个 OSPF 网络中通告。
 •P-bit（ Propagate bit）用于告知转化路由器该条 Type7LSA 是否需要转化。
 •缺省情况下，转换路由器的是 NSSA 区域中 Router ID 最大的区域边界路由器（ ABR）。
 •只有 P-bit 置位并且 FA（ Forwarding Address）不为 0 的 Type7LSA 才能转化为 Type5LSA。 FA 用来表示发送的某个目的地址的报文将被转发到 FA 所指定的地址。
 •区域边界路由器产生的 Type7LSA 不会置位 P-bit。
 · NSSA 区域中的默认路由不会进行 7 转 5。

 

 

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image016.png)


 如上图所示，在边界路由器 R1 中引入的外部路由， R1 会向 nssa 区域产生一份 7 类 lsa，
 这份 7 类 lsa 中的 P 位置为 0，即使 R2 作为 7 转 5 的 ASBR， R2 也不会把这个 7 类 lsa 泛洪
 到 area 0 中。或者在 R3 上使用产生默认路由的命令， R3 就会成为 ASBR，产生一份 7 类 lsa，这个
 默认路由不会被边界路由器转成五类，不会向 area 0 产生 5 类 lsa，所以这条默认路由的 7类 lsa 中 P 位置为 0，只能自 nssa 区域传递。

 

 

## （五） 除了 LSA 的确认机制补充

每次LSA的更新通过LSU报文发出，接收者需要回送LSack对此次更新的LSA信息进行确认。

但是在MA网络中当Dother想DR/BDR更新LSU后 DR不会回送LSack

 

DR 可以进行隐式确认， 当 DR 收到 DR others 的 LSU 时，不需要回复 LSACK，因为 DR 会向其他的 DR others 发送 LSU 的更新，这就进行了隐式确认（ DR others 向 DR发送更新为 224.0.0.6， DR 向 DR others 发送的更新地址为 224.0.0.5）。

 

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image017.png)

 

 

 

 

# 五-----OSPF区域划分---减小 OSPF LSDB 的大小


 ## （ 1） 分区域设计：
 因为 1， 2LSA 只在本区域泛洪，分区域设计可以减少每个区域 1、 2 类 LSA 的数量
 ##  （ 2） 特殊区域：
 特殊区域无法传递 5LSA，可以减少 OSPF domain 中 5LSA 的数量

行了 SPF 计算后，进行路由过滤。
 ##  （ 4） 汇总：
 summary+no-advertise，汇总也可以执行过滤。需要注意做了虚链路的区域不能针对
 area0 的路由进行汇总，否则可能会产生环路
 （ 5） 另外 2 类 LSA 也可以减小 OSPF LSDB 大小的能力，但这不属于我们控制范围。

 

# 六-----OSPF特殊区域
在OSPF中，除了Stub，Totally Stub，NSSA，Totally NSSA；其他区域通常为普通区域（包括AREA 0骨干区域）
##  （1）---Stub-末节区域：
与AS外部没有太多路由通信失误边缘区域；过滤Type4 LSA和Type5 LSA，减少边界路由器的压力。
 放行LSA：Type1 LSA；Type2 LSA（仅广播网拥有此类LSA）；Type3 LSA拒绝LSA：Type4     LSA；Type5 LSA；Type7 LSA
##  （2）---Totally Stub-完全末节区域：
同样处于AS边缘；且只有一个连接其他区域的ABR，没有ASBR；没有虚连接穿越非骨干区域。
- 放行LSA：Type1 LSA；Type2 LSA（仅广播网拥有此类LSA）
- 拒绝LSA：Type3 LSA；Type4 LSA；Type5 LSA；Type7 LSA

 

##  （3）---NSSA----Not-So-Stubby     Area，非纯末梢区域：
可以位于非边缘区域，可以有多个ABR，可以有一个或多个ASBR；将ASBR引入的外部路由以Type7 LSA进入NSSA区域并在本NSSA区域泛洪，然后在ABR上转换为Type5 LSA后以自己的身份发布到区域外。
- 放行LSA：Type1 LSA；Type2 LSA（仅广播网拥有此类LSA）；Type3 LSA
- 拒绝LSA：Type4 LSA；Type5 LSA；
- -Totally NSSA完全非纯末梢区域：
- 可以位于非边缘区域，可以有多个ABR，可以有一个或多个ASBR；将ASBR引入的外部路由以Type7 LSA进入NSSA区域并在本NSSA区域泛洪，然后在ABR上转换为Type5 LSA后以自己的身份发布到区域外。
- 放行LSA：Type1 LSA；Type2 LSA（仅广播网拥有此类LSA）
- 拒绝LSA：Type3 LSA；Type4 LSA；Type5 LSA；

 

 


# 六-----OSPF 路由选路的原则，及在什么情况下会负载


 （ 1） 选路原则：区域内的>区域间的>TYPE1>TYPE2
 （ 2） 外部路由负载条件： 1 cost 一致， 2 区域一致
 （ 3） 如下图： R4 与 R5 上分别引入外部路由，问 R2 如何去往这两条外部路由。

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image019.jpg)

 

R2 如果想要访问 R5：

R2 会通过 area0 进行访问，因为非骨干区域传来的 4LSA， ABR
 不参与计算， R2 会选择 R1 作为他的下一跳

R2 如果想要访问 R4：

R2 会通过 area1 进行访问，因为 R2 会通过 area0 和 area1 都收到 R4 的 1LSA， R2 上去往 R4 会有两个下一跳，但两个下一跳属于不同的区域，所以 R2 去往 R4 不能负载，如果 R2 通过两条路的 cost 值相同， R2 则会选择区域号大的作为 R2 的下一跳， R2 会选择 R3 作为下一跳。

 

# 七-----4 类 LSA 的作用及场景**


 作用：描述 ABR 到 ASBR 的 cost，只要与 ASBR 不在同一个区域，就会产生 4 类 lsa，
 用于告诉本区域内路由器 ASBR 的位置。
 场景：

![img](file:///C:/Users/xyu/AppData/Local/Temp/msohtmlclip1/01/clip_image021.jpg)

 

R1 上引入一条路由，问题：
 1） 当区域 1 是普通区域时，哪些区域里面有 4 类 lsa, R8 上面有几条 4 类 LSA？

 

\1.  Area 0和Area 0这两个区域存在4类LSA

在非ASBR所在的区域需要通过4类LSA来确定ASBR的拓扑信息，是的收到ASBR通过的描述外部路由LSA的OSPF设备计算是确定“下一跳”。

 

（2）R8存在2条4类LSA

常规区域所在的ABR都会向本区域的成员下发4类LSA


 2） 当区域 1 是 NSSA 区域时，哪些区域里面有 4 类 LSA,R8 上面有几条 4 类 LSA？

\1.  只有区域2存在4类LSA

①NSSA区域内通过所在区域的1，2类LSA可以确定ASBR所在的位置，无需4类LSA。

②当描述外部网络的7类LSA离开NASS区域后会被NSSA区域所在的ABR进行“7转5”，通过5类LSA形势通告给OSPF自治系统的其他常规区域。此时这些“新生”的5类LSA是ABR产生通告，进行“7转5”动作的ABR会被其他区域的成员判定为ASBR，骨干区域的成员依然通过1类2类LSA确定此ASBR的位置。

③区域2没有骨干区域的1类和2类LSA，所以需要区域2所在的ABR下发4类LSA来描述ASBR的拓扑信息。

 

\1.  R8存在4条4类LSA信息

这4条中没2条用来描述同一台ASBR的拓扑信息，即2条描述的内容是3.3.3.3，另外2条描述4.4.4.4.

\1.  常规区域的ABR会各自通告描述ASBR的的4L类LSA信息，区域2有2台ABR。所以针对同一台ASBR通告了2条4类LSA。

\2.  NSSA区域中的ABR都具备“7转5”的能力，默认情况下存在多个ABR时会选取R-ID最大的ABR进行“7转5”向其他区域通过描述外网信息的5类LSA。

\3.  需要注意的是这并不影响NSSA区域的其他ABR成为一台ASBR，因为这些ABR具备产生5类LSA的能力和作为去往自治系统外部的能力

 

\- -
 # 八-----OSPF V2 与 V3 的区别

 

（ 1） V2 有认证， V3 无认证（通过 ipv6 实现）
 （ 2） V2 基于 IP， V3 基于链路
 （ 3） V3 实现了拓扑与路由的分离（ 1， 2LSA 中不再有网络信息）
 （ 4） V3 头部增加了实例号字段，可以实现一个接口配置多个进程
 （ 5） 报文发送的目的地址不同
 （ 6） V3 必须手工指定 router-id
 （ 7） 增加了两类 LSA， Type8: Link-LSA； Type9: Intra-Area-Prefix-LSA （ 需要明白这两条 LSA
 的作用）

 


 # 九------OSPFv3 和 OSPFv2 协议比较如下：
 相同点：
 1> 网络类型和接口类型。

54
 2> 接口状态机和邻居状态机。
 3> 链路状态数据库（ LSDB）。
 4> 洪泛机制（ Flooding mechanism）。
 5> 相同类型的报文： Hello 报文、 DD 报文、 LSR 报文、 LSU 报文和 LSAck 报文。
 6> 路由计算基本相同。
 不同点：
 1> OSPFv3 基于链路，而不是网段。
 OSPFv3 运行在 IPv6 协议上， IPv6 是基于链路而不是基于网段的。在配置 OSPFv3 时，
 不需要考虑是否配置在同一网段，只要在同一链路，就可以不配置 IPv6 全局地址而直接
 建立联系。
 2> OSPFv3 上移除了 IP 地址的意义。
 这样做的目的是为了使“拓扑与地址分离”。 OSPFv3 可以不依赖 IPv6 全局地址的配置
 来计算出 OSPFv3 的拓扑结构。 IPv6 全局地址仅用于 Vlink 接口。
 3> OSPFv3 的报文及 LSA 格式发生改变。
 OSPFv3 报文不包含 IP 地址。 OSPFv3 的 Router LSA 和 Network LSA 里不包含 IP 地
 址。 IP 地址部分由新增的两类 LSA（ LinkLSA 和 Intra Area Prefix LSA）宣告。 OSPFv3 的Router ID、 Area ID 和 LSA Link State ID 不再表示 IP 地址，但仍保留 IPv4 地址格式。广播、NBMA 及 P2MP 网络中，邻居不再由 IP 地址标识，只由 Router ID 标识。
 4> OSPFv3 的 LSA 报文里添加 LSA 的洪泛范围。
 OSPFv3 在 LSA 报文头的 LSA Type 里，添加 LSA 的洪泛范围，这使得 OSPFv3 的路由器更加灵活，可以处理不能识别的 LSA：
 OSPFv3 可以存储或洪泛不识别的报文，而 OSPFv2 只简单丢弃掉不识别的报文。 OSPFv3允许洪泛范围为区域或链路本地，并且设置 U 位（报文可按洪泛范围为链路本地来处理）的不识别报文存储或通过 stub 区域。
 例如： R1 和 R2 都可识别某类 LSA，它们之间通过 R3 连接，但 R3 不识别该类 LSA。这样，当 R1 洪泛此类 LSA 时， R3 虽然不识别，但还是可以洪泛给 R2， R2 收到后继续处理。5> OSPFv3 支持一个链路上多个进程。一个 OSPFv2 物理接口，只能和一个 OSPFv2 实例绑定。但是一个 OSPFv3 的物理接口，
 可以和多个 OSPFv3 实例绑定，并用不同的 Instance ID 区分。这些运行在同一条物理链路上的多个 OSPFv3 实例，分别与链路对端设备建立邻居及发送报文，且互不干扰。这样可以充分共享同一链路资源。
 6> OSPFv3 利用 IPv6 链路本地地址。
 IPv6 使用链路本地地址在同一链路上发现邻居及自动配置等。运行 IPv6 的路由器不转发目的地址为链路本地地址的 IPv6 报文，此类报文只在同一链路有效。链路本地单播地
 址从 FE80/10 开始。OSPFv3 是运行在 IPv6 上的路由协议，同样适用链路本地地址来维持邻居，同步 LSA数据库。除 Vlink 外的所有 OSPFv3 接口都使用链路本地地址作为源地址及下一跳来发送OSPFv3 报文。
 这样做的好处是：不需要配置 IPv6 全局地址，就可以得到 OSPFv3 拓扑，实现拓扑与地址分离。通过在链路上泛洪的报文不会传到其他链路上，来减少报文不必要的泛洪来节省带宽。
 7> OSPFv3 移除所有认证字段。
 OSPFv3 的认证直接使用 IPv6 的认证及安全处理，不再需要其自身来完成认证，使用协议时只需关注协议本身即可。
 8> 新增两种 LSA。
 Link LSA：用于路由器宣告各个链路上对应的链路本地地址及其所配置的 IPv6 全局地址，仅在链路内洪泛。 Intra Area Prefix LSA：用于向其他路由器宣告本路由器或本网络（广播网络及 NBMA）的 IPv6 全局地址信息，在区域内洪泛。
 9> OSPFv3 只通过路由器 ID 来标识邻居。
 OSPFv2 在广播网络、 NBMA 及 P2MP 网络中是通过 IPv4 接口地址来标识的。 OSPFv3
 只通过 Router ID 来标识邻居，这样即使没有配置 IPv6 全局地址，或者 IPv6 全局地址配置都不在同一网段， OSPFv3 的邻居还是可以建立并维护的，以达到“拓扑与地址分离”的目的。

 