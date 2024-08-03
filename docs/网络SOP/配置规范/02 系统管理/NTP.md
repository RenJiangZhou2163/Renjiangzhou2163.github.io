# 系统管理

## 1. NTP

### 1.1. NTP需要配置preference参数设置优先选择的服务器

通过 **ntp-service unicast-server** *server-ip* 命令指定多个不同的NTP服务器时，要为其中一个NTP服务器增加preference参数，将其设置为优选服务器，可以避免本设备在不同的NTP服务器之间反复震荡切换。

#### 1.1.1. 应用场景

在系统视图下，执行 **ntp-service unicast-server** *server-ip* 命令指定多个不同的NTP远端服务器。

#### 1.1.2. 配置规范

在系统视图下执行 **ntp-service unicast-server** *server-ip* **preference** 命令，将其中一个服务器设置为优选服务器。

#### 1.1.3. 非规范配置的风险

**风险描述**

当在远端服务器上执行 **ntp-service unicast-server** *server-ip* 命令，未配置 **preference** 参数时，NTP客户端会在多个远端服务器之间反复切换，导致NTP客户端时间频繁变化，并记录大量日志。

**风险的判断方法**

在用户视图下，执行 **display current-configuration** **configuration** *ntp* 命令查询NTP客户端的配置。

从显示信息可以看出，指定的远端服务器个数超过1个并且都没有配置 **preference** 参数。

```shell
<HUAWEI> display current-configuration configuration ntp
#
ntp-service unicast-server 10.1.1.1
ntp-service unicast-server 10.1.1.2
#
```

**风险的恢复方案**

请按照配置规范进行配置。

### 1.2. NTP MD5/SHA56认证配置规范

NTP设置MD5/SHA56认证方式并且配置 **ntp-service authentication enable** 命令后必须同时配置多条命令，否则无法和NTP服务器进行时钟同步。

#### 1.2.1. 应用场景

设备作为NTP客户端，在系统视图下配置了 **ntp-service authentication enable** 命令，使能NTP验证功能。

#### 1.2.2. 配置规范

在系统视图下，如下命令必须同时配置，才能保证NTP客户端和NTP服务器进行时钟同步。

```shell
<HUAWEI> system-view
[HUAWEI] ntp-service authentication enable 
[HUAWEI] ntp-service reliable authentication-keyid 169 
[HUAWEI] ntp-service unicast-server 172.0.0.1 authentication-keyid 169
[HUAWEI] ntp-service authentication-keyid 169 authentication-mode md5 cipher Root@123
```

#### 1.2.3. 非规范配置的风险

**风险描述**

缺少以下一条或多条配置时，NTP客户端无法和服务器进行时钟同步。

- **ntp-service authentication-keyid** *key-id* **authentication-mode** *mode* **cipher** *password*
- **ntp-service reliable authentication-keyid** *key-id* *key-id*
- **ntp-service unicast-server** *server-ip* **authentication-keyid** *key-id*（该命令仅客户端涉及，服务器端不涉及）

**风险的判断方法**

在用户视图下，执行 **display current-configuration** **configuration** *ntp* 命令查询NTP客户端的配置。

从显示信息可以看出，设备启用了NTP身份验证功能，缺少密钥的配置。

```shell
<HUAWEI> display current-configuration configuration ntp
#
ntp-service authentication enable
#
```

**风险的恢复方案**

请按照配置规范进行配置。


