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
###步骤3：修改spring配置文件
引入shiro.spring.xml
``` xml
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
	<property name="securityManager" ref="securityManager" />
	<property name="loginUrl" value="https://cas.hengtiansoft.com:8443/cas/login?service=yourApplication/shiro-cas" />
	<!-- exp -->
    <!--<property name="loginUrl" value="https://cas.hengtiansoft.com:8443/cas/login?service=http://feedback.hengtiansoft.com/shiro-cas"/> -->
	<property name="unauthorizedUrl" value="/accessDenied.html"></property>
	<property name="filterChainDefinitions">
		<value>
			/shiro-cas=casFilter
			/logout=logout
			/index.html=authc
			/**=authc
		</value>
	</property>
	<property name="filters">
		<util:map>
			<entry key="casFilter" value-ref="casFilter" />
		</util:map>
	</property>
</bean>

<bean id="logout" class="org.apache.shiro.web.filter.authc.LogoutFilter">
  	<property name="redirectUrl" value=" https://cas.hengtiansoft.com:8443/cas/logout?service=yourApplication"/>
  	<!-- exp -->
    <!--<property name="redirectUrl"  value="https://cas.hengtiansoft.com:8443/cas/logout
    ?service=http://feedback.hengtiansoft.com"/> -->
</bean>

<bean id="casFilter" class="org.apache.shiro.cas.CasFilter">
	<property name="failureUrl" value="error.jsp"></property>
</bean>

<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
	<property name="realm" ref="casRealm" />
	<property name="cacheManager" ref="cacheManager" />
</bean>

	<!--用户授权/认证信息Cache, 采用EhCache 缓存 -->
<bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager" />


<bean id="casRealm" class="MyCasRealm">
<property name="casServerUrlPrefix" value="https://cas.hengtiansoft.com:8443/cas " />
	<!--客户端的回调地址设置，必须和上面的shiro-cas过滤器拦截的地址一致 -->
	<property name="casService" value="yourApplication/shiro-cas" />
	<!-- exp -->
    <!--<property name="casService" value="http://feedback.hengtiansoft.com/shiro-cas" /> -->
</bean>

<!--保证实现了Shiro内部lifecycle函数的bean执行 -->
<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
	depends-on="lifecycleBeanPostProcessor">
	<property name="proxyTargetClass" value="true" />
</bean> 
<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>

<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
	<property name="securityManager" ref="securityManager" />
</bean>

```
>有单点登录情况下，登录认证是在 casserver 进行的，那么执行流程是这样的：用户从 cas server 登录成功后，跳到 cas client 的CasRealm执行默认的doGetAuthorizationInfo和doGetAuthenticationInfo，此时doGetAuthenticationInfo做的工作是把登录用户信息传递给 shiro ，保持默认即可，而对于授权的处理，可以通过MyCasRealm（继承CasRealm的自定义类）覆写doGetAuthorizationInfo进行自定义授权认证。

###步骤4：下载证书

从[恒天CAS页面](https://cas.hengtiansoft.com:8443/cas/login)下载证书

###步骤5：将证书添加到本地JVM信任库
下载来的证书需要添加到本地JVM信任库当中。`(需要管理员权限)`
* 首先进入jdk1.6.0_24\jre\lib\security
* 然后在security目录运行以下命令
	keytool -import -alias cacerts -keystore cacerts -file `D:\tiancan\cas.hengtiansoft.com.cer`  
	-- 高亮部分为步骤4下载下来的证书在你本地的路劲
	期间需要输入密码，密码为 `changeit`

###步骤6：获取CAS返回的信息
在java代码中获取 staffId ,staffName
``` java

List list = SecurityUtils.getSubject().getPrincipals().asList();
Map<String, Object> info = (Map<String, Object>)list.get(1);
if(info != null){
    String staffId = info.get("name").toString();
    List  value = (List)info.get("value");
    String staffId = (String)value.get(0);
    String staffName = (String)value.get(1);
}

`or`

String staffName = SecurityUtils.getSubject().getPrincipal().toString();

```







