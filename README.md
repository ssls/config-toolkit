# 分布配置工具包

#### 在大型集群和分布式应用中，配置不宜分散到集群结点中，应该集中管理。

<hr>

#### 依赖
* JAVA 6+
* TOMCAT 7+

#### 模块
* EasyZk - 封装应用到zookeeper的属性配置的获取及更新
* ConfigWeb - 提供web界面维护属性配置，提供配置导入导出功能

#### 特性
* 集中管理集群配置
* 实现配置热更新
* Spring集成
* 本地配置覆盖
* 配置管理web界面
* 版本控制，支持灰度发布
* 支持为配置项添加注释

#### RELEASE NOTES
[https://github.com/dangdangdotcom/config-toolkit/wiki/release-notes](https://github.com/dangdangdotcom/config-toolkit/wiki/release-notes "Release Notes")

#### 词典
* ConfigFactory - 配置工厂，用于定义zookeeper地址及配置根目录
* ConfigNode - 配置组，一组配置信息，对应于遗留系统中的properties属性文件

#### 使用
- maven
<pre><code>
    &lt;dependency&gt;
      &lt;groupId&gt;com.dangdang&lt;/groupId&gt;
      &lt;artifactId&gt;config-toolkit-easyzk&lt;/artifactId&gt;
      &lt;version&gt;2.0.2-RELEASE&lt;/version&gt;
    &lt;/dependency&gt;
</code></pre>
- 直接使用
<pre><code>
    // 创建配置工厂指向zk的地址及配置在zk中的根地址，映射到zookeeper中的/projectx/modulex
	ConfigFactory configFactory = new ConfigFactory("zoo.host1:8181,zoo.host2:8181,zoo.host3:8181", "/projectx/modulex");
    // 从工厂中加载某配置组，映射到zookeeper中的/projectx/modulex/group0
	ConfigNode node = configFactory.getConfigNode("group0");
    // 从配置组中获取某配置，映射到zookeeper中的/projectx/modulex/group0/name
	Assert.assertNotNull(node.getProperty("name"));
</code></pre>

- Spring PlaceholderConfigurer集成
<pre><code>
	&lt;bean id="configFactory" class="com.dangdang.config.service.easyzk.ConfigFactory"&gt;
		&lt;constructor-arg name="connectStr" value="zoo.host1:8181,zoo.host2:8181,zoo.host3:8181" /&gt;
		&lt;constructor-arg name="rootNode" value="/projectx/modulex" /&gt;
	&lt;/bean&gt;

	&lt;bean id="zookeeperSources" class="com.dangdang.config.service.easyzk.support.spring.ZookeeperSourceFactory" factory-method="create"&gt;
		&lt;constructor-arg name="configFactory" ref="configFactory" /&gt;
		&lt;constructor-arg name="nodes"&gt;
			&lt;list&gt;
				&lt;value&gt;config-group1&lt;/value&gt;
				&lt;value&gt;config-group2&lt;/value&gt;
				&lt;value&gt;config-group3&lt;/value&gt;
			&lt;/list&gt;
		&lt;/constructor-arg&gt;
	&lt;/bean&gt;

	&lt;bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer"&gt;
		&lt;property name="order" value="1" /&gt;
		&lt;property name="ignoreUnresolvablePlaceholders" value="true" /&gt;
		&lt;property name="propertySources" ref="zookeeperSources" /&gt;
	&lt;/bean&gt;
</code></pre>

- spring spel集成

旧util properties用法：
<pre><code>
	&lt;util:properties id="configToolkitCommon" location="classpath:config-toolkit.properties" /&gt;
</code></pre>
Config-toolkit支持：
<pre><code>
	&lt;bean id="configToolkitCommon" class="com.dangdang.config.service.easyzk.ConfigNode" 
		factory-bean="configFactory" factory-method="getConfigNode"&gt;
		&lt;constructor-arg name="node" value="config-toolkit" /&gt;
	&lt;/bean&gt;
</code></pre>

- 本地配置覆盖(一般用于调试集群中的单点)

> 在classpath下添加本地配置文件,格式为XML,默认为local-override.xml,可以通过指定环境变量来修改
`-Dlocal-override.file=yourfile.xml`<br/>
[例]：
<pre><code>
	&lt;?xml version="1.0" encoding="UTF-8"?&gt;
	&lt;node-factories xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://crnlmchina.github.io/local-override.xsd"&gt;
		&lt;node-factory root="/projectx/modulex"&gt;
			&lt;group id="property-group1"&gt;
				&lt;node key="string_property_key"&gt;Welcome here.&lt;/node&gt;
			&lt;/group&gt;
		&lt;/node-factory&gt;
	&lt;/node-factories&gt;
</code></pre>

> 以上的配置会覆盖zookeeper中`/projectx/modulex/property-group1/string_property_key`的值为`Welcome here.`

- Config Web 管理界面

> 为避免误操作其它数据，Config Web将不再提供授权功能，仅保留鉴权，遗留系统的密码需要重置。<br/>
鉴权密码为节点的值，使用SHA1 HEX字符串加密，请手动操作zookeeper命令行创建密码。<br/>
一般linux系统都带有python，可以使用python脚本方便生成：<br/>

> `python -c "import hashlib;print hashlib.sha1('abc').hexdigest();"`
> 
> `# a9993e364706816aba3e25717850c26c9cd0d89d`
> 
> `echo "set /aaa/bbb a9993e364706816aba3e25717850c26c9cd0d89d" |./zkCli.sh -server localhost:2181`

![Config Web Snapshot](http://crnlmchina.github.io/config-web2.jpg)