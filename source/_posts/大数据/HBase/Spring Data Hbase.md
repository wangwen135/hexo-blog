---
title: Spring Data Hbase
date: 2016-09-23 22:32
tags: 
  - HBase
  - Spring
  - Spring Data
categories:
  - [大数据, HBase]
---


https://docs.spring.io/spring-hadoop/docs/current/reference/html/springandhadoop-hbase.html


通过hbase-configuration命名空间元素（或其支持HbaseConfigurationFactoryBean）为HBase提供基本配置。
```
<!-- default bean id is 'hbaseConfiguration' that uses the existing 'hadoopCconfiguration' object -->
<hdp:hbase-configuration configuration-ref="hadoopCconfiguration" />
```

配置连接

```
<!-- delete associated connections but do not stop the proxies -->
<hdp:hbase-configuration stop-proxy="false" delete-connection="true">
  foo=bar
  property=value
</hdp:hbase-configuration>
```

```
<!-- specify ZooKeeper host/port -->
<hdp:hbase-configuration zk-quorum="${hbase.host}" zk-port="${hbase.port}">
```

属性和配置文件
```
<hdp:hbase-configuration properties-ref="some-props-bean" properties-location="classpath:/conf/testing/hbase.properties"/>

```

核心HbaseTemplate--一个与HBase交互的高层次抽象。该模板需要HBase配置，一旦设置，该模板就是线程安全的，并且可以同时在多个实例中重用：

```
// default HBase configuration
<hdp:hbase-configuration/>

// wire hbase configuration (using default name 'hbaseConfiguration') into the template
<bean id="htemplate" class="org.springframework.data.hadoop.hbase.HbaseTemplate" p:configuration-ref="hbaseConfiguration"/>
```

使用例子
```
// writing to 'MyTable'
template.execute("MyTable", new TableCallback<Object>() {
  @Override
  public Object doInTable(HTable table) throws Throwable {
    Put p = new Put(Bytes.toBytes("SomeRow"));
    p.add(Bytes.toBytes("SomeColumn"), Bytes.toBytes("SomeQualifier"), Bytes.toBytes("AValue"));
    table.put(p);
    return null;
  }
});
```

```
// read each row from 'MyTable'
List<String> rows = template.find("MyTable", "SomeColumn", new RowMapper<String>() {
  @Override
  public String mapRow(Result result, int rowNum) throws Exception {
    return result.toString();
  }
}));
```


配置文件例子
```
  <hdp:configuration id="hadoopConfiguration">
        fs.defaultFS=${spring.hadoop.fsUri}
        <!-- hadoop.tmp.dir=/tmp/hadoop -->
    </hdp:configuration>

    <hdp:file-system id="hadoopFs" configuration-ref="hadoopConfiguration" />

    <hdp:hbase-configuration id="hbaseConfiguration" configuration-ref="hadoopConfiguration" zk-quorum="${hbase.zk.host}" zk-port="${hbase.zk.port}" />

    <bean id="hbaseTemplate" class="org.springframework.data.hadoop.hbase.HbaseTemplate">
        <property name="configuration" ref="hbaseConfiguration" />
    </bean>

    <!-- 配置一个connectionfactory使用原生的API -->
    <bean id='hbaseConnection' class='org.apache.hadoop.hbase.client.ConnectionFactory' factory-method="createConnection" destroy-method="close">
        <constructor-arg index="0" ref="hbaseConfiguration" />
    </bean>
```
>**Connection 说明：**  
创建连接是一个重量级操作。连接实现是线程安全的，因此客户端可以创建一个连接，并与不同的线程共享它。Table和Admin实例是轻量级的，并且不是线程安全的。通常，每个客户端应用程序都实例化一个连接，每个线程都将获得自己的Table实例。不推荐使用Table和Admin的缓存或池。

```
@Component
public class IconTableInit implements InitializingBean {

    private static Logger logger = LoggerFactory.getLogger(IconTableInit.class);

    @Autowired
    private Connection connection;

    @Override
    public void afterPropertiesSet() throws Exception {
        // 如果表不存在则创建表
        Admin admin = null;
        try {
            admin = connection.getAdmin();
            if (!admin.tableExists(TableName.valueOf(TABLE_NAME))) {

                logger.info("HBase中不存在{}表！", TABLE_NAME);

                HTableDescriptor tableDescriptor = new HTableDescriptor(TableName.valueOf(TABLE_NAME));
                HColumnDescriptor columnDescriptor = new HColumnDescriptor(CF_INFO_B);
                tableDescriptor.addFamily(columnDescriptor);
                admin.createTable(tableDescriptor);

                logger.info("{}表创建成功！", TABLE_NAME);
            }
        } catch (Exception e) {
            logger.error("系统启动时检查icon表是否创建异常！", e);
            throw new GeneralException("#####  系统启动时连接HBase出现异常！ #####");
        } finally {
            if (admin != null) {
                admin.close();
            }
        }
    }
}
```
