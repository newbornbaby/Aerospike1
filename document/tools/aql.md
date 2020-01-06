# AQL

​		aql工具为数据库、UDF和索引管理提供了类似SQL的命令行接口。

## 1 Usage

​		在运行aql--help时，会提供aql的使用选项。下面是用法的摘要。

```bash
Usage: aql [OPTIONS]
------------------------------------------------------------------------------
 -V, --version        Print AQL version information.
 -O, --options        Print command-line options message.
 -E, --help           Print command-line options message and AQL commands
                      documentation.
 -c, --command=cmd    Execute the specified command.
 -f, --file=path      Execute the commands in the specified file.
 -e, --echo           Enable echoing of commands. Default: disabled
 -v, --verbose        Enable verbose output. Default: disabled

Configuration File Allowed Options
----------------------------------

[cluster]
 -h, --host=HOST
                      HOST is "<host1>[:<tlsname1>][:<port1>],..."
                      Server seed hostnames or IP addresses. The tlsname is
                      only used when connecting with a secure TLS enabled
                      server. Default: localhost:3000
                      Examples:
                        host1
                        host1:3000,host2:3000
                        192.168.1.10:cert1:3000,192.168.1.20:cert2:3000
 --services-alternate
                      Use to connect to alternate access address when the
                      cluster's nodes publish IP addresses through access-address
                      which are not accessible over WAN and alternate IP addresses
                      accessible over WAN through alternate-access-address. Default: false.
 -p, --port=PORT Server default port. Default: 3000
 -U, --user=USER User name used to authenticate with cluster. Default: none
 -P, --password
                      Password used to authenticate with cluster. Default: none
                      User will be prompted on command line if -P specified and no
                       password is given.
 --auth
                      Set authentication mode when user/password is defined. Modes are
                      (INTERNAL, EXTERNAL, EXTERNAL_INSECURE) and the default is INTERNAL.
                      This mode must be set EXTERNAL when using LDAP
 --tls-enable         Enable TLS on connections. By default TLS is disabled.
 --tls-cafile=TLS_CAFILE
                      Path to a trusted CA certificate file.
 --tls-capath=TLS_CAPATH.
                      Path to a directory of trusted CA certificates.
 --tls-protocols=TLS_PROTOCOLS
                      Set the TLS protocol selection criteria. This format
                      is the same as Apache's SSLProtocol documented at http
                      s://httpd.apache.org/docs/current/mod/mod_ssl.html#ssl
                      protocol . If not specified the asadm will use ' -all
                      +TLSv1.2' if has support for TLSv1.2,otherwise it will
                      be ' -all +TLSv1'.
 --tls-cipher-suite=TLS_CIPHER_SUITE
                     Set the TLS cipher selection criteria. The format is
                     the same as Open_sSL's Cipher List Format documented
                     at https://www.openssl.org/docs/man1.0.1/apps/ciphers.
                     html
 --tls-keyfile=TLS_KEYFILE
                      Path to the key for mutual authentication (if
                      Aerospike Cluster is supporting it).
 --tls-keyfile-password=TLS_KEYFILE_PASSWORD
                      Password to load protected tls-keyfile.
                      It can be one of the following:
                      1) Environment varaible: 'env:<VAR>'
                      2) File: 'file:<PATH>'
                      3) String: 'PASSWORD'
                      Default: none
                      User will be prompted on command line if --tls-keyfile-password
                      specified and no password is given.
 --tls-certfile=TLS_CERTFILE <path>
                      Path to the chain file for mutual authentication (if
                      Aerospike Cluster is supporting it).
 --tls-cert-blacklist <path>
                      Path to a certificate blacklist file. The file should
                      contain one line for each blacklisted certificate.
                      Each line starts with the certificate serial number
                      expressed in hex. Each entry may optionally specify
                      the issuer name of the certificate (serial numbers are
                      only required to be unique per issuer).Example:
                      867EC87482B2
                      /C=US/ST=CA/O=Acme/OU=Engineering/CN=TestChainCA
 --tls-crl-check      Enable CRL checking for leaf certificate. An error
                      occurs if a valid CRL files cannot be found in
                      tls_capath.
 --tls-crl-checkall   Enable CRL checking for entire certificate chain. An
                      error occurs if a valid CRL files cannot be found in
                      tls_capath.
[aql]
 -z, --threadpoolsize=count
                      Set the number of client threads used to talk to the
                      server. Default: 16
 -o, --outputmode=mode
                      Set the output mode. (json | table | raw | mute)
                      Default: table
 -n, --outputtypes    Disable outputting types for values (e.g., GeoJSON, JSON)
                      to distinguish them from generic strings
 -T, --timeout=ms     Set the timeout (ms) for commands. Default: 1000
 -u, --udfuser=path   Path to User managed UDF modules.
                      Default: /opt/aerospike/usr/udf/lua


Default configuration files are read from the following files in the given order:
/etc/aerospike/astools.conf ~/.aerospike/astools.conf
The following sections are read: (cluster aql include)
The following options effect configuration file behavior
 --no-config-file
                      Do not read any config file. Default: disabled
 --instance=name
                      Section with these instance is read. e.g in case instance `a` is specified
                      sections cluster_a, aql_a is read.
 --config-file=path
                      Read this file after default configuration file.
 --only-config-file=path
                      Read only this configuration file.
```

## 2 Commands

```bash
COMMANDS
  DDL
      CREATE INDEX <index> ON <ns>[.<set>] (<bin>) NUMERIC|STRING|GEO2DSPHERE
      CREATE LIST/MAPKEYS/MAPVALUES INDEX <index> ON <ns>[.<set>] (<bin>) NUMERIC|STRING|GEO2DSPHERE
      DROP INDEX <ns>[.<set>] <index>
      Examples:

          CREATE INDEX idx_foo ON test.demo (foo) NUMERIC
          DROP INDEX test.demo idx_foo

  MANAGE UDFS
      REGISTER MODULE '<filepath>'
      REMOVE MODULE <filename>

          <filepath> is file path to the UDF module(in single quotes).
          <filename> is file name of the UDF module.

      Examples:

          REGISTER MODULE '~/test.lua' 
          REMOVE MODULE test.lua

  USER ADMINISTRATION
      CREATE USER <user> PASSWORD <password> ROLE[S] <role1>,<role2>...
          pre-defined roles: read|read-write|read-write-udf|sys-admin|user-admin
      DROP USER <user>
      SET PASSWORD <password> [FOR <user>]
      GRANT ROLE[S] <role1>,<role2>... TO <user>
      REVOKE ROLE[S] <role1>,<role2>... FROM <user>
      CREATE ROLE <role> PRIVILEGE[S] <priv1[.ns1[.set1]]>,<priv2[.ns2[.set2]]>... WHITELIST addr1,addr2...
          priv: read|read-write|read-write-udf|sys-admin|user-admin|data-admin
          ns:   namespace.  Applies to all namespaces if not set.
          set:  set name.  Applies to all sets within namespace if not set.
                sys-admin, user-admin and data-admin can't be qualified with namespace or set.
          addr: IP address.
      DROP ROLE <role>
      GRANT PRIVILEGE[S] <priv1[.ns1[.set1]]>,<priv2[.ns2[.set2]]>... TO <role>
      REVOKE PRIVILEGE[S] <priv1[.ns1[.set1]]>,<priv2[.ns2[.set2]]>... FROM <role>
      SET WHITELIST addr1,addr2... FOR <role>


  DML
      INSERT INTO <ns>[.<set>] (PK, <bins>) VALUES (<key>, <values>)
      DELETE FROM <ns>[.<set>] WHERE PK = <key>
      TRUNCATE <ns>[.<set>] [upto <LUT>] 

          <ns> is the namespace for the record.
          <set> is the set name for the record.
          <key> is the record's primary key.
          <bins> is a comma-separated list of bin names.
          <values> is comma-separated list of bin values, which may include type cast expressions. Set to NULL (case insensitive & w/o quotes) to delete the bin.
          <LUT> is last update time upto which set or namespace needs to be truncated. LUT is either nanosecond since Unix epoch like 1513687224599000000 or in date string in format like "Dec 19 2017 12:40:00".

        Type Cast Expression Formats:

            CAST(<Value> AS <TypeName>)
            <TypeName>(<Value>)

        Supported AQL Types:

              Bin Value Type                    Equivalent Type Name(s)
           ===============================================================
            Integer                           DECIMAL, INT, NUMERIC
            Floating Point                    FLOAT, REAL
            Aerospike CDT (List, Map, etc.)   JSON
            Aerospike List                    LIST
            Aerospike Map                     MAP
            GeoJSON                           GEOJSON
            String                            CHAR, STRING, TEXT, VARCHAR
           ===============================================================

        [Note:  Type names and keywords are case insensitive.]

      Examples:

          INSERT INTO test.demo (PK, foo, bar) VALUES ('key1', 123, 'abc')
          INSERT INTO test.demo (PK, foo, bar) VALUES ('key1', CAST('123' AS INT), JSON('{"a": 1.2, "b": [1, 2, 3]}'))
          INSERT INTO test.demo (PK, foo, bar) VALUES ('key1', LIST('[1, 2, 3]'), MAP('{"a": 1, "b": 2}'))
          INSERT INTO test.demo (PK, gj) VALUES ('key1', GEOJSON('{"type": "Point", "coordinates": [123.4, -456.7]}'))
          DELETE FROM test.demo WHERE PK = 'key1'

  INVOKING UDFS
      EXECUTE <module>.<function>(<args>) ON <ns>[.<set>]
      EXECUTE <module>.<function>(<args>) ON <ns>[.<set>] WHERE PK = <key>
      EXECUTE <module>.<function>(<args>) ON <ns>[.<set>] WHERE <bin> = <value>
      EXECUTE <module>.<function>(<args>) ON <ns>[.<set>] WHERE <bin> BETWEEN <lower> AND <upper>

          <module> is UDF module containing the function to invoke.
          <function> is UDF to invoke.
          <args> is a comma-separated list of argument values for the UDF.
          <ns> is the namespace for the records to be queried.
          <set> is the set name for the record to be queried.
          <key> is the record's primary key.
          <bin> is the name of a bin.
          <value> is the value of a bin.
          <lower> is the lower bound for a numeric range query.
          <upper> is the lower bound for a numeric range query.

      Examples:

          EXECUTE myudfs.udf1(2) ON test.demo
          EXECUTE myudfs.udf1(2) ON test.demo WHERE PK = 'key1'

  OPERATE
      OPERATE <op(<bin>, params...)>[with_policy(<map policy>),] [<op(<bin>, params...)> with_policy (<map policy>) ...] ON <ns>[.<set>] where PK=<key>

          <op> name of operation to perform.
          <bin> is the name of a bin.
          <params> parameters for operation.
          <map policy> map operation policy.
          <ns> is the namespace for the records to be queried.
          <set> is the set name for the record to be queried.
          <key> is the record's primary key.

      OP
          LIST_APPEND (<bin>, <val>)         
          LIST_INSERT (<bin>, <index>, <val>)
          LIST_SET    (<bin>, <index>, <val>)
          LIST_GET    (<bin>, <index>)       
          LIST_POP    (<bin>, <index>)       
          LIST_REMOVE (<bin>, <index>)       
          LIST_APPEND_ITEMS (<bin>, <list of vals>)         
          LIST_INSERT_ITEMS (<bin>, <index>, <list of vals>)
          LIST_GET_RANGE    (<bin>, <startindex>[, <count>])
          LIST_POP_RANGE    (<bin>, <startindex>[, <count>])
          LIST_REMOVE_RANGE (<bin>, <startindex>[, <count>])
          LIST_TRIM         (<bin>, <startindex>[, <count>])
          LIST_INCREMENT    (<bin>, <index>, <numeric val>) 
          LIST_CLEAR        (<bin>) 
          LIST_SIZE         (<bin>) 
          MAP_PUT             (<bin>, <key>, <val>) [with_policy (<map policy>)]
          MAP_PUT_ITEMS       (<bin>, <map>)  [with_policy (<map policy>)]
          MAP_INCREMENT       (<bin>, <key>, <numeric val>) [with_policy (<map policy>)]
          MAP_DECREMENT       (<bin>, <key>, <numeric val>) [with_policy (<map policy>)]
          MAP_GET_BY_KEY      (<bin>, <key>)  
          MAP_REMOVE_BY_KEY   (<bin>, <key>)  
          MAP_GET_BY_VALUE    (<bin>, <value>)
          MAP_REMOVE_BY_VALUE (<bin>, <value>)
          MAP_GET_BY_INDEX    (<bin>, <index>)
          MAP_REMOVE_BY_INDEX (<bin>, <index>)
          MAP_GET_BY_RANK     (<bin>, <rank>) 
          MAP_REMOVE_BY_RANK  (<bin>, <rank>) 
          MAP_REMOVE_BY_KEY_LIST    (<bin>, <list of keys>)         
          MAP_REMOVE_BY_VALUE_LIST  (<bin>, <list of vals>)         
          MAP_GET_BY_KEY_RANGE      (<bin>, <startkey>, <endkey>)   
          MAP_REMOVEBY_RANGE        (<bin>, <startkey>, <endkey>)   
          MAP_GET_BY_VALUE_RANGE    (<bin>, <startval>, <endval>)   
          MAP_REMOVE_BY_VALUE_RANGE (<bin>, <startval>, <endval>)   
          MAP_GET_BY_INDEX_RANGE    (<bin>, <startindex>[, <count>])
          MAP_REMOVE_BY_INDEX_RANGE (<bin>, <startindex>[, <count>])
          MAP_GET_BY_RANK_RANGE     (<bin>, <startrank> [, <count>])
          MAP_REMOVE_BY_RANK_RANGE  (<bin>, <startrank> [, <count>])
          MAP_CLEAR     (<bin>) 
          MAP_SET_TYPE  (<bin>, <map type>) 
          MAP_SIZE      (<bin>) 
          TOUCH   ()            
          READ    (<bin>)       
          WRITE   (<bin>, <val>)
          PREPEND (<bin>, <val>)
          APPEND  (<bin>, <val>)
          INCR    (<bin>, <numeric val>)

      MAP_POLICY
          AS_MAP_UNORDERED
          AS_MAP_KEY_ORDERED
          AS_MAP_KEY_VALUE_ORDERED
          AS_MAP_UPDATE
          AS_MAP_UPDATE_ONLY
          AS_MAP_CREATE_ONLY

      Examples:

          OPERATE LIST_APPEND(listbin, 1), LIST_APPEND(listbin2, 10) ON test.demo where PK = 'key1'
          OPERATE LIST_POP_RANGE(listbin, 1, 10) ON test.demo where PK = 'key1'


  QUERY
      SELECT <bins> FROM <ns>[.<set>]
      SELECT <bins> FROM <ns>[.<set>] WHERE <bin> = <value>
      SELECT <bins> FROM <ns>[.<set>] WHERE <bin> BETWEEN <lower> AND <upper>
      SELECT <bins> FROM <ns>[.<set>] WHERE PK = <key>
      SELECT <bins> FROM <ns>[.<set>] IN <indextype> WHERE <bin> = <value>
      SELECT <bins> FROM <ns>[.<set>] IN <indextype> WHERE <bin> BETWEEN <lower> AND <upper>
      SELECT <bins> FROM <ns>[.<set>] IN <indextype> WHERE <bin> CONTAINS <GeoJSONPoint>
      SELECT <bins> FROM <ns>[.<set>] IN <indextype> WHERE <bin> WITHIN <GeoJSONPolygon>

          <ns> is the namespace for the records to be queried.
          <set> is the set name for the record to be queried.
          <key> is the record's primary key.
          <bin> is the name of a bin.
          <value> is the value of a bin.
          <indextype> is the type of a index user wants to query. (LIST/MAPKEYS/MAPVALUES)
          <bins> can be either a wildcard (*) or a comma-separated list of bin names.
          <lower> is the lower bound for a numeric range query.
          <upper> is the lower bound for a numeric range query.

      Examples:

          SELECT * FROM test.demo
          SELECT * FROM test.demo WHERE PK = 'key1'
          SELECT foo, bar FROM test.demo WHERE PK = 'key1'
          SELECT foo, bar FROM test.demo WHERE foo = 123
          SELECT foo, bar FROM test.demo WHERE foo BETWEEN 0 AND 999
          SELECT * FROM test.demo WHERE gj CONTAINS CAST('{"type": "Point", "coordinates": [0.0, 0.0]}' AS GEOJSON)

  AGGREGATION
      AGGREGATE <module>.<function>(<args>) ON <ns>[.<set>]
      AGGREGATE <module>.<function>(<args>) ON <ns>[.<set>] WHERE <bin> = <value>
      AGGREGATE <module>.<function>(<args>) ON <ns>[.<set>] WHERE <bin> BETWEEN <lower> AND <upper>

          <module> is UDF module containing the function to invoke.
          <function> is UDF to invoke.
          <args> is a comma-separated list of argument values for the UDF.
          <ns> is the namespace for the records to be queried.
          <set> is the set name for the record to be queried.
          <key> is the record's primary key.
          <bin> is the name of a bin.
          <value> is the value of a bin.
          <lower> is the lower bound for a numeric range query.
          <upper> is the lower bound for a numeric range query.

      Examples:

          AGGREGATE myudfs.udf2(2) ON test.demo WHERE foo = 123
          AGGREGATE myudfs.udf2(2) ON test.demo WHERE foo BETWEEN 0 AND 999

  EXPLAIN
      EXPLAIN SELECT * FROM <ns>[.<set>] WHERE PK = <key>

          <ns> is the namespace for the records to be queried.
          <set> is the set name for the record to be queried.
          <key> is the record's primary key.

      Examples:

          EXPLAIN SELECT * FROM test.demo WHERE PK = 'key1'


  INFO
      SHOW NAMESPACES | SETS | BINS | INDEXES
      SHOW SCANS | QUERIES
      STAT NAMESPACE <ns> | INDEX <ns> <indexname>
      STAT SYSTEM
      ASINFO <ASInfoCommand>

  JOB MANAGEMENT
      KILL_QUERY <transaction_id>
      KILL_SCAN <scan_id>

  USER ADMINISTRATION
      SHOW USER [<user>]
      SHOW USERS
      SHOW ROLE <role>
      SHOW ROLES

  MANAGE UDFS
      SHOW MODULES
      DESC MODULE <filename>

          <filepath> is file path to the UDF module(in single quotes).
          <filename> is file name of the UDF module.

      Examples:

          SHOW MODULES
          DESC MODULE test.lua

  RUN <filepath>

  SYSTEM <bash command>


  SETTINGS
        ECHO                          (true | false, default false)
        VERBOSE                       (true | false, default false)
        OUTPUT                        (TABLE | JSON | MUTE | RAW, default TABLE)
        OUTPUT_TYPES                  (true | false, default true)
        TIMEOUT                       (time in ms, default: 1000)
        SOCKET_TIMEOUT                (time in ms, default: -1)
        LUA_USERPATH                  <path>, default : /opt/aerospike/usr/udf/lua
        USE_SMD                       (true | false, default false)
        RECORD_TTL                    (time in sec, default: 0)
        RECORD_PRINT_METADATA         (true | false, default false, prints record metadata)
        REPLICA_ANY                   (true | false, default false)
        KEY_SEND                      (true | false, default false)
        DURABLE_DELETE                (true | false, default false)
        FAIL_ON_CLUSTER_CHANGE        (true | false, default true, policy applies to scans)
        SCAN_PRIORITY                 priority of scan (LOW, MEDIUM, HIGH, AUTO), default : AUTO
        NO_BINS                       (true | false, default false, No bins as part of scan and query result)
        LINEARIZE_READ                (true | false, default false, Make read linearizable, applicable only for namespace with strong_consistency enabled.)


      To get the value of a setting, run:

          aql> GET <setting>

      To set the value of a setting, run:

          aql> SET <setting> <value>

      To reset the value of a setting back to default, run:

          aql> RESET <setting>


    OTHER
        HELP
        QUIT|EXIT|Q
```

### 2.1 DDL（数据定义语言）

​		CREATE INDEX <index> ON <ns>[.<set>] (<bin>) NUMERIC | STRING | GEO2DSPHERE

​		给指定的namespace（数据库）.set（表）的元素创建索引。NUMERIC 数字索引、STRING 字符串索引、*GEO2DSPHERE（未知）*。

​		e.g: create index aa on dfp.dfp (createTime) numeric

​		DROP INDEX <ns>[.<set>] <index> 

​		删除指定的索引

​		DROP INDEX dfp.dfp aa

### 2.2 DML（数据操纵语言）

​		INSERT INTO <ns>[.<set>] (PK, <bins>) VALUES (<key>, <values>)

​		插入一条数据。

​		DELETE FROM <ns>[.<set>] WHERE PK = <key> 

​		删除一条数据。

​		TRUNCATE <ns>[.<set>] [upto <LUT>]

​		截断数据。

​		<ns> is the namespace for the record.          

​		<set> is the set name for the record.          

​		<key> is the record's primary key.          

​		<bins> is a comma-separated list of bin names.          

​		<values> is comma-separated list of bin values, which may include type cast expressions. Set to NULL (case insensitive & w/o quotes) to delete the bin.          		<LUT> is last update time upto which set or namespace needs to be truncated. LUT is either nanosecond since Unix epoch like 1513687224599000000 or in date string in format like "Dec 19 2017 12:40:00". 

​		Type Cast Expression Formats:

​				CAST(<Value> AS <TypeName>) 

​				<TypeName>(<Value>)

​		**aql支持的数据类型转换以及语法**

Supported AQL Types:               

Bin Value Type                    Equivalent Type Name(s)           ===============================================================            Integer                                  DECIMAL, INT, NUMERIC

Floating Point                     FLOAT, REAL            

Aerospike CDT (List, Map, etc.)   JSON            

Aerospike List                      LIST            

Aerospike Map                     MAP            

GeoJSON                               GEOJSON            

String                                    CHAR, STRING, TEXT, VARCHAR           ===============================================================         [Note:  Type names and keywords are case insensitive.]

例子：

INSERT INTO test.demo (PK, foo, bar) VALUES ('key1', 123, 'abc') 

INSERT INTO test.demo (PK, foo, bar) VALUES ('key1', CAST('123' AS INT), JSON('{"a": 1.2, "b": [1, 2, 3]}')) 

INSERT INTO test.demo (PK, foo, bar) VALUES ('key1', LIST('[1, 2, 3]'), MAP('{"a": 1, "b": 2}')) 

INSERT INTO test.demo (PK, gj) VALUES ('key1', GEOJSON('{"type": "Point", "coordinates": [123.4, -456.7]}'))   

DELETE FROM test.demo WHERE PK = 'key1'

### 2.3 QUERY（查询）

SELECT <bins> FROM <ns>[.<set>]

SELECT <bins> FROM <ns>[.<set>] WHERE <bin> = <value>

SELECT <bins> FROM <ns>[.<set>] WHERE <bin> BETWEEN <lower> AND <upper>

SELECT <bins> FROM <ns>[.<set>] WHERE PK = <key>

SELECT <bins> FROM <ns>[.<set>] IN <indextype> WHERE <bin> = <value>

SELECT <bins> FROM <ns>[.<set>] IN <indextype> WHERE <bin> BETWEEN <lower> AND <upper>

SELECT <bins> FROM <ns>[.<set>] IN <indextype> WHERE <bin> CONTAINS <GeoJSONPoint>

SELECT <bins> FROM <ns>[.<set>] IN <indextype> WHERE <bin> WITHIN <GeoJSONPolygon>

bins：元数据

ns：命名空间

set：表名

注意bin=的时候这个bin需要是key或者建立了索引的。

例子

SELECT * FROM test.demo          

SELECT * FROM test.demo WHERE PK = 'key1'          

SELECT foo, bar FROM test.demo WHERE PK = 'key1'          

SELECT foo, bar FROM test.demo WHERE foo = 123          

SELECT foo, bar FROM test.demo WHERE foo BETWEEN 0 AND 999          

SELECT * FROM test.demo WHERE gj CONTAINS CAST('{"type": "Point", "coordinates": [0.0, 0.0]}' AS GEOJSON) 

### 2.4 INFO（详情）

SHOW NAMESPACES | SETS | BINS | INDEXES 

SHOW SCANS | QUERIES

### 2.5 用户权限管理

​		CREATE USER <user> PASSWORD <password> ROLE[S] <role1>,<role2>...

​		创建用户并授予用户role1、role2...角色。

​		DROP USER <user>

​		删除用户

​		SET PASSWORD <password> [FOR <user>]

​		为用户创建密码

​		GRANT ROLE[S] <role1>,<role2>... TO <user>

​		授予用户roles1、role2...角色

​		REVOKE ROLE[S] <role1>,<role2>... FROM <user>

​		撤销用户role1、role2...角色

​		CREATE ROLE <role> PRIVILEGE[S] <priv1[.ns1[.set1]]>,<priv2[.ns2[.set2]]>... WHITELIST addr1,addr2...

​		priv: read|read-write|read-write-udf|sys-admin|user-admin|data-admin

​		ns:   namespace.  Applies to all namespaces if not set.

​	    set:  set name.  Applies to all sets within namespace if not set.

 	  addr: IP address.

​		创建角色，使该角色拥有对于ns.set的特定读写权限，并且限制了连接ip来源。





