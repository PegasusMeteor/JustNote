# CAS Serivce Management




CAS Service management




CAS management https://github.com/apereo/cas-management

https://github.com/apereo/cas-management-overlay


1. 修改jdk 版本。


org.gradle.java.home=/usr/lib/jdk-11.0.10


修改
casmgmt.version=6.3.0
因为已经没有snapshot版本了






2. 修改项目目录下的  /etc/cas/config/management.properties 


```

cas.server.name=http://10.0.41.74:8090
cas.server.prefix=${cas.server.name}/cas
mgmt.serverName=http://10.0.41.74:8091
mgmt.adminRoles[0]=ROLE_ADMIN
mgmt.userPropertiesFile=file:/etc/cas/config/users.json
server.port=8091
server.ssl.enabled=false
logging.config=file:/etc/cas/config/log4j2-management.xml
```

将 /etc/cas/config/ 目录下的内容，拷贝到系统的 同目录下。


执行 ./build.sh  package

3. 将打包好的cas-management.war 拷贝到 服务器上的指定位置。

运行  java -Xdebug -Xrunjdwp:transport=dt_socket,address=5001,server=y,suspend=n -jar cas-management.war

nohup  java -Xdebug -Xrunjdwp:transport=dt_socket,address=5001,server=y,suspend=n -jar cas-management.war > cas-management.log 2>&1 &


nohup java -jar cas-management.war  > cas-management.log 2>&1 &


如果 与 cas 服务运行在同一个主机上，需要注意配置是否会相互覆盖。

或者使用 tomcat 来部署多个应用。

如果使用tomcat 部署多个应用的话，需要添加下面的配置。

```xml
 <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
        <Context docBase="cas" path="/cas" reloadable="true"/>
        <Context docBase="cas-management" path="/cas-management" reloadable="true"/>

      </Host>
```








