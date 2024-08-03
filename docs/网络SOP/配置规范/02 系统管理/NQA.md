# 系统管理

## 1. NQA

### 1.1. NQA探测周期配置规范

NQA测试例周期配置过短，在测试例一轮探测尚未执行结束，下一轮就开始了，导致测试结果为no result，影响其他联动协议，存在其他协议联动NQA失效的风险。

#### 1.1.1. 应用场景

NQA探测。

#### 1.1.2. 配置规范

1. 在NQA视图下，执行 **stop** 命令，停止测试例执行。
2. 在NQA视图下，执行 **frequency** *interval* 命令，配置NQA探测周期。需确保：探测周期>（探测次数-1）* 探测间隔 + 探测超时时间。
3. 在NQA视图下，执行 **start** **now** 命令，开始执行测试例。

#### 1.1.3. 非规范配置的风险

**风险描述**

在NQA视图下，执行 **frequency** *interval* 命令配置NQA测试例的探测周期，如果配置的探测周期<（探测次数-1）*探测间隔+探测超时时间，则发生问题时单板重启，NQA联动的其他业务，如路由协议、VRRP等，会出现联动功能失效，从而影响到业务转发。

业务现象如下：

NQA测试结果为no result。

**风险的判断方法**

用户视图下，执行 **display current-configuration** **configuration** *nqa* 命令查看NQA测试例配置。

从显示信息可以看出，NQA测试例的探测周期为20秒，探测次数为5，探测间隔为6秒，探测超时时间为4秒。探测周期< （探测次数-1）* 探测间隔 + 探测超时时间。因此该测试例探测结果为“no result”。

```shell
<HUAWEI> display current-configuration configuration nqa
#
nqa test-instance 1 1
 test-type icmp
 destination-address ipv4 127.0.0.1
 frequency 20
 interval seconds 6
 timeout 4
 probe-count 5
 start now
#
```

**风险的恢复方案**

请按照配置规范进行配置。