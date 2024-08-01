# Azure上的Red Hat Enterprise Linux映像

## 切换到 RHEL7 上的非 EUS 存储库

若要删除版本锁，请使用以下命令。 以 `root` 的身份运行命令。

1. 删除 `releasever` 文件。

   ```bash
   sudo rm /etc/yum/vars/releasever
   ```

2. 禁用 EUS 存储库。

   ```bash
   sudo yum --disablerepo='*' remove 'rhui-azure-rhel7-eus'
   ```

3. 添加非 EUS 存储库。

   ```bash
   sudo yum --config=https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel7.config install rhui-azure-rhel7
   ```

4. 更新 RHEL VM。

   ```bash
   sudo yum update
   ```

## 切换到 RHEL8 上的非 EUS 存储库

若要删除版本锁，请使用以下命令。 以 `root` 的身份运行命令。

1. 删除 `releasever` 文件。

   ```bash
   sudo rm /etc/dnf/vars/releasever
   ```

2. 禁用 EUS 存储库。

   ```bash
   sudo dnf --disablerepo='*' remove 'rhui-azure-rhel8-eus'
   ```

3. 添加非 EUS 存储库。

   ```bash
   wget https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel8.config
   sudo dnf --config=rhui-microsoft-azure-rhel8.config install rhui-azure-rhel8
   ```

4. 更新 RHEL VM。

   ```bash
   sudo dnf update
   ```

## 切换到 RHEL9 上的非 EUS 存储库

若要删除版本锁，请使用以下命令。 以 `root` 的身份运行命令。

1. 删除 `releasever` 文件。

   ```bash
   sudo rm /etc/dnf/vars/releasever
   ```

2. 禁用 EUS 存储库。

   ```bash
   sudo dnf --disablerepo='*' remove 'rhui-azure-rhel9-eus'
   ```

3. 添加非 EUS 存储库。

   ```bash
   sudo dnf --config='https://rhelimage.blob.core.windows.net/repositories/rhui-microsoft-azure-rhel9.config' install rhui-azure-rhel9
   ```

4. 更新 RHEL VM。

   ```bash
   sudo dnf update
   ```

## 参考文档

| 标题 | 文档链接                                                     | 更新时间   |
| ---- | ------------------------------------------------------------ | ---------- |
|      | https://learn.microsoft.com/zh-cn/azure/virtual-machines/workloads/redhat/redhat-rhui?tabs=rhel7 | 2023-12-06 |
