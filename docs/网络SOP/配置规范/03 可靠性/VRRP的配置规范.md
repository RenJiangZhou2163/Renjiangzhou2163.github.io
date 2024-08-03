# 可靠性

## 1. VRRP的配置规范

### 1.1. VRRP中间链路需要配置Eth-Trunk保护的配置规范

部署VRRP时，如果接入设备的上行网关的端口采用了端口间的备份，即无故障的情况下只有一个端口转发流量，在转发流量的端口发生故障之后备份端口开始转发流量。此时，如果VRRP的中间心跳丢失则会导致VRRP出现双主的情况，此时很有可能导致接入设备上的流量中断。

#### 1.1.1. 应用场景

如[图1-5](https://support.huawei.com/enterprise/zh/doc/EDOC1000124120?idPath=24030814|9856750|22715517|9858933|15837&section=j003#fig_dc_vrp_precaution_006901)所示，UMG上行网关的端口GE1/0/0和GE1/0/1采用的端口间的备份，只有一个端口转发功能被开启。在DeviceA和DeviceB上部署VRRP功能，同时在VRRP中间心跳链路配置Eth-Trunk保护。

**图1-5** VRRP中间链路配置Eth-Trunk保护组网图
![img](../.images/download)

#### 1.1.2. 配置规范

在DeviceA和DeviceB上配置VRRP备份组。在DeviceA上配置较高优先级，作为Master设备承担流量；在DeviceB上配置较低优先级，作为备用路由器，实现冗余备份。

在DeviceA和DeviceB上均需配置Eth-Trunk接口，并将以太网物理接口加入Eth-Trunk接口，同时添加跨板成员口，确保一个单板故障之后Eth-Trunk中仍有状态是Up的链路。

#### 1.1.3. 非规范配置的风险

**风险描述**

当Eth-Trunk的成员口所在的心跳链路故障之后，同时满足下列条件时，UMG的业务大概率中断：

- UMG的上行网关的端口只支持端口间的备份，即一般只能有一个端口Up；
- DeviceA和DeviceB的VRRP备份组间，心跳链路所在的端口未使用跨板Trunk方式。

**风险的判断方法**

![img](../.images/download) **说明：**

需要在DeviceA和DeviceB两台设备上都要执行此命令检查是否存在跨板成员口。

1. 查看对应Eth-Trunk的成员口是否是跨板成员口。

   ```shell
   <HUAWEI> display interface eth-trunk 1
   Eth-Trunk1 current state : UP
   Line protocol current state : UP
   Link quality grade : GOOD
   Description:HUAWEI, Eth-Trunk1 Interface
   Switch Port, TPID : 8100(Hex), Hash arithmetic : According to flow,Maximal BW: 2G, Current BW: 1G, The Maximum Transmit Unit is 1500
   IP Sending Frames' Format is PKTFMT_ETHNT_2, Hardware address is 0018-82d9-e71b
   Physical is ETH_TRUNK
   Current system time: 2017-04-11 12:15:41
       Last 300 seconds input rate 0 bits/sec, 0 packets/sec
       Last 300 seconds output rate 0 bits/sec, 0 packets/sec
       Realtime 2 seconds input rate 0 bits/sec, 0 packets/sec
       Realtime 2 seconds output rate 0 bits/sec, 0 packets/sec
       Input: 3 packets,939 bytes
              0 unicast,0 broadcast,3 multicast
              0 errors,0 drops
       Output:3 packets,917 bytes
              0 unicast,0 broadcast,3 multicast
              0 errors,0 drops
       Input bandwidth utilization  :    0%
       Output bandwidth utilization :    0%
   -----------------------------------------------------
   PortName                      Status      Weight
   -----------------------------------------------------
   GigabitEthernet1/1/8          DOWN        1
   GigabitEthernet3/1/0          UP          1
   -----------------------------------------------------
   The Number of Ports in Trunk : 2
   The Number of UP Ports in Trunk : 1   
   ```

2. 如果确定Eth-Trunk存在多个成员口并且成员口来自不同单板，比如GigabitEthernet1/1/8和GigabitEthernet3/1/0，前者是1号单板端口，后者是3号单板端口，则此时不存在该风险；否则如果成员口都来自同一个单板，则存在风险。

**风险的恢复方案**

在DeviceA和DeviceB上均需配置Eth-Trunk接口，并将以太网物理接口加入Eth-Trunk接口，同时添加跨板成员口，确保一个单板故障之后Eth-Trunk中仍有状态是Up的链路。

### 1.2. TE Tunnel没有配置误码倒换导致业务受损

#### 1.2.1. 应用场景

如下图所示，在IPRAN场景中，ASG和RSG之间部署TE hot-standby。

![img](../.images/download)

#### 1.2.2. 配置规范

在ASG的Tunnel口上，通过命令行 **mpls te bit-error-detection** 配置RSVP-TE隧道的误码倒换功能。

#### 1.2.3. 非规范配置的风险

**风险描述**

当未配置误码倒换功能时，由于ASG和RSG之间的传输设备存在误码，会导致ASG下挂的基站产生中断。

**风险的判断方法**

查看ASG上的Tunnel配置，看是否已经配置了误码倒换。

**风险的恢复方案**

同配置规范。