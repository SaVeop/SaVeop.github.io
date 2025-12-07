# Docker 存储空间不足处理办法

Docker 存储空间不足后，容易引起容器应用无法正常运行，例如 **RocketMQ 会拒绝写入消息**、**Docker 无法启动** 等情况。本笔记总结常见问题的排查方法与解决步骤。

------

# 一、查看空间占用情况

## 1. 查看磁盘与分区情况

```
lsblk
```

## 2. 查看文件系统的磁盘使用情况

```
df -h
```

如果发现：

- `/dev/mapper/centos-root` 占用 > 90%
   → **RocketMQ 可能拒绝写入消息**

## 3. 查看 Docker 空间占用

```
docker system df
```

如果发现：

- **Local Volumes 占用过高**
   → Docker 可能无法正常启动。

------

# 二、备份镜像

> 在清理缓存及日志前，**务必先备份**，防止数据丢失。

## 1. 关闭 docker 服务器

停止虚拟机或关闭 docker 服务。

## 2. 通过 VirtualBox 导出虚拟机镜像

路径：`管理 > 导出虚拟电脑`

## 3. 选择文件路径并导出

## 4. 重启虚拟机

虚拟机默认登录用户：`root`

## 5. 重启 Docker

若正常启动，跳到下一步；如果启动失败：

```
systemctl start docker
```

若依旧失败，则需手动删除部分日志：

```
cd /var/lib/docker/containers
rm *-json.log
systemctl start docker
```

## 6. 启动 docker-compose 服务

```
cd /docker
docker-compose up -d mysql redis nacos rmqbroker1 rmqnamesrv seata-server
```

------

# 三、清理缓存

## （一）清理 Docker 空间（推荐）

> 清理前务必保证所有 **docker-compose 应用均已启动**，否则未启动的应用可能被当作垃圾清理掉！

```
docker system prune -a --volumes
```

清理内容包括：

- 未使用镜像
- 未使用容器
- 未使用网络
- 未使用卷（volumes）

## （二）清理 overlay2（高风险，暂不使用）

手动清理 overlay2 可能导致数据丢失，需要完全停机并备份。**不建议使用**。

------

# 四、扩展磁盘空间（VirtualBox 虚拟机）

当清理方式无效时，需要扩容磁盘。

------

# 步骤 1：扩展虚拟硬盘

## 1. 必须先关闭虚拟机

## 2. 扩展 vdi 虚拟硬盘

列出磁盘：

```
.\VBoxManage list hdds
```

扩容 VDI：

```
.\VBoxManage modifyhd "D:\Vms\centos-7-20\centos-7-20-disk001.vdi" --resize 100000
```

再次查看确认大小已变：

```
.\VBoxManage list hdds
```

## 3. 重启虚拟机

------

# 步骤 2：查看磁盘信息

## 1. 查看系统磁盘

```
lsblk
```

## 2. 查看分区信息

```
fdisk -l /dev/sda
```

------

# 步骤 3：使用 fdisk 扩展分区（不丢数据）

> 因为 `/dev/sda2` 是 LVM PV，删除分区仅删除分区表，不会删除数据。

## 1. 进入 fdisk

```
fdisk /dev/sda
```

## 2. 删除原分区（这里只删除分区表）

- 输入 `p` 查看
- 输入 `d` 删除分区 `2`

## 3. 重新创建分区

- 输入 `n` 创建新的分区 2
- 起始扇区必须与旧分区一致（fdisk -l 中查到）
- 结束扇区按 **Enter** 使用默认值（最大空间）

## 4. 保存分区表

```
w
```

## 5. 若提示 “设备或资源忙”

执行：

```
partprobe /dev/sda
```

再次查看：

```
fdisk /dev/sda
```

------

# 步骤 4：扩展物理卷 PV

## 1. 查看 PV

```
pvdisplay /dev/sda2
```

## 2. 扩展 PV

```
pvresize /dev/sda2
```

## 3. 确认

```
pvdisplay
```

------

# 步骤 5：扩展逻辑卷 LV

## 1. 查看 LV

```
lvdisplay
```

## 2. 使用全部剩余空间扩展 LV

```
lvextend -l +100%FREE /dev/centos/root
```

## 3. 再次查看

```
lvdisplay
```

------

# 步骤 6：扩展文件系统

## 1. 查看文件系统类型

```
df -T
```

若 `/dev/centos/root` 为 xfs：

## 2. 扩展 XFS 文件系统

```
xfs_growfs /dev/centos/root
```

## 3. 验证

```
df -h
```

扩容成功后，磁盘空间将从约 17G → 79G（或你指定的容量）。