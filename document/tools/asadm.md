# Asadm

## 1 概述

​		Aerospike Admin是一个交互式python实用程序，主要用于获取集群当前运行状况的摘要信息，还用于跨集群执行动态配置和优化命令。

​		有关在集群中运行asadm所需的参数列表，请参阅`asadm--help`。关于该命令的一些配置可以查看 `/etc/aerospike/astools.conf` 文件

​		执行asadm后，您将收到一个“Admin>”提示，键入help并按return键以获取命令和说明列表。

## 2 Usage

### 2.1 asinfo

​		**列出当前集群节点的详细信息（包括版本、服务ip端口、配置、分区数量）。**

- asinfo:
  "asinfo" provides raw access to the info protocol.
  Options:
  -v <command>   - The command to execute
  -p <port>      - Port to use in case of XDR info command
  				 and XDR is not in asd
  -l             - Replace semicolons ";" with newlines.
  --no_node_name - Force to display output without printing node names.
  Modifiers: like, with
  Default: Executes an info command.

### 2.2 collectinfo

​		**“collect info”用于收集集群信息、aerospeck conf文件和系统统计信息。**

- collectinfo:
"collectinfo" is used to collect cluster info, aerospike conf file and system stats.
Modifiers: with
Default: Collects cluster info, aerospike conf file for local node and system stats from all nodes if remote server credentials provided.
If credentials are not available then it will collect system stats from local node only.
  Options:
	-n           <int>           - Number of snapshots. Default: 1
	-s           <int>           - Sleep time in seconds between each snapshot. Default: 5 sec
	--enable-ssh                 - Enable remote server system statistics collection.
	--ssh-user   <string>        - Default user id for remote servers. This is System user id (not Aerospike user id).
	--ssh-pwd    <string>        - Default password or passphrase for key for remote servers. This is System password (not Aerospike password).
	--ssh-port   <int>           - Default SSH port for remote servers. Default: 22
	--ssh-key    <string>        - Default SSH key (file path) for remote servers.
	--ssh-cf     <string>        - Remote System Credentials file path.
								   If server credentials are not available in credential file then default credentials will be used 
								   File format : each line should contain <IP[:PORT]>,<USER_ID>,<PASSWORD or PASSPHRASE>,<SSH_KEY>
								   Example:  1.2.3.4,uid,pwd
											 1.2.3.4:3232,uid,pwd
											 1.2.3.4:3232,uid,,key_path
											 1.2.3.4:3232,uid,passphrase,key_path
											 [2001::1234:10],uid,pwd
	
	[2001::1234:10]:3232,uid,,key_path
	--output-prefix <string>     - Output directory name prefix.
	--asconfig-file <string>     - Aerospike config file path to collect. Default: /etc/aerospike/aerospike.conf

### 2.3 exit

​		退出登陆

- exit:
  Terminate session

### 2.4 features

​		显示运行Aerospike群集时使用的功能。

- features:
  Displays features used in running Aerospike cluster.
  Modifiers: like, with

### 2.5 help

​		命令帮助手册

- help:
  Returns documentation related to a command
  for example, to retrieve documentation for the "info"
  command use "help info".

### 2.6 info

​		显示网络、命名空间和XDR摘要信息。

- info:
The "info" command provides summary tables for various aspects
of Aerospike functionality.
Modifiers: with
Default: Displays network, namespace, and XDR summary information.
  - dc:
	  Displays summary information for each datacenter.
  - namespace:
	The "namespace" command provides summary tables for various aspects
	of Aerospike namespaces.
	Modifiers: with
	Default: Displays usage and objects information for namespaces
	  - object:
		Displays object information for each namespace.
	  - usage:
		Displays usage information for each namespace.
  - network:
	  Displays network information for Aerospike.
  - set:
	  Displays summary information for each set.
  - sindex:
	  Displays summary information for Secondary Indexes (SIndex).
  - xdr:
	  Displays summary information for Cross Datacenter
	  Replication (XDR).

### 2.7 pager

- pager:
Set pager for output
  - off:
	  Removes pager and prints output normally
  - on:
	  Displays output with vertical and horizontal paging for each output table same as linux 'less' command.
	  Use arrow keys to scroll output and 'q' to end page for table.
	  All linux less commands can work in this pager option.
  - scroll:
	  Display output in scrolling mode

### 2.8 show

​		用于显示Aerospike的统计配置。

- show:
"show" is used to display Aerospike Statistics configuration.
  - config:
	"show config" is used to display Aerospike configuration settings
	Modifiers: diff, like, with
	Default: Displays service, network, and namespace configuration
	  Options:
		-r <int>     - Repeating output table title and row header after every r columns.
					   default: 0, no repetition.
		-flip        - Flip output table to show Nodes on Y axis and config on X axis.
	  - cluster:
		Displays Cluster configuration
	  - dc:
		Displays datacenter configuration
	  - namespace:
		Displays namespace configuration
	  - network:
		Displays network configuration
	  - service:
		Displays service configuration
	  - xdr:
		Displays XDR configuration
  - distribution:
	"distribution" is used to show the distribution of object sizes
	and time to live for node and a namespace.
	Modifiers: for, with
	Default: Shows the distributions of Time to Live and Object Size
	  - eviction:
		Shows the distribution of namespace Eviction TTLs for server version 3.7.5 and below
	  - object_size:
		Shows the distribution of Object sizes for namespaces
		  Options:
			-b               - Force to show byte wise distribution of Object Sizes.
							   Default is rblock wise distribution in percentage
			-k <buckets>     - Maximum number of buckets to show if -b is set.
							   It distributes objects in same size k buckets and 
							   display only buckets which has objects in it. Default is 5.
	  - time_to_live:
		Shows the distribution of TTLs for namespaces
  - latency:
	Modifiers: for, like, with
	Default: Displays latency information for Aerospike cluster.
	  Options:
		-f <int>     - Number of seconds (before now) to look back to.
					   default: Minimum to get last slice
		-d <int>     - Duration, the number of seconds from start to search.
					   default: everything to present
		-t <int>     - Interval in seconds to analyze.
					   default: 0, everything as one slice
		-m           - Set to display the output group by machine names.
  - mapping:
	"show mapping" is used to display Aerospike mapping from IP to Node_id and Node_id to IPs
	Modifiers: like
	Default: Displays mapping IPs to Node_id and Node_id to IPs
	  - ip:
		Displays IP to Node_id mapping
	  - node:
		Displays Node_id to IPs mapping
  - pmap:
	Displays partition map analysis of Aerospike cluster.
  - statistics:
	Displays statistics for Aerospike components.
	Modifiers: for, like, with
	Default: Displays bin, set, service, and namespace statistics
	  Options:
		-t           - Set to show total column at the end. It contains node wise sum for statistics.
		-r <int>     - Repeating output table title and row header after every r columns.
					   default: 0, no repetition.
		-flip        - Flip output table to show Nodes on Y axis and stats on X axis.
	  - bins:
		  Displays bin statistics
	  - dc:
		  Displays datacenter statistics
	  - namespace:
		  Displays namespace statistics
	  - service:
		  Displays service statistics
	  - sets:
		  Displays set statistics
	  - sindex:
		  Displays sindex statistics
	  - xdr:
		  Displays XDR statistics

### 2.9 summary

​		显示Aerospike群集的摘要。

- summary:
  Displays summary of Aerospike cluster.
  Options:
  -l                        - Enable to display namespace output in List view. Default: Table view
  --enable-ssh              - Enable remote server system statistics collection.
  --ssh-user   <string>     - Default user id for remote servers. This is System user id (not Aerospike user id).
  --ssh-pwd    <string>     - Default password or passphrase for key for remote servers. This is System password (not Aerospike password).
  --ssh-port   <int>        - Default SSH port for remote servers. Default: 22
  --ssh-key    <string>     - Default SSH key (file path) for remote servers.
  --ssh-cf     <string>     - Remote System Credentials file path.
  							If server credentials are not available in credential file then default credentials will be used 
  							File format : each line should contain <IP[:PORT]>,<USER_ID>,<PASSWORD or PASSPHRASE>,<SSH_KEY>
  							Example:  1.2.3.4,uid,pwd
  									  1.2.3.4:3232,uid,pwd
  									  1.2.3.4:3232,uid,,key_path
  									  1.2.3.4:3232,uid,passphrase,key_path
  									  [2001::1234:10],uid,pwd

  [2001::1234:10]:3232,uid,,key_path
  Modifiers: with

### 2.10 watch

​		为指定的暂停和迭代运行命令。

- watch:
"watch" Runs a command for a specified pause and iterations.
Usage: watch [pause] [iterations] [--no-diff] command]
   pause:      the duration between executions.
			   [default: 2 seconds]
   iterations: Number of iterations to execute command.
			   [default: until keyboard interrupt]
   --no-diff:  Do not do diff highlighting
Example 1: Show "info network" 3 times with 1 second pause
		   watch 1 3 info network
Example 2: Show "info namespace" with 5 seconds pause until
		   interrupted
		   watch 5 info namespace

