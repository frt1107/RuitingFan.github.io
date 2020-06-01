# 数据服务规范

## 工程结构规范

### spring boot工程代码结构：

1. constant：主要用来写各种全局变量类，类名称命名按照业务常量进行命名；
2. enums：主要用来写各种枚举类；
3. entity：主要用来写各种实体类，类命名命名按照具体实体进行命名；例如：UserEntity；
4. property：主要用来写各种配置，比如spring boot要增加https支持，就需要写对应的配置，其他类似；
5. filter：主要用来写各种过滤逻辑，比如加了验证的接口，请求要过滤的需要卸载这里；
6. client：数据存储客户端，主要针对对spring boot支持不好的组件，例如hbase、elasticsearch等大数据存储；类命名按照数据存储组件进行命名；例如：HBaseClient
7. repository：数据读取层，用来读取各种数据库的数据，这里最好写通用的读取数据接口；此层主要使用spring boot内置的数据连接或者client里面的数据连接来读取数据；类命名按照数据存储组件进行命名；例如：HBaseRepository；
8. service：数据服务层，此层主要用来做业务服务，分类服务接口和服务实现；从repository层获取数据，并供controller层调用；类命名按照业务功能进行命名；例如：UserService，UserServiceImpl；
9. controller：数据控制层，主要用来写数据接口，对外提供数据服务，调用service层接口；类命名按照业务功能进行命名；例如：UserController
10. util：主要用来编写各种通用工具类；例如StringUtils

### 日志规范

1. 代码中不能使用system进行输出，所有日志必须使用日志框架slf4j进行输出；
2. log.info中输出日志不能拼接字符串，必须以如下方式输出:logger.info("info:{}",id)

### 代码打包规范

1. 代码打包使用spring默认 maven插件和maven-compiler插件进行打包；
2. 所有的配置文件必须排除在完整jar包之外；以便工程部署时能对配置文件进行修改。
3. 打包示例：

```xml
<build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.kevin.monitor.Application</mainClass>
                    <layout>ZIP</layout>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>

        </plugins>
        <resources>
            <resource>
                <filtering>true</filtering>
                <directory>src/main/resources/</directory>
                <excludes>
                    <exclude>application.properties</exclude>
                    <exclude>logback-spring.xml</exclude>
                    <exclude>keystore.p12</exclude>
                </excludes>
            </resource>
        </resources>
    </build>
```

### 工程部署规范

1. 工程部署必须新建目录，最好以服务名称进行命名；
2. 工程的jar包和配置文件放在同一路径下；
3. 启动时必须指定classpath路径
4. 目录命名规范如下：部署目录/home/ibs_iie/rest_service/${service_name}/，其中service_name为服务名称

工程部署脚本示例：

```bash
#!/bin/bash
prog={programname} #自定义程序名称

start(){
	echo $"$prog is starting...."
	nohup java  -Dfile.encoding=utf-8 -Dloader.path=. -jar {jar} 2 > /dev/null 1>&1 &

}

restart(){
	stop
	start
}
status(){
	pid=`ps -ef|grep $prog | grep -v grep | awk '{print $2}'`
	if [ "${pid}" != "" ];then
		echo $"$prog is running with pid: $pid"
	else
		echo $"$prog is stopped!"
	fi
}
stop(){
	echo $"Stoppint $prog: "
	pid=`ps -ef | grep $prog | grep -v grep | awk '{print $2}'`
	kill -9 $pid
	echo $"$prog with $pid stopped!"
}

case "$1" in
	start)
		start
		;;
	restart)
		restart
		;;
	stop)
		stop
		;;
	status)
		status
		;;
	*)
		echo $"Usage: $prog {start|stop|restart|status||help}"
esac
exit 
```

## 服务接口规范

### 接口地址规范

```shell
https://ip:port/{project.name}/api/{version}/{model}/{interface_name}
参数说明：
	project.name:代表服务名称，例如xxx_service；
	version:版本号，例如v1；
	model：模块名称，例如profile；
	interface_name：接口名称，例如xxx_search
```

### 接口参数规范

1. 参数名称必须小写，如果有多个部分组成，则必须以下划线“_”进行连接，比如：user_id；
2. 所有查询类接口，必须加分页

### 响应结果规范

1. 响应结果必须使用json格式进行返回；
2. 响应的状态使用默认http请求状态；
3. 响应结果中具体的数据结果必须以result进行标识，result中写具体的请求结果，其他字段按需设置；
4. 接口请求异常的，除了有http异常码之外，返回结果必须将异常信息返回；

## 安全规范

1. 所有接口采用https形式；
2. 接口请求至少使用简单token验证；