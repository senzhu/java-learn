#基于 Apache  shiro  的恒天 SSO (CAS)解决方案
>该方案为CAS和Shiro在spring中集成

###步骤1：引入相关jar包
如果你的项目是maven project,请在pom.xml里添加shiro及CAS相关的jar.
##### pom
``` xml
	<properties>	
		<shiro.version>1.2.2</shiro.version>	
	</properties>
<!-- Shiro -->
	<dependency>
		<groupId>org.apache.shiro</groupId>
		<artifactId>shiro-spring</artifactId>
		<version>${shiro.version}</version>
	</dependency>
	<dependency>
		<groupId>org.apache.shiro</groupId>
		<artifactId>shiro-ehcache</artifactId>
		<version>${shiro.version}</version>
	</dependency>
	<dependency>
		<groupId>org.apache.shiro</groupId>
		<artifactId>shiro-aspectj</artifactId>
		<version>${shiro.version}</version>
	</dependency>

	<dependency>
       		<groupId>org.apache.shiro</groupId>
      		<artifactId>shiro-cas</artifactId>
		 <version>${shiro.version}</version>
    	</dependency>

	<dependency>
		<groupId>net.sf.ehcache</groupId>
		<artifactId>ehcache-core</artifactId>
		<version>2.6.8</version>
	</dependency>
```
