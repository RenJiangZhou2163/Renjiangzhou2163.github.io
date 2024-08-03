# 可靠性

## 1. BFD的配置规范

### 1.1. BFD for LSP会话报文来回路径需要一致才能保证LSP路径正常切换

配置BFD for LSP会话检测时，从源端设备到宿端设备的LSP路径与回程的LSP或者IP路径不一致。回程LSP或者IP路径故障时，BFD会话Down，导致LSP路径误切换。

#### 1.1.1. 应用场景

如[图1-1](https://support.huawei.com/enterprise/zh/doc/EDOC1000124120?idPath=24030814|9856750|22715517|9858933|15837&section=j003#fig_dc_vrp_precaution_001401)所示，RouterA与RouterB之间存在两条路径，路径1和路径2。

**图1-1** BFD for LSP会话报文来回路径不一致导致LSP路径误切换组网图

![img](https://raw.githubusercontent.com/RenJiangZhou2163/PicGo/main/Blogs/Pictures/1001.png)

#### 1.1.2. 配置规范

在网络部署BFD for LSP会话检测时：

1. 动态BFD for LSP会话，可以考虑配置路由约束方式，保证来回路径一致，例如使用高优先级的静态路由。
2. 静态BFD for LSP会话（不包括BFD for TE-LSP），可以考虑配置路由约束方式，保证来回路径一致，例如使用高优先级的静态路由。
3. 静态BFD for TE-LSP会话，通过严格显式路径约束LSP来回路径，且来回路径均配置BFD for TE-LSP类型会话，保证来回路径一致。

#### 1.1.3. 非规范配置的风险

**风险描述**

1. 配置动态BFD for LSP会话检测，去程路径1为LSP路径，回程路径2为IP路径。
2. 配置静态BFD for LSP会话（不包括BFD for TE-LSP），回程路径2为LSP路径，且与RouterA到RouterB的LSP路径不共路。
3. 路径1和路径2都配置BFD for TE-LSP会话，RouterA和RouterB的的TE-LSP没有配置严格的显式路径。

当满足上述任意一个条件时，BFD会话去程和回程路径不一致，回程路径故障时，可能导致去程LSP路径误切换。

业务现象如下：

BFD会话本意用来检测路径1，但是当路径2故障时，由于BFD回程路径不通，会话Down，触发路径1上的LSP误切换到故障路径2上，导致业务流量丢失。

**风险的判断方法**

1. 下面以动态BFD会话去程为LSP路径，回程为IP路由为例。其它情况的判断方法请根据实际组网情况而定，必须确保BFD会话报文的去程和回程路径一致。

2. 在RouterA查询BFD会话邻居信息。

   \# 在用户视图下，执行 **display bfd session** **all** **verbose** 命令查看RouterA到RouterB的BFD邻居信息。加粗字体为RouterA到RouterB的BFD会话的邻居和下一跳。

   ```
   <HUAWEI> display bfd session all verbose
   --------------------------------------------------------------------------------
     State : Up                    Name : dyn_16396
   --------------------------------------------------------------------------------
     Local Discriminator    : 16396            Remote Discriminator   : 16392 
     Session Detect Mode    : Asynchronous Mode Without Echo Function
     BFD Bind Type          : TE_LSP 
     Bind Session Type      : Dynamic  
     Bind Peer IP Address   : 2.2.2.2  
     NextHop Ip Address     : 10.1.1.2        
     ......
   ```

3. 在RouterB查询BFD会话的邻居信息。

   \# 在用户视图下，执行 **display bfd session** **all** **verbose** 命令查看RouterB到RouterA的BFD邻居信息。加粗字体为RouterB到RouterA的BFD会话的邻居。

   ```
   <HUAWEI> display bfd session all verbose 
   --------------------------------------------------------------------------------
     (Multi Hop) State : Up                    Name : dyn_16392
   --------------------------------------------------------------------------------
     Local Discriminator    : 16392            Remote Discriminator   : 16396 
     Session Detect Mode    : Asynchronous Mode Without Echo Function
     BFD Bind Type          : Peer IP Address  
     Bind Session Type      : Entire_Dynamic  
     Bind Peer IP Address   : 1.1.1.1             
     ......
   ```

4. 在RouterB查询到RouterA的路由信息。

   在用户视图下，执行 **display ip routing-table** *ip-address mask* **verbose** 命令查询RouterB到RouterA的路由信息。加粗字体为RouterB到RouterA的路由下一跳。BFD会话从RouterA到RouterB的下一跳为10.1.1.2，而从RouterB到RouterA的下一跳为10.2.1.2，BFD会话报文来回路径不一致。

   ```
   <HUAWEI> display ip routing-table 1.1.1.1 32 verbose
   Route Flags: R - relay, D - download to fib, T - to vpn-instance, B - black hole route
   ------------------------------------------------------------------------------
   Routing Table : _public_
   Summary Count : 1
   Destination: 1.1.1.1/32          
        Protocol: ISIS-L2         Process ID: 1              
      Preference: 15                    Cost: 10             
         NextHop: 10.2.1.2         Neighbour: 0.0.0.0        
           State: Inactive Adv           Age: 1d04h15m09s         
             Tag: 0                 Priority: high           
           Label: NULL               QoSInfo: 0x0           
      IndirectID: 0xE600087A    
    RelayNextHop: 0.0.0.0          Interface: GigabitEthernet0/5/5
        TunnelID: 0x0                  Flags:   
   ```

**风险的恢复方案**

请按照配置规范进行配置。



### 1.2. 静态BFD两端设备配置对称实现流量不中断

配置静态BFD会话检测，两端设备配置不对称导致业务切换行为不一致，流量中断。

#### 1.2.1. 应用场景

如[图1-2](https://support.huawei.com/enterprise/zh/doc/EDOC1000124120?idPath=24030814|9856750|22715517|9858933|15837&section=j003#fig_dc_vrp_precaution_001501)所示，配置静态BFD会话检测RouterA和RouterB之间的链路。

**图1-2** 静态BFD两端设备配置不对称导致流量中断组网图

![img](https://raw.githubusercontent.com/RenJiangZhou2163/PicGo/main/Blogs/Pictures/1002.png)


#### 1.2.2. 配置规范

在对称的静态BFD会话上配置对称的本地行为参数：

- 两端设备分别在BFD会话视图下执行 **wtr** *wtr-value* 命令配置相同的等待恢复时间。
- 两端设备分别在BFD会话视图下执行 **process-interface-status** 命令配置当前BFD会话与其绑定的接口进行状态联动。
- 两端设备分别在BFD会话视图下执行 **process-pst** 命令配置允许BFD会话修改端口状态表PST。

#### 1.2.3. 非规范配置的风险

**风险描述**

当不满足配置规范的任意一个条件时，两端业务切换行为不一致，流量中断。

**风险的判断方法**

1. 查询静态BFD会话的**wtr** *wtr-value*信息。

   \# 在RouterA的BFD会话视图下，执行 **display this** 命令进行查询。 **discriminator local** 必须要求与RouterB上远端描述符匹配。RouterA配置了wtr且时间为2分钟。

   ```
   <HUAWEI> system-view
   [HUAWEI] bfd session a
   [HUAWEI-bfd-session-a] display this
   #
   bfd a bind peer-ip 10.1.1.2
    discriminator local 1  
    discriminator remote 2
    wtr 2 
    commit
   #
   return
   ```

   \# 在RouterB的BFD会话视图下，执行 **display this** 命令进行查询。 **discriminator local** 必须与RouterA上远端描述符匹配。RouterB没有配置wtr，与RouterA配置不一致。

   ```
   <HUAWEI> system-view
   [HUAWEI] bfd session a
   [HUAWEI-bfd-session-a] display this
   #
   bfd a bind peer-ip 10.1.1.1
    discriminator local 2
    discriminator remote 1  
    commit
   #
   return
   ```

2. 查询静态组播BFD会话的两端设备 **process-interface-status** 信息。

   \# 在RouterA的BFD会话视图下，执行 **display this** 命令进行查询。 **discriminator local** 必须与RouterB上远端描述符相匹配。RouterA配置了端口联动。

   ```
   <HUAWEI> system-view
   [HUAWEI] bfd session a
   [HUAWEI-bfd-session-a] display this
   #
   bfd a bind peer-ip default-ip interface GigabitEthernet1/0/0
    discriminator local 11  
    discriminator remote 12
    process-interface-status  
    commit
   #
   return
   ```

   \# 在RouterB的BFD会话视图下，执行 **display this** 命令进行查询。 **discriminator local** 必须与RouterA上远端描述符相匹配。RouterB没有配置端口联动，与RouterA配置不一致。

   ```
   <HUAWEI> system-view
   [HUAWEI] bfd session a
   [HUAWEI-bfd-session-a] display this
   #
   bfd a bind peer-ip default-ip interface GigabitEthernet1/0/0
    discriminator local 12
    discriminator remote 11  
    commit
   #
   return
   ```

3. 查询的静态BFD会话的两端**process-pst**信息。

   \# 在RouterA的BFD会话视图下，执行 **display this** 命令进行查询。 **discriminator local** 必须与RouterB上远端描述符相匹配。RouterA配置了pst联动。

   ```
   <HUAWEI> system-view
   [HUAWEI] bfd session a
   [HUAWEI-bfd-session-a] display this
   #
   bfd a bind peer-ip 10.1.1.2 interface GigabitEthernet1/0/0
    discriminator local 11  
    discriminator remote 12
    process-pst 
    commit
   #
   return
   ```

   \# 在RouterB的BFD会话视图下，执行 **display this** 命令进行查询。**discriminator local**必须与RouterA上远端描述符相匹配。RouterB没有配置pst联动，与RouterA配置不一致。

   ```
   <HUAWEI> system-view
   [HUAWEI] bfd session a
   [HUAWEI-bfd-session-a] display this
   #
   bfd a bind peer-ip 10.1.1.1 interface GigabitEthernet1/0/0
    discriminator local 12
    discriminator remote 11 
    commit
   #
   return
   ```

**风险的恢复方案**

请按照配置规范进行配置。

### 1.3. BFD描述符配置规范

配置静态BFD会话的情况下，在路由相互可达的网络设备上很可能出现配置的BFD会话远端描述符冲突，导致BFD会话震荡。

#### 1.3.1. 应用场景

如[图1-3](https://support.huawei.com/enterprise/zh/doc/EDOC1000124120?idPath=24030814|9856750|22715517|9858933|15837&section=j003#fig_dc_vrp_precaution_001601)所示，配置静态BFD会话检测网络上两个网络节点RouterA和RouterB间链路。

**图1-3** BFD描述符配置冲突导致BFD会话周期性震荡组网图
![img](https://raw.githubusercontent.com/RenJiangZhou2163/PicGo/main/Blogs/Pictures/1003.png)

#### 1.3.2. 配置规范

在部署网络BFD会话检测时，描述符需要统一规划，避免冲突。

#### 1.3.3. 非规范配置的风险

**风险描述**

1. RouterB配置静态BFD和RouterA建立会话，并在BFD视图下执行**discriminator** **local** *discr-value* 命令配置本地描述符为a；
2. RouterC上配置一个静态BFD会话，并在BFD视图下执行**discriminator** **remote** *discr-value* 命令配置其远端描述符也为a。

当满足上述条件时，BFD会话周期性震荡，引起绑定BFD会话的业务震荡。

业务现象如下：

BFD会话会周期性震荡，且查看两端设备中BFD会话Down的原因都是因为邻居Down。

**风险的判断方法**

1. 查询已经配置的静态BFD会话的两端配置信息。

   \# 在RouterB的BFD会话视图下，执行**display this**命令。**discriminator local** 必须与RouterA上远端描述符相匹配。

   ```
   <HUAWEI> system-view
   [HUAWEI] bfd session a
   [HUAWEI-bfd-session-a] display this
   #
   bfd a bind peer-ip 10.1.1.2
    discriminator local 1  
    discriminator remote 2
    commit
   #
   return
   ```

   \# 在RouterA的BFD会话视图下，执行**display this**命令。**discriminator local** 必须与RouterB上远端描述符相匹配。

   ```
   <HUAWEI> system-view
   [HUAWEI] bfd session a
   [HUAWEI-bfd-session-a] display this
   #
   bfd a bind peer-ip 10.1.1.1
    discriminator local 2
    discriminator remote 1  
    commit
   #
   return
   ```

2. 查询BFD会话的详细信息。

   \# 在用户视图下，执行**display bfd session all verbose** 命令查询RouterA的会话详细信息。BFD会话邻居状态为Down。

   ```shell
   <HUAWEI> display bfd session all verbose 
   --------------------------------------------------------------------------------
     (Multi Hop) State : Down                  Name : a
   --------------------------------------------------------------------------------
     Local Discriminator    : 1                Remote Discriminator   : 2 
     Session Detect Mode    : Asynchronous Mode Without Echo Function
     BFD Bind Type          : Peer IP Address  
     

     Active Multi           : -                  
     Last Local Diagnostic  : Neighbor Signaled Session Down
     ……
   ```

   \# 在用户视图下，执行**display bfd session all verbose** 命令查询RouterB的会话详细信息。BFD会话邻居状态为Down。

   ```shell
   <HUAWEI> display bfd session all verbose 
   --------------------------------------------------------------------------------
     (Multi Hop) State : Down                  Name : a
   --------------------------------------------------------------------------------
     Local Discriminator    : 2                Remote Discriminator   : 1 
     Session Detect Mode    : Asynchronous Mode Without Echo Function
     BFD Bind Type          : Peer IP Address  
     

     Active Multi           : -                  
     Last Local Diagnostic  : Neighbor Signaled Session Down
   ……
   ```

3. 查询网络中其他设备上的BFD会话配置。

   \# 网络中与RouterA和RouterB有路由连接的其他设备（RouterC）上，在用户视图下执行**display current-configuration configuration bfd-session**命令查询网络中的其他设备上是否存在本地描述符冲突的BFD会话。**discriminator remote**与B设备上远端描述符冲突。

   ```shell
   <HUAWEI> display current-configuration configuration bfd-session 
   #
   bfd a bind peer-ip 20.1.1.1
    discriminator local 5
    discriminator remote 1  
    commit
   #
   return
   ```

**风险的恢复方案**

修改RouterC上的BFD会话的远端描述符为其他值。

### 1.4. 静态BFD for CR-LSP隧道往返路径需要保持一致才能实现业务正常切换

静态BFD检测CR-LSP时，如果往返路径不一致，可能导致BFD检测Down，触发业务误切换，可能导致业务中断。

#### 1.4.1. 应用场景

配置静态BFD for CR-LSP。

#### 1.4.2. 配置规范

配置静态BFD for CR-LSP时，配置往返TE隧道的显示路径一致。

#### 1.4.3. 非规范配置的风险

**风险描述**

当静态BFD检测的CR-LSP往返路径不一致时，BFD检测可能Down，检测结果不能反映实际的CR-LSP的连通性。

业务现象如下：

BFD检测Down，触发业务误切换，可能导致业务中断。

**风险的判断方法**

1. 在用户视图下，执行**display current-configuration** **bfd-session**命令，查看BFD配置。

   由显示信息可以看出，Tunnel0/0/27的主LSP本地标识符为local1，远端标识符为remote2；Tunnel0/0/27的备LSP本地标识符为local3，远端标识符为remote4。

   ```shell
   <HUAWEI> display current-configuration configuration bfd-session
#
   bfd tunnel1 bind mpls-te interface Tunnel0/0/27 te-lsp   
    discriminator local 1
    discriminator remote 2 
    process-pst
    commit
   #
   bfd tunnel1-back bind mpls-te interface Tunnel0/0/27 te-lsp backup 
    discriminator local 3 
    discriminator remote 4
    process-pst
    commit
   #
   return
   ```
   
2. 在用户视图下，执行**display current-configuration** **interface Tunnel**命令，查看TE隧道配置。

   由显示信息可以看出，隧道目的地址为**192.168.1.1**，主LSP的显示路径为**main-to-devicea**，备LSP的显示路径为**backup-to-devicea**。

   ```shell
   <HUAWEI> display current-configuration interface Tunnel 0/0/27
   #
   interface Tunnel0/0/27
    description huawei
    mtu 1600
    ip address unnumbered interface LoopBack1
    tunnel-protocol mpls te
    destination 192.168.1.1 
    mpls te tunnel-id 27
    mpls te record-route label
    mpls te path explicit-path main-to-devicea
    mpls te path explicit-path backup-to-devicea secondary
    mpls te backup hot-standby mode revertive wtr 60 
    mpls te backup ordinary best-effort 
    mpls te igp shortcut
    mpls te igp metric absolute 10
    mpls te commit
    isis enable 100
    statistic enable #
   return
   ```

3. 在用户视图下，执行**display mpls te tunnel-interface** *tunnel-name*命令，查看隧道的三元组信息。

   由显示信息可以看出，Tunnel 0/0/27的Session ID为27，Ingress LSR ID为192.168.1.2，主LSP ID为3，备LSP ID为32772。

   ```shell
   <HUAWEI> display mpls te tunnel-interface Tunnel 0/0/27
   ----------------------------------------------------------------
                        Tunnel0/0/27
   ----------------------------------------------------------------
    Tunnel State Desc   :  UP
    Active LSP          :  Primary LSP
    Session ID          :  27 
    Ingress LSR ID    :  192.168.1.2    Egress LSR ID:  192.168.1.1
    Admin State         :  UP               Oper State   :  UP
    Primary LSP State      : UP
    Main LSP State       : READY               LSP ID  : 3 
    Hot-Standby LSP State  : UP
    Main LSP State       : READY               LSP ID  : 32772 
   ```

4. 查询隧道实际经过的路径。查询到的路径用于后面和对端设备查询的路径进行比较。

   执行**display current-configuration** **interface tunnel 0/0/27**命令，查看是否配置了**mpls te record-route**（或**mpls te record-route label**），如果配置了，执行**display mpls te tunnel path** **lsp-id** *ingress-lsr-id* *session-id* *local-lsp-id*命令，查询隧道实际经过的路径。

   否则，执行**tracert lsp te** **tunnel***interface-number* 命令查询。

   ```shell
   <HUAWEI> display mpls te tunnel path lsp-id 192.168.1.2 27 3
    Tunnel Interface Name : Tunnel0/0/27
    Lsp ID : 192.168.1.2 :27 :3 
    Hop Information 
     Hop 0   192.168.1.2 
     Hop 1   100.0.2.7 
     Hop 2   100.0.2.8 
     Hop 3   192.168.1.1

   <HUAWEI> tracert lsp te Tunnel 0/0/27
     LSP Trace Route FEC: TE TUNNEL IPV4 SESSION QUERY Tunnel0/0/1, press CTRL_C to break.
     TTL   Replier            Time    Type      Downstream 
     0                                Ingress   192.168.1.2/[3 ]
     1     192.168.1.1       32 ms   Egress 
   ```

5. 在用户视图下，使用命令行**display explicit-path** *path-name*，查看隧道的显式路径配置。

   - 如果设备上没有配置显式路径或隧道配置中没有使用显式路径，建议配置严格显式路径并在隧道下配置使用。
   - 如果配置的显式路径与隧道实际经过的路径相比，缺少某些跳，建议修改显式路径的配置，将其补充完整。
   - 如果显式路径配置为松散模式，建议修改为严格模式，并将路径补充完整。

   ```shell
   <HUAWEI> display explicit-path main-to-devicea
   1      62.231.253.26       Strict      Include                       
   2      62.231.253.166      Strict      Include                       
   3      62.231.253.77       Strict      Include
   4      62.231.253.133      Strict      Include                       
   5      62.231.253.53       Strict      Include 

   <HUAWEI> display explicit-path backup-to-devicea
   1      62.231.253.162      Strict      Include                       
   2      62.231.253.73       Strict      Include                       
   3      62.231.253.129      Strict      Include 
   ```

6. 根据TE隧道目的地址（192.168.1.1），找到TE隧道尾节点的设备。在TE隧道尾节点设备上，根据头节点静态BFD for CR-LSP的标示符，找到静态BFD for CR-LSP信息。

   用户视图下执行**display bfd configuration** **discriminator** *local-discr-value* **verbose**命令，其中参数*local-discr-value*指定头节点静态BFD for CR-LSP会话中Remote Discriminator的值（2），找到对应TE隧道(Tunnel0/0/28)。

   ```shell
   <HUAWEI> display bfd configuration discriminator 2 verbose
   --------------------------------------------------------------------------------
     BFD Session Configuration Name : to_3           
   --------------------------------------------------------------------------------
     Local Discriminator    : 2              Remote Discriminator   : 1       
     BFD Bind Type          : TE_LSP                                              
     Bind Session Type      : Static                                              
     Bind Interface         : Tunnel0/0/28    TE LSP Type            : Primary   
     TOS-EXP                : 7                Local Detect Multi     : 3         
     Min Tx Interval (ms)   : 10              Min Rx Interval (ms)   : 10       
     WTR Interval (ms)      : -                Process PST            : Enable    
     Proc Interface Status  : Disable                                             
     Bind Application       : LSPM | L2VPN 
     Session Description    : -                                                   
   --------------------------------------------------------------------------------    
   ```

7. 在TE隧道尾节点设备上，针对上一步获取到的隧道名称（如例子中的Tunnel0/0/28），根据上述步骤，找出该隧道实际经过的路径，并与查到Tunnel0/0/28路径进行比较。

   如果一致（方向相反），则不存在问题。如果不一致，则说明往返路径不一致。

**风险的恢复方案**

修改显式路径，将隧道实际经过的路径调整为与TE隧道头节点上对应隧道的路径一致。

### 1.5. 内层链路的BFD检测间隔应小于外层链路的检测间隔

多层保护场景下，如果内层链路的BFD检测间隔大于外层链路的检测间隔，当外层链路感知故障时，内层保护尚未切换。

#### 1.5.1. 应用场景

配置了多层保护的场景。例如，配置BFD for RSVP、BFD for CR-LSP或BFD for TE的场景。

#### 1.5.2. 配置规范

在网络部署BFD会话检测时，存在多层保护场景情况下，对BFD检测间隔的配置遵循内层小外层大的原则。

不同场景下，BFD检测间隔的配置不同。例如：

- BFD for RSVP场景：请参见（可选）调整BFD检测参数。
- 静态BFD for CR-LSP场景：请参见配置入节点BFD参数和配置出节点BFD参数。
- 动态BFD for CR-LSP场景：请参见（可选）调整入节点的BFD检测参数。
- BFD for TE场景：请参见配置入节点BFD参数和配置出节点BFD参数。

#### 1.5.3. 非规范配置的风险

**风险描述**

多层保护场景下，内层链路的BFD检测间隔大于外层链路的检测间隔，当Tunnel出现故障时，检测外层Tunnel的BFD会话由于检测间隔小，则先感知到链路故障，通知Tunnel业务做保护切换，因此浪费了内层的检测LSP的保护切换。

内层保护机制浪费、整体切换方案退化为只剩外层冗余保护，切换性能也随之退化。

业务现象如下：

外层链路已经感知故障，而内层保护尚未切换，如果外层链路配置了保护路径，则可能出现保护切换退化为外层保护，因此浪费了内层的保护切换策略。

**风险的判断方法**

如下判断方法适用内层为动态BFD for CR-LSP，外层为静态BFD for TE，且BFD for CR-LSP和BFD for TE绑定同一个Tunnel接口的场景。其它场景下，请根据实际情况来进行判断。

在用户视图下，执行**display bfd session all verbose**命令查看BFD的检测间隔。

从显示信息可以看出，检测内层链路的BFD Bind Type为TE_LSP的会话的检测间隔Detect Interval (ms)为2664，而检测外层链路的BFD Bind Type为TE_TUNNEL的会话的检测间隔Detect Interval (ms)为300，即内层链路的BFD检测间隔大于外层链路的检测间隔。

```shell
<HUAWEI> display bfd session all verbose
--------------------------------------------------------------------------------
  State : Up                    Name : dyn_16393
--------------------------------------------------------------------------------
  Local Discriminator    : 16393            Remote Discriminator   : 16402 
  Session Detect Mode    : Asynchronous Mode Without Echo Function
  BFD Bind Type          : TE_LSP 
  Bind Session Type      : Dynamic  
  Bind Peer IP Address   : 2.2.2.2         
  NextHop Ip Address     : 10.1.1.2 
  Bind Interface         : Tunnel1          TE LSP Type            : Primary 
  Tunnel ID              : 33  
  FSM Board Id           : 9                TOS-EXP                : 6
  Min Tx Interval (ms)   : 999              Min Rx Interval (ms)   : 888 
  Actual Tx Interval (ms): 999              Actual Rx Interval (ms): 888 
  Local Detect Multi     : 48               Detect Interval (ms)   : 2664 
  Echo Passive           : Disable          Acl Number             : - 
  Destination Port       : 3784             TTL                    : 1 
  Proc Interface Status  : Disable          Process PST            : Enable     
  WTR Interval (ms)      : -                Config PST             : Enable     
  Active Multi           : 3   
  Last Local Diagnostic  : No Diagnostic
  Bind Application       : TE
  Session TX TmrID       : -                Session Detect TmrID   : - 
  Session Init TmrID     : -                Session WTR TmrID      : - 
  Session Echo Tx TmrID  : -   
  Session Description    : - 
--------------------------------------------------------------------------------

--------------------------------------------------------------------------------
  State : Up                    Name : te
--------------------------------------------------------------------------------
  Local Discriminator    : 111              Remote Discriminator   : 111 
  Session Detect Mode    : Asynchronous Mode Without Echo Function
  BFD Bind Type          : TE_TUNNEL 
  Bind Session Type      : Static  
  Bind Peer IP Address   : 2.2.2.2         
  NextHop Ip Address     : -.-.-.-  
  Bind Interface         : Tunnel1                                           
  Tunnel ID              : 33  
  FSM Board Id           : 9                TOS-EXP                : 1
  Min Tx Interval (ms)   : 100              Min Rx Interval (ms)   : 100 
  Actual Tx Interval (ms): 100              Actual Rx Interval (ms): 100 
  Local Detect Multi     : 3                Detect Interval (ms)   : 300 
  Echo Passive           : Disable          Acl Number             : - 
  Destination Port       : 3784             TTL                    : 1 
  Proc Interface Status  : Disable          Process PST            : Disable    
  WTR Interval (ms)      : -                Config PST             : Disable    
  Active Multi           : 3   
  Last Local Diagnostic  : No Diagnostic
  Bind Application       : No Application Bind
  Session TX TmrID       : -                Session Detect TmrID   : - 
  Session Init TmrID     : -                Session WTR TmrID      : - 
  Session Echo Tx TmrID  : -   
  Session Description    : - 
--------------------------------------------------------------------------------

Total UP/DOWN Session Number : 2/0
```

**风险的恢复方案**

调整内层保护链路的BFD会话检测间隔小于外层链路的BFD会话的检测间隔。

### 1.6. 双向LSP场景下两端设备的BFD配置需对称

双向LSP场景下，BFD两端配置不对称，导致LDP LSP状态Down，LDP LSP承载的业务中断。

#### 1.6.1. 应用场景

如[图1-4](https://support.huawei.com/enterprise/zh/doc/EDOC1000124120?idPath=24030814|9856750|22715517|9858933|15837&section=j003#fig_dc_vrp_precaution_004901)所示，从RouterA到RouterB存在LSP，且从RouterB到RouterA也存在LSP。RouterA配置BFD For Peer IP，RouterB配置BFD For TE-LSP，同时RouterA存在与Peer IP地址相同的LDP LSP，当TE LSP故障后，LDP LSP承载的业务中断。

**图1-4** BFD两端配置不对称导致LDP LSP业务中断组网图
![img](.images/download)

#### 1.6.2. 配置规范

两端设备BFD检测类型配置保持一致，均为BFD For TE-LSP。

#### 1.6.3. 非规范配置的风险

**风险描述**

RouterB的TE LSP故障后，LDP LSP承载的业务中断。

业务现象如下：

TE LSP故障后，LDP LSP的BFD状态Down导致业务中断。

**风险的判断方法**

1. 查看两端的BFD配置是否对称。

   如下显示信息说明两端BFD配置不对称。RouterA配置的是BFD For Peer IP，RouterB配置的是BFD For TE-LSP。

   \# 在RouterA任意视图下执行**display current-configuration configuration bfd**命令。

   ```shell
   <HUAWEI> display current-configuration configuration bfd
   #
   bfd ieclsptowac bind peer-ip 100.1.1.1
    discriminator local 101
    discriminator remote 302
    min-tx-interval 50
    min-rx-interval 50
    commit
   #
   ```

   \# 在RouterB任意视图下执行**display current-configuration configuration bfd**命令。

   ```shell
   <HUAWEI> display current-configuration configuration bfd
   #
   bfd ieclsptowac bind mpls-te interface Tunnel0/0/1 te-lsp
   discriminator local 302
    discriminator remote 101
    min-tx-interval 50
    min-rx-interval 50
    commit
   #
   ```

2. 查询RouterA检测Peer IP的BFD会话的状态。

   \# 在RouterA的任意图下执行**display bfd session all**命令，发现BFD会话状态为Down。

   ```shell
   <HUAWEI> display bfd session all
   -----------------------------------------------------------------------
   Local Remote     PeerIpAddr      State     Type        InterfaceName
   -----------------------------------------------------------------------
   5     6          100.1.1.1       Down      S_IP_PEER         -
   -----------------------------------------------------------------------     
   Total UP/DOWN Session Number : 0/1
   ```

   \# 在RouterB的任意视图下执行**display mpls lsp** **include** *ip-address mask-len* **verbose**命令，发现LDP LSP的BFD状态为Down。

   ```shell
   <HUAWEI> display mpls lsp include 100.1.1.1 32 verbose 
   -------------------------------------------------------------------------------
                    LSP Information: LDP LSP
   -------------------------------------------------------------------------------
     No                  :  1
     VrfIndex            :        
     Fec                 :  100.1.1.1/32
     Nexthop             :  100.2.1.1
     In-Label            :  NULL
     Out-Label           :  153468
     In-Interface        :  ----------
     Out-Interface       :  GigabitEthernet1/0/0
     LspIndex            :  191839
     Token               :  0x2001476
     FrrToken            :  0x0
     LsrType             :  Ingress
     Outgoing token      :  0x0
     Label Operation     :  PUSH
     Mpls-Mtu            :  9000
     TimeStamp           :  144908sec
     Bfd-State           :  Down
   BGPKey              :  ------
   ```

**风险的恢复方案**

由于LDP LSP已经被BFD 置Down，需要在配置BFD For TE-LSP的一端配置BFD For Peer IP先让BFD协商UP，再删除BFD配置，LDP LSP的转发状态即可恢复。


