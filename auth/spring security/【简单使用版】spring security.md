# Security



## 6，自定义登录逻辑

> springSecurity要求自定义登录逻辑必须有PasswordEncoder密码解析器

```java
/**
 * Security 配置类
 *
 * @author yanggy
 * @date 2021/10/12 23:49
 */
@Configuration
public class SecurityConfig {

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```



```java
/**
 *  UserDetailsService 实现类
 *
 * @author yanggy
 * @date 2021/10/12 23:51
 */
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private PasswordEncoder passwordEncoder;

    /**
     *查询数据库用户，生成用户信息用户名，密码，权限
     *
     * @param username 用户输入的用户名
     * @return SpringSecurity的User
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 查询数据库，判断用户是否存在
        if (!username.equals("用户名")) {
            throw new UsernameNotFoundException("用户名不存在");
        }
        // 把查询处理的密码进行解析，或者直接放入构造方法中
        String encodePassword = passwordEncoder.encode("密码");
        return new User(username, encodePassword, AuthorityUtils.commaSeparatedStringToAuthorityList("权限1,权限2"));
    }
}
```



## 7, 自定义登录页面



**配置**

```java
protected void configure(HttpSecurity http) throws Exception {
    // 表单提交
    http.formLogin()
        // 前台表达登录地址，当发现/login表达提交地址后，去执行UserDetailServiceImpl
        .loginProcessingUrl("/login")
        // UsernamePasswordAuthenticationFilter 过滤器进行认证
        // 自定义用户参数名
        .usernameParameter("username")
        // 自定义密码参数名
        .passwordParameter("password")
        // 自定义登录页面
        .loginPage("/login.html")
        // 登录成功后的跳转请求路径, Post请求
        .successForwardUrl("/toSuccessLogin")
        // 登录失败后跳转请求路径，Post请求
        .failureForwardUrl("/toLoginError");

    // 授权认证
    http.authorizeRequests()
        // error.html 不需要被认证
        .antMatchers("/error.html").permitAll()
        // login.html不需要被认真
        .antMatchers("/login.html").permitAll()
        // 其他所有请求需要被认真，必须登录之后被访问
        .anyRequest().authenticated();

    // 关闭csrf（防火墙）
    http.csrf().disable();

}
```



## 10, 自定义成功登录处理器

> 现在的项目大多是前后端分离的，或者跳转到如百度页面这个要站外跳转是办不到的。需要自定义成功处理器 **AuthenticationSuccessHandler**
>
> 看源码发现：successForwardUrl（）其实调用的时successHandler方法
>
>  this.successHandler(new ForwardAuthenticationSuccessHandler(forwardUrl));
>
> public class ForwardAuthenticationSuccessHandler implements **AuthenticationSuccessHandler**

```java
/**
 * 自定义成功登录处理器，与.successForwardUrl("/toSuccessLogin")有冲突保留其一
 * 不需要交给spring
 * @author yanggy
 * @date 2021/10/13 20:19
 */
public class CustomAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    
    private String url;
    
    public CustomAuthenticationSuccessHandler(String url) {
        this.url = url;
    }
    
    @Override
    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
        // 重定向
        httpServletResponse.sendRedirect("http://www.baidu.com");
        // 获取用户
        User principal = (User) authentication.getPrincipal();
    }
}
```

**配置: 与**.successForwardUrl保留其一

```java
http.formLogin()
                // 前台表达登录地址，当发现/login表达提交地址后，去执行UserDetailServiceImpl
                .loginProcessingUrl("/login")
                // UsernamePasswordAuthenticationFilter 过滤器进行认证
                // 自定义用户参数名
                .usernameParameter("username")
                // 自定义密码参数名
                .passwordParameter("password")
                // 自定义登录页面
                .loginPage("/login.html")
                // 登录成功后的跳转请求路径, Post请求
                //.successForwardUrl("/toSuccessLogin")
                // 自定义成功登录处理器，与.successForwardUrl("/toSuccessLogin")有冲突保留其一
                .successHandler(new CustomAuthenticationSuccessHandler())
                // 登录失败后跳转请求路径，Post请求
                .failureForwardUrl("/toLoginError");
```



## 11, 自定义失败处理器

> 同自定义成功登录处理器

```java
/**
 * 自定义失败登录处理器（同成功登录处理器）
 * 不需要给spring 配置是new即可
 * @author yanggy
 * @date 2021/10/13 20:31
 */
public class CustomAuthenticationFailureHandler implements AuthenticationFailureHandler {

    private String url;

    public CustomAuthenticationFailureHandler(String url) {
        this.url = url;
    }

    @Override
    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
        // 失败处理
    }
}
```



## 13, 角色权限判断

*   antMatch().hasAuthority(“xxx”)
*   antMatch().hasAnyAuthority(“xxx”)

**PS:** 严格区分大小写



> 判断用户是否具有特定的权限，用户的权限是自定义等逻辑逻辑创建User对象时指定的。



## 14 .anyRequest()

> Security 拦截请求，放行某些请求
>
> 一般最后 处理没有被匹配的请求
>
>  .anyRequest().authenticated();



## 15，访问控制的方法

在 ExpressionUrlAuthorizationConfigure类中可以查看

* permitAll

> 允许任何人访问

* denyAll

> 所有人都不能访问

* anonymous

> 可以匿名访问，和permitAll相似，但是执行到拦截链中

* authenticated

> 需要认证才能访问

* fullyAuthenticated

> 需要完成认证

* remeberMe

> 勾选了记住我，才能访问



## 16，regexMachers

> 正则表达式匹配，匹配路径
>
> .regexMachers() // 可以指定请求方式





## 17，antMatchers

> .antMatchers(...)
>
> 参数时**ant**表达式，它时url的规则

**ant 表达式**

是url规则

如：

* `?`：匹配一个字符
* `*`： 匹配0个或多个字符
* `**`: 匹配0个或多个目录

在实际开发中经常需要方向所有的额静态资源，下面演示方向js文件夹下的所有脚本资源

```
.antMatchers("/js/**", "/css/**").permitAll()

.antMatchers("/js/**/*.js).permitAll()
```





## 18, Spring Security Oauth2

![image-20211014214627222](assert/image-20211014214627222.png)

* Authorize Endpoint: 授权端点，进行授权
* Token Endpoint: 令牌端点，经过授权拿到对应的token
* Introspection Endpoint：校验端点，交易Token的合法性
* Revocation Endpoint：撤销端点，撤销授权



## 18 角色的判断

前面的是权限的判断，就是在userDetailService中添加到User中权限字符串没有ROLE_开头的

**ROLE_角色**

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private PasswordEncoder passwordEncoder;

    /**
     *查询数据库用户，生成用户信息用户名，密码，权限
     *
     * @param username 用户输入的用户名
     * @return SpringSecurity的User
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 查询数据库，判断用户是否存在
        if (!username.equals("用户名")) {
            throw new UsernameNotFoundException("用户名不存在");
        }
        // 把查询处理的密码进行解析，或者直接放入构造方法中
        String encodePassword = passwordEncoder.encode("密码");
        return new User(username, encodePassword, AuthorityUtils.commaSeparatedStringToAuthorityList("权限1,权限2,ROLE_角色1,ROLE_角色2"));// ROLE_告诉security这个是角色不是权限
    }
}
```

```java
.antMatchers("/xxx").hasRole("角色1");// 这里的角色不需要ROLE_ 框架自动替换，大小写敏感
					.hasAnyRole("角色1", "角色2")
```



## 19，退出登录

**LogoutConfigure.java 定义有**

```java
// 退出的请求url
http.logoutUrl("/logout")
    // 退出成功后的跳转url, 是页面还是请求 post还是get
    .logoutSuceessUrl("/xxx")
```



## 20，自定义403页面

 第一步：自定义一个类 implements AccessDeniedHandler

第二部：在配置类上添加

```java
/**
 * 自定义403处理
 *
 * @author yanggy
 * @date 2021/10/14 22:23
 */
@Component
public class CustomAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AccessDeniedException e) throws IOException, ServletException {
        // 403 处理
        httpServletResponse.setStatus(HttpServletResponse.SC_FORBIDDEN);
        httpServletResponse.setHeader("Content-Type", "application/json;charset=utf-8");
        httpServletResponse.getWriter().println("status:'error', msg: '权限不够，请联系管理员'");
    }
}
```

```java
// 异常处理
http.exceptionHandling()
    .accessDeniedHandler(customAccessDeniedHandler)
```



## 22，_access结合自定义方法实现授权

**.anyRequest.anthendicate()**

```java
/**
 *  自定义权限控制
 */
public interface ICustomAuthenticationService {

    boolean hasPermission(HttpServletRequest request, Authentication authentication);
}
```

```java
/**
 * 自定义权限控制
 *
 * @author yanggy
 * @date 2021/10/14 22:45
 */
@Service
public class CustomAuthenticationServiceImpl implements ICustomAuthenticationService {
    @Override
    public boolean hasPermission(HttpServletRequest request, Authentication authentication) {
        User principal = (User) authentication.getPrincipal();
        // 获取权限
        Collection<GrantedAuthority> authorities = principal.getAuthorities();
        // 权限判断 (GrantedAuthority是接口，要使用实现类判断)， 用户是否有访问当前url的权限,这时User权限字符串是url
        return authorities.contains(new SimpleGrantedAuthority(request.getRequestURI()));
    }
}
```

```java
// 授权认证
http.authorizeRequests()
        // error.html 不需要被认证
        .antMatchers("/error.html").permitAll()
        // login.html不需要被认真
        .antMatchers("/login.html").permitAll()
        .anyRequest().access("@customAuthenticationServiceImpl.hasPermission(request, authentication)")
        // 其他所有请求需要被认真，必须登录之后被访问
        .anyRequest().authenticated();
```

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private PasswordEncoder passwordEncoder;

    /**
     *查询数据库用户，生成用户信息用户名，密码，权限
     *
     * @param username 用户输入的用户名
     * @return SpringSecurity的User
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 查询数据库，判断用户是否存在
        if (!username.equals("用户名")) {
            throw new UsernameNotFoundException("用户名不存在");
        }
        // 把查询处理的密码进行解析，或者直接放入构造方法中
        String encodePassword = passwordEncoder.encode("密码");
        return new User(username, encodePassword,
                AuthorityUtils.commaSeparatedStringToAuthorityList("权限1,权限2, /dosome"));
    }
}
```



## 23，基于注解的访问控制

> 在springsecurity中提供了一些访问控制的注解，这些注解都是默认**不可用**的，需要通过@EnableGlobalMethodSecurity进行开启后使用
>
> 如果设置的条件一些，程序正常运行，如果不允许报500
>
> 这些注解可以卸载Service接口或者方法上也可以写在Controller或者Controller方法上。通常情况下写在控制器方法上。



## 23.1 @Secure("ROLE_xxx") 角色控制

> @Secure("ROLE_XXX") 基于角色的访问控制,大小写敏感

```java
@Secure("ROLE_abc")
@RequestMappering("/xxx")
public R dosome() {
    reture "dosome";
}
```



## 24，RememberMe记住我功能实现

> spring security中RememberMe为记住我的功能，用户只需要在登录时添加remember-me复选框，取值为true。spring security会把用户信息存储到数据库中，以后就可以不登录访问了。底层依赖jdbc
>
> 默认失效时间两周

第一 导入依赖：

* mybatis 依赖
* MySQL依赖

第二，代码配置

```java
// remember-me记住我
http.rememberMe()
    // 失效时间，默认两周
    .tokenValiditySeconds(60*60*24*3)
    //.rememberMeParameter("remember-me")
    // 自定义登录逻辑
    .userDetailsService(userDetailsService)
    // 持久层对象
    .tokenRepository(persistentTokenRepository);


/****************************************************/


 /**
 * security 持久层对象
 *
 * @return
 */
@Bean
public PersistentTokenRepository getPersistentTokenRepository() {
    final JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
    jdbcTokenRepository.setDataSource(dataSource);
    // TODO 自动创建表。第一次启动的时候需要，第二次启动需要注释掉，否则报错
    jdbcTokenRepository.setCreateTableOnStartup(true);
    return jdbcTokenRepository;
}
```



## 29，退出登录解读源码

```java
// 退出登录
http.logout()
    // 和登录成功的handler一样，通常不设置
    //.addLogoutHandler()
    // 前台退出请求的url
    .logoutUrl("/logout")
    // 退出登录跳转url
    .logoutSuccessUrl("/login.html");
```



## 30,  常见的认证机制

**1，HTTP Basci Auth**

> 简单来说就是每次请求api时都提供用户的username和password。有暴露用户密码给第三方客服端的风险，如在使用第三方服务请求自己的资源时



**2，cookie Auth**

## 配置

```java

import com.ruoty.framework.security.handler.CustomAccessDeniedHandler;
import com.ruoty.framework.web.service.UserDetailsServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl;
import org.springframework.security.web.authentication.rememberme.PersistentTokenRepository;

import javax.sql.DataSource;

/**
 * Security 配置类
 *
 * @author yanggy
 * @date 2021/10/12 23:49
 */
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private CustomAccessDeniedHandler customAccessDeniedHandler;

    @Autowired
    private UserDetailsServiceImpl userDetailsService;

    @Autowired
    private DataSource dataSource;

    @Autowired
    private PersistentTokenRepository persistentTokenRepository;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 表单提交
        http.formLogin()
                // 前台表达登录地址，当发现/login表达提交地址后，去执行UserDetailServiceImpl
                .loginProcessingUrl("/login")
                // UsernamePasswordAuthenticationFilter 过滤器进行认证
                // 自定义用户参数名
                .usernameParameter("username")
                // 自定义密码参数名
                .passwordParameter("password")
                // 自定义登录页面
                .loginPage("/login.html")
                // 登录成功后的跳转请求路径, Post请求
                .successForwardUrl("/toSuccessLogin")
                // 登录失败后跳转请求路径，Post请求
                .failureForwardUrl("/toLoginError");

        // 授权认证
        http.authorizeRequests()
                // error.html 不需要被认证
                .antMatchers("/error.html").permitAll()
                // login.html不需要被认真
                .antMatchers("/login.html").permitAll()
                // 自定义权限控制
                //.anyRequest().access("@customAuthenticationServiceImpl.hasPermission(request, authentication)")
                // 其他所有请求需要被认真，必须登录之后被访问
                .anyRequest().authenticated();

        // remember-me记住我
        http.rememberMe()
                // 失效时间，默认两周
                .tokenValiditySeconds(60*60*24*3)
                //.rememberMeParameter("remember-me")
                // 自定义登录逻辑
                .userDetailsService(userDetailsService)
                // 持久层对象
                .tokenRepository(persistentTokenRepository);

        // 自定义异常处理 403
        http.exceptionHandling()
                .accessDeniedHandler(customAccessDeniedHandler);

        // 退出登录
        http.logout()
                // 和登录成功的handler一样，通常不设置
                //.addLogoutHandler()
                // 前台退出请求的url
                .logoutUrl("/logout")
                // 退出登录跳转url
                .logoutSuccessUrl("/login.html");

        // 关闭csrf（防火墙）
        http.csrf().disable();
    }

    /**
     * 密码解码器
     *
     * @return
     */
    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * security 持久层对象
     *
     * @return
     */
    @Bean
    public PersistentTokenRepository getPersistentTokenRepository() {
        final JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        jdbcTokenRepository.setDataSource(dataSource);
        // TODO 自动创建表。第一次启动的时候需要，第二次启动需要注释掉，否则报错
        jdbcTokenRepository.setCreateTableOnStartup(true);
        return jdbcTokenRepository;
    }
}

```

