---
title: Shiro 整合 Spring Boot 控制角色权限
date: 2018-01-25 20:29:17
tags:
---

安全无处不在，趁着放假读了一下 Shiro 文档，并记录一下 Shiro 整合 Spring Boot 在数据库中根据角色控制访问权限

# 简介
Apache Shiro是一个功能强大、灵活的，开源的安全框架。它可以干净利落地处理身份验证、授权、企业会话管理和加密。
![Shiro Architecture](./ShiroFeatures.png)

上图是  Shiro 的基本架构

### Authentication（认证）
有时被称为“登录”，用来证明用户是用户他们自己本人

###  Authorization（授权）
访问控制的过程，即确定“谁”访问“什么”

###  Session Management（会话管理）
管理用户特定的会话，在 Shiro 里面可以发现所有的用户的会话信息都会由 Shiro 来进行控制

### Cryptography（加密）
在对数据源使用加密算法加密的同时，保证易于使用

# Start

### 环境
Spring Boot 1.5.9
MySQL 5.7
Maven 3.5.2
Spring Data Jpa
Lombok

### 添加依赖
这里只给出主要的 Shiro 依赖
```
		<dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring-boot-starter</artifactId>
            <version>1.4.0-RC2</version>
        </dependency>
```
### 配置
我们暂时只需要用户表、角色表，在 Spring boot 中修改配置文件将自动为我们创建数据库表
```
server:
  port: 8888
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root
    url: jdbc:mysql://localhost:3306/shiro?characterEncoding=utf-8&useSSL=false
  jpa:
    generate-ddl: true
    hibernate:
      ddl-auto: update
    show-sql: true
```
### 实体
#### Role.java
```
@Data
@Entity
public class Role {

	@Id
	@GeneratedValue
	private Integer id;

	private Long userId;

	private String role;

}
```
#### User.java
```
@Data
@Entity
public class User {
	@Id
	@GeneratedValue
	private Long id;

	private String username;

	private String password;

}
```

### Realm
首先建立 Realm 类，继承自 AuthorizingRealm，自定义我们自己的授权和认证的方法。Realm 是可以访问特定于应用程序的安全性数据（如用户，角色和权限）的组件。
#### Realm.java
```
public class Realm extends AuthorizingRealm {

	@Autowired
	private UserService userService;
	
	//授权
	@Override
	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
		//从凭证中获得用户名
		String username = (String) SecurityUtils.getSubject().getPrincipal();
		//根据用户名查询用户对象
		User user = userService.getUserByUserName(username);
		//查询用户拥有的角色
		List<Role> list = roleService.findByUserId(user.getId());
		SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
		for (Role role : list) {
			//赋予用户角色
			info.addStringPermission(role.getRole());
		}
		return info;
	}

	//认证
	@Override
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {

		//获得当前用户的用户名
		String username = (String) authenticationToken.getPrincipal();

		//从数据库中根据用户名查找用户
		User user = userService.getUserByUserName(username);
		if (userService.getUserByUserName(username) == null) {
			throw new UnknownAccountException(
					"没有在本系统中找到对应的用户信息。");
		}

		SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user.getUsername(), user.getPassword(),getName());
		return info;
	}

}

```

### Shiro 配置类
#### ShiroConfig.java
```
@Configuration
public class ShiroConfig {

	@Bean
	public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
		ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
		shiroFilterFactoryBean.setSecurityManager(securityManager);
		Map<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
		//以下是过滤链，按顺序过滤，所以/**需要放最后
		//开放的静态资源
		filterChainDefinitionMap.put("/favicon.ico", "anon");//网站图标
		filterChainDefinitionMap.put("/**", "authc");
		shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
		return shiroFilterFactoryBean;
	}


	@Bean
	public DefaultWebSecurityManager securityManager() {
		DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager(myRealm());
		return defaultWebSecurityManager;
	}

	@Bean
	public MyRealm myRealm() {
		MyRealm myRealm = new MyRealm();
		return myRealm;
	}
}
```
### 控制器
#### UserController.java
```
@Controller
public class UserController {

	@Autowired
	private UserService userService;

	@GetMapping("/")
	public String index() {
		return "index";
	}

	@GetMapping("/login")
	public String toLogin() {
		return "login";
	}

	@GetMapping("/admin")
	public String admin() {
		return "admin";
	}

	@PostMapping("/login")
	public String doLogin(String username, String password) {
		UsernamePasswordToken token = new UsernamePasswordToken(username, password);
		Subject subject = SecurityUtils.getSubject();
		try {
			subject.login(token);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return "redirect:admin";
	}

	@GetMapping("/home")
	public String home() {
		Subject subject = SecurityUtils.getSubject();
		try {
			subject.checkPermission("admin");
		} catch (UnauthorizedException exception) {
			System.out.println("没有足够的权限");
		}
		return "home";
	}

	@GetMapping("/logout")
	public String logout() {
		return "index";
	}
}

```

### Service
#### UserService.java

```
@Service
public class UserService {

	@Autowired
	private UserDao userDao;

	public User getUserByUserName(String username) {
		return userDao.findByUsername(username);
	}

	@RequiresRoles("admin")
	public void send() {
		System.out.println("我现在拥有角色admin，可以执行本条语句");
	}

}
```
### 展示层
#### admin.html
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<html lang="en"/>
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
</head>
<body>

<form action="/login" method="post">
    <input type="text" name="username" />
    <input type="password" name="password" />
    <input type="submit" value="登录" />
</form>

</body>
</html>
```
#### home.html
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<html lang="en"/>
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
</head>
<body>
home
</body>
</html>
```

#### index.html
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<html lang="en"/>
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
</head>
<body>
index
<a href="/login">请登录</a>
</body>
</html>
```

#### login.html
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<html lang="en"/>
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
</head>
<body>

<form action="/login" method="post">
    <input type="text" name="username" />
    <input type="password" name="password" />
    <input type="submit" value="登录" />
</form>

</body>
</html>
```

### 总结
这个小案例实现了根据角色来控制用户访问，其中最重要的就是 Realm，它充当了Shiro与应用安全数据间的“桥梁”或者“连接器”
