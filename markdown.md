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
###步骤2：修改web.xml

添加shiro filter，如果你的服务器不支持servlet 3.0,把\<async-supported\>true\</async-supported\>注释掉,注意SingleSignOutFilter需要在shiroFilter之前。
##### web.xml
``` xml
<!—cas logout filter -->
	<filter>
		<filter-name>CAS Single Sign Out Filter</filter-name>
		<filter-class>org.jasig.cas.client.session.SingleSignOutFilter</filter-class>
	  </filter>
	
	  <filter-mapping>
		<filter-name>CAS Single Sign Out Filter</filter-name>
		<url-pattern>/shiro-cas</url-pattern>
	  </filter-mapping>
	
	<listener>
	<listener-class>
	org.jasig.cas.client.session.SingleSignOutHttpSessionListener
	</listener-class>
	</listener>
	
	<!-- shiro -->
	<filter>
		<filter-name>shiroFilter</filter-name>
		<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
		<async-supported>true</async-supported>
		<init-param>
			<param-name>targetFilterLifecycle</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>shiroFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```














