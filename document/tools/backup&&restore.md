# Backup and Restore

​		Aerospike提供备份和还原群集数据的能力。在正常情况下，数据复制（集群内）和跨数据中心复制（XDR）可确保即使在发生硬件故障或网络中断时也不会丢失数据。但是，最好定期创建备份，以便于从灾难性数据中心故障或管理事故中恢复。

## 1 Asbackup

### 1.1 概述

​		使用Aerospike的标准客户端，扫描ns（命名空间）或ns中的特定set（集合）备份所有数据。

​		asbackup实用程序用于将命名空间或集从Aerospike群集备份到本地存储，asbackup同时查询群集中的多个节点。默认情况下，最多可同时备份10个节点。可以使用--parallel命令行选项更改此限制。每个群集节点的数据都是按分区备份的。如果记录是在备份其分区后创建或更新的，则备份不会反映这一点。

​		当群集正在迁移时（即，由于节点故障而重新平衡），所执行的任何备份都可能不完全一致。同一记录可能有多个副本或丢失的记录。使用--no cluster change选项使asbackup中止，以防它检测到挂起的迁移。

​		当使用--directory选项运行时，asbackup会在给定目录中创建多个.asb备份文件。备份由所有创建的文件组成。或者，-output file使asbackup将完整备份存储在给定的单个文件中。如果-指定为文件，则as backup将备份写入stdout。

### 1.2 Usage

```bash
asbackup --usage     #显示命令用法
asbackup --z         #显示命令用法
```

```bash
asbackup --host 1.2.3.4 --namespace test --directory backup_2015_08_24
```

​		运行as backup的最基本形式是指定要备份的集群（--host）、要备份的命名空间（--namespace）以及备份文件的本地目录（--directory）。假设我们有一个包含IP地址为1.2.3.4的节点的集群。以上命令表示，将1.2.3.4集群上的test空间中的所有数据，备份到backup_2015_08_24目录下。

### 1.3 Estimating the Backup Size（预估大小）



```bash
asbackup --host 192.168.3.111 --namespace bsfit --estimate
```

​		该命令可以预估命名空间数据的大致大小。

### 1.4 Incremental Backup（增量备份）

​		从服务器版本3.12开始，可以指定时间戳，以便仅备份自时间戳X以来更新的记录。可以建立一个操作例程来执行增量每日备份。请参阅“数据选项”部分中的--modified after选项。

```bash
-a, --modified-after
<YYYY-MM-DD_HH:MM:SS>
Perform an incremental backup; only include records
that changed after the given date and time. The system's
local timezone applies. If only HH:MM:SS is specified, then
today's date is assumed as the date. If only YYYY-MM-DD is
specified, then 00:00:00 (midnight) is assumed as the time.
```

```bash
eg:
asbackup --host 192.168.3.111 --namespace bsfit --directory backup_2020_1_7 --modified-after 2020-01-07
```

​		表示备份2020-01-07 00:00:00之后的数据。当然类似的还有：

​		-b, --modified-before <YYYY-MM-DD_HH:MM:SS>  表示该时间之前的数据。

### 1.5 Connection Options（连接选项）

| Option               | Default   | Description                |
| :------------------- | :-------- | :------------------------- |
| -h                   | 127.0.0.1 | 充当群集入口点的主机。     |
| -p                   | 3000      | 端口                       |
| -U                   | -         | 具有读取权限的用户名。     |
| -P                   | -         | 密码                       |
| -l                   | All nodes | 用于指定需要备份的群集节点 |
| -w                   | 10        | 并行备份的最大节点数。     |
| --tls-enable         | disabled  | 指示应使用TLS连接。        |
| --services-alternate | false     |                            |
|                      |           |                            |

```bash
asbackup --node-list 1.2.3.4:3000,5.6.7.8:3000 --namespace test --directory backup_2015_08_24
```

​		以上命令指定了需要备份集群中1.2.3.4、5.6.7.8两个节点的test命名空间的数据到backup_2015_08_24目录下。

### 1.6 备份文件格式分析

`["+"] [SP] ["n"] [SP] [escape({namespace})] [LF]`
`["+"] [SP] ["d"] [SP] [{digest}] [LF]`
`["+"] [SP] ["s"] [SP] [escape({set})] [LF]`
`["+"] [SP] ["g"] [SP] [{gen-count}] [LF]`
`["+"] [SP] ["t"] [SP] [{expiration}] [LF]`
`["+"] [SP] ["b"] [SP] [{bin-count}] [LF]`
`["-"] [SP] ["N"] [SP] [escape({bin-name})]`
`["-"] [SP] ["I"] [SP] [escape({bin-name})] [SP] [{int-value}] [LF]`
`["-"] [SP] ["D"] [SP] [escape({bin-name})] [SP] [{float-value}] [LF]`
`["-"] [SP] ["S"] [SP] [escape({bin-name})] [SP] [{string-length}] [SP] [{string-data}] [LF]`
`["-"] [SP] ["B"] ["!"]? [SP] [escape({bin-name})] [SP] [{bytes-length}] [SP] [{bytes-data}] [LF]`

## 2 AsRestore

### 2.1 概述

​		使用Aerospike的标准客户端，从asbackup创建的备份还原ns或set数据。	

​		asrestore实用程序还原使用asbackup创建的备份。如果集群上的命名空间已经包含现有记录，则可配置的写入策略决定哪些记录优先于命名空间中的记录或来自备份的记录。

