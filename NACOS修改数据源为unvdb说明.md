# nacos 2.2.4版本MySQL数据源修改为unvdb数据源说明
- 依赖环境要求：jdk17+，maven3.6+
- 可以直接使用distribution模块的 [pak/nacos-server-2.2.4.tar.gz](http://10.99.16.144:9999/mysb-back-end/appeal-despatch-nacos/-/blob/develop/distribution/pak/nacos-server-2.2.4.tar.gz)包，解压后直接运行bin/startup.sh即可。
- 也可以使用distribution模块的 [pak/nacos-server-2.2.4.tar.gz](http://10.99.16.144:9999/mysb-back-end/appeal-despatch-nacos/-/blob/develop/distribution/pak/nacos-server-2.2.4.tar.gz)包，解压后修改conf/application.properties文件，配置unvdb数据源相关配置，然后运行bin/startup.sh即可。
- 注意事项：
1. 请在distribution模块 [/conf/application.properties](http://10.99.16.144:9999/mysb-back-end/appeal-despatch-nacos/-/blob/develop/distribution/conf/application.properties)文件中配置unvdb数据源相关配置如下。
```properties
spring.sql.init.platform=unvdb
db.num=1
db.url.0=jdbc:postgresql://10.1.48.215:5679/appeal-nacos-dev?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=unvdb
db.password.0=KYHjyJNOKB1fV7Lz
db.pool.config.driverClassName=org.postgresql.Driver
# Connection pool configuration: hikariCP
db.pool.config.connectionTimeout=30000
db.pool.config.validationTimeout=10000
db.pool.config.maximumPoolSize=20
db.pool.config.minimumIdle=2
```
## 1. 引入unvdb驱动包 
- 在项目根目录下pom.xml添加unvdb驱动maven节点依赖，版本：42.2.5
```text
<unvdb.version>42.2.5</unvdb.version>
        
<!-- 引入unvdb数据库驱动包-->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>${unvdb.version}</version>
</dependency>
```
- 在config模块pom.xml添加maven节点依赖
```text
<dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
</dependency>
```
## 2. 在plugin工程模块下，增加插件的方法修改SLQ语法
- plugin模块下 /plugin/datasource/src/main/resources/META-INF/services 在该目录下的
  com.alibaba.nacos.plugin.datasource.mapper.Mapper配置文件中最下面增加以下数据。

```text
com.alibaba.nacos.plugin.datasource.impl.unvdb.ConfigInfoAggrMapperByUnvDb
com.alibaba.nacos.plugin.datasource.impl.unvdb.ConfigInfoBetaMapperByUnvDb
com.alibaba.nacos.plugin.datasource.impl.unvdb.ConfigInfoMapperByUnvDb
com.alibaba.nacos.plugin.datasource.impl.unvdb.ConfigInfoTagMapperByUnvDb
com.alibaba.nacos.plugin.datasource.impl.unvdb.ConfigTagsRelationMapperByUnvDb
com.alibaba.nacos.plugin.datasource.impl.unvdb.HistoryConfigInfoMapperByUnvDb
com.alibaba.nacos.plugin.datasource.impl.unvdb.TenantInfoMapperByUnvDb
com.alibaba.nacos.plugin.datasource.impl.unvdb.TenantCapacityMapperByUnvDb
com.alibaba.nacos.plugin.datasource.impl.unvdb.GroupCapacityMapperByUnvDb
```
- 在com.alibaba.nacos.plugin.datasource.constants.DataSourceConstant.java类中增加参数常量配置
```text
 public static final String UNVDB = "unvdb";
```
- 在plugin模块com.alibaba.nacos.plugin.datasource.impl路径下增加unvdb包文件夹，根据SPI机制进行代码扩展。
```text
ConfigInfoAggrMapperByUnvDb.java
ConfigInfoBetaMapperByUnvDb.java
ConfigInfoMapperByUnvDb.java
ConfigInfoTagMapperByUnvDb.java
ConfigTagsRelationMapperByUnvDb.java
HistoryConfigInfoMapperByUnvDb.java
TenantInfoMapperByUnvDb.java
TenantCapacityMapperByUnvDb.java
GroupCapacityMapperByUnvDb.java
```
## 本地IDE开发工具启动NACOS 进行测试验证
- 在distribution模块， /conf/application.properties文件中配置unvdb数据源相关配置如下。
```properties
spring.sql.init.platform=unvdb
db.num=1
db.url.0=jdbc:postgresql://10.1.48.215:5679/appeal-nacos-dev?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=unvdb
db.password.0=KYHjyJNOKB1fV7Lz
db.pool.config.driverClassName=org.postgresql.Driver
# Connection pool configuration: hikariCP
db.pool.config.connectionTimeout=30000
db.pool.config.validationTimeout=10000
db.pool.config.maximumPoolSize=20
db.pool.config.minimumIdle=2
```
- 在启动类jvm中配置单机启动 
```properties
# 单机启动： standalone 参数，部署集群请查看官方文档
-Dnacos.standalone=true
```
- 启动 console 模块 com.alibaba.nacos.Nacos.java 类。
- 启动成功后，访问http://localhost:8848/nacos/index.html，登录账号：nacos，密码：nacos。

## 编译 NACOS
- 在 appeal-despatch-nacos 工程跟目录下执行以下编译命令。
```text
mvn -Prelease-nacos -Dmaven.test.skip=true -Dpmd.skip=true -Dcheckstyle.skip=true  -Drat.skip=true clean install -U
```
## NACOS 表数据脚本
- SQL脚本位置，在 appeal-despatch-nacos\distribution\conf 目录下
  - 说明：自增id调整为seq序列，时间问题调整等，[unvdb-schema.sql](http://10.99.16.144:9999/mysb-back-end/appeal-despatch-nacos/-/blob/develop/distribution/conf/unvdb-schema.sql)脚本是已经调整过的脚本，可以直接使用。
```text
# 数据库初始化脚本
appeal-despatch-nacos\distribution\conf\unvdb-schema.sql
```

## 启动 NACOS
- 启动脚本，在 appeal-despatch-nacos\distribution\bin 目录下
- jar包位置，生成的jar包在 appeal-despatch-nacos\distribution\target 目录下
```text
#启动命令(standalone代表着单机模式运行，非集群模式):   
启动：startup.cmd -m standalone
启动：sh startup.sh -m standalone

#集群启动：
启动：startup.cmd
启动：sh startup.sh

#停止命令:
停止：shutdown.cmd
停止：sh shutdown.sh
```
