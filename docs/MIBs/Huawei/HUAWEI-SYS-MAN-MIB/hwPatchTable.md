# hwPatchTable详细描述

该表用来配置补丁操作和查询。

该表的索引是hwSlotIndex, hwPatchIndex。

| OID                                   | 节点名称                | 数据类型                                                                           | 最大访问权限   | 含义                                                                                                                                                                                                                         | 实现规格                |
| ------------------------------------- | ----------------------- | ---------------------------------------------------------------------------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.1  | hwPatchSlotIndex        | Integer32                                                                          | not-accessible | 插槽索引。                                                                                                                                                                                                                   | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.2  | hwPatchIndex            | Unsigned32                                                                         | not-accessible | 补丁索引。                                                                                                                                                                                                                   | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.3  | hwPatchUsedFileName     | DisplayString                                                                      | read-only      | 补丁文件名字。                                                                                                                                                                                                               | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.4  | hwPatchVersion          | DisplayString                                                                      | read-only      | 补丁文件版本。                                                                                                                                                                                                               | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.5  | hwPatchDescription      | DisplayString                                                                      | read-only      | 补丁描述。                                                                                                                                                                                                                   | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.6  | hwPatchProgramVersion   | DisplayString                                                                      | read-only      | 主机软件的版本号。                                                                                                                                                                                                           | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.7  | hwPatchFuncNum          | Integer32                                                                          | read-only      | 补丁包含的功能个数。                                                                                                                                                                                                         | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.8  | hwPatchTextLen          | Integer32                                                                          | read-only      | 补丁编码长度。                                                                                                                                                                                                               | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.9  | hwPatchDataLen          | Integer32                                                                          | read-only      | 补丁文件数据长度。                                                                                                                                                                                                           | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.10 | hwPatchType             | INTEGER { hotCommon(1), hotTemporary(2), coolCommon(3) ,coolTemporary(4)}          | read-only      | 补丁类型。hotCommon(1): 热补丁。hotTemporary(2): 临时热补丁。coolCommon(3): 冷补丁。coolTemporary(4): 临时冷补丁。                                                                                                           | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.11 | hwPatchBuildTime        | DateAndTime                                                                        | read-only      | 补丁创建时间。                                                                                                                                                                                                               | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.12 | hwPatchActiveTime       | DateAndTime                                                                        | read-only      | 补丁生效时间。                                                                                                                                                                                                               | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.13 | hwPatchAdminStatus      | INTEGER { run (1), active(2), deactive (3), delete (4)}                            | read-write     | 补丁操作状态：run (1)表示将补丁状态变为运行（run）状态；active(2) 表示将补丁状态变为激活（active）状态；deactive (3)表示将补丁状态变为去激活（deactive）状态；delete (4) 表示将补丁删除。                                    | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.14 | hwPatchOperateState     | INTEGER { patchRunning(1), patchActive(2), patchDeactive (3), patchDeleting（4） } | read-only      | 补丁运行状态：patchRunning(1)表示当前补丁运行状态为运行态（running）；patchActive(2)表示当前补丁运行状态为激活态（active）；patchDeactive(3)表示当前补丁运行状态为未激活态（deactive）；patchDeleting(4)表示当前补丁被删除。 | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.15 | hwPatchOperateDestType  | INTEGER { all(1), slave(2),slot(3),chassis(4),unused(5)}                           | read-create    | 补丁操作类型。                                                                                                                                                                                                               | 实现与MIB文件定义一致。 |
| 1.3.6.1.4.1.2011.5.25.19.1.8.5.1.1.16 | hwPatchOperateDestIndex | DisplayString                                                                      | read-create    | 补丁操作索引。                                                                                                                                                                                                               | 实现与MIB文件定义一致。 |

#### 创建约束

该表不支持创建操作。

#### 修改约束

无

#### 删除约束

该表不支持删除操作。

#### 读取约束

无