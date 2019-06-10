---
title: Spring Boot Security下的注册登录
date: 2017-12-24 18:22:03
categories: tech
tags: 
- spring
- java
---

最近想用 JAVA 做一个 Web 应用。想爽一下这个目前”最流行”的语言。于是选择了 Spring Boot 作为项目框架做开发。感觉到Spring 确实是很成熟的框架了。

官方英文文档的阅读让我想去海底龙宫种一棵树。

[Spring Boot Security Application](https://bartcode.co.uk/2014/12/spring-boot-security-application) 这个博主这篇14年文章给了我很多帮助。网上有翻译。但是那个是机器翻的。我还以为是屎呢。

用 Spring Security 可以比较方便地开发应用中的几个问题: 

- 用户角色有用户，管理员。两者权限不一致
- 非管理员用户可以查看自己的信息，但不能窥视其他用户
- 定制表单登录和CSRF保护
- “记住我”验证懒惰

## 准备
- 从[start.spring.io](https://start.spring.io/)官网下载初始化应用。我下的是 gradle 的 spring boot 1.5.9 版本
- [Securing a Web Application](https://spring.io/guides/gs/securing-web/) 看一下官网上的入门 guide
- 数据库用的是 mysql
- gradle配置: 
```gradle
group 'com.pwxc'
version '1.0-SNAPSHOT'

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.9.RELEASE")
    }
}


apply plugin: 'java'
apply plugin: 'war'
apply plugin: 'idea'
apply plugin: 'org.springframework.boot'

sourceCompatibility = 1.8
targetCompatibility = 1.8

jar {
    baseName = 'gs-serving-web-content'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

dependencies {
    compile("org.springframework.boot:spring-boot-starter-thymeleaf")
    compile("org.springframework.boot:spring-boot-devtools")
    testCompile("junit:junit")
    testCompile("org.springframework.boot:spring-boot-starter-test")
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.hsqldb:hsqldb")
    testCompile("org.springframework.security:spring-security-test")
    compile("org.springframework.boot:spring-boot-starter-security")

    // JPA Data (We are going to use Repositories, Entities, Hibernate, etc...)
    compile("org.springframework.boot:spring-boot-starter-data-jpa")
    compile("com.h2database:h2")
    // Use MySQL Connector-J
    compile("mysql:mysql-connector-java")
}
```
## 域模型
先建一个 User 的实体, Role 是一个枚举类，枚举 USER,ADMIN。
```java
@Entity // This tells Hibernate to make a table out of this class
public class User {
    @Id
    @GeneratedValue(generator="system-uuid", strategy= GenerationType.AUTO)
    @GenericGenerator(name = "system-uuid", strategy = "uuid")
    @Column(name="id", nullable = false, updatable = false)
    private String id;

    @Column(name="name", nullable = false)
    private String name;

    @Column(name="email", nullable = false, unique = true)
    private String email;

    @Column(name="phone")
    private String phone;

    @Column(name="password", nullable = false)
    private String password;

    @Column(name="role", nullable = false)
    @Enumerated(EnumType.STRING)
    private Role role;

    @Column(name="createdAt", nullable = false)
    private Long createdAt;

    @Column(name="updatedAt", nullable = false)
    private Long updatedAt;
    ...
}

```

同时再写一个 data transfer object (DTO) 作为 Web 层和服务层的中介。既可以不让 User 泄露到 Web 层又方便我们做表单的验证。

```java
public class UserCreateForm {

    @NotEmpty(message = "邮箱不能为空")
    private String email;

    @NotEmpty(message = "名字不能为空")
    private String name;

    @NotEmpty(message = "密码不能为空")
    private String password;

    @NotEmpty(message = "重复密码不能为空")
    private String passwordRepeated;

    @NotNull
    private Role role = Role.USER;
    ...
}

```

@NotEmpty 和 @NotNull 的区别在与前者会对空字符串或集合会返回 false, 后者只会验证 Null。

## 服务层
UserService 接口
```java
public interface UserService {

    Optional<User> getUserById(String id);

    Optional<User> getUserByEmail(String email);

    Collection<User> getAllUsers();

    User create(UserCreateForm form);
}    
```

UserService 接口的实现
```java
@Service
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;

    @Autowired
    public UserServiceImpl(UserRepository IUserRepository) {
        this.userRepository = IUserRepository;
    }

    @Override
    public Optional<User> getUserById(String id) {
        return userRepository.findOneById(id);
    }

    @Override
    public Optional<User> getUserByEmail(String email) {
        return userRepository.findOneByEmail(email);
    }

    @Override
    public Collection<User> getAllUsers() {
        return userRepository.findAll(new Sort("email"));
    }

    @Override
    public User create(UserCreateForm form) {
        User user = new User();
        Date date = new Date();
        Long timestamp = Long.valueOf(date.getTime()/1000);
        user.setEmail(form.getEmail());
        user.setName(form.getName());
        user.setPassword(new BCryptPasswordEncoder().encode(form.getPassword()));
        user.setRole(form.getRole());
        user.setCreatedAt(timestamp);
        user.setUpdatedAt(timestamp);

        return userRepository.save(user);
    }
}   
```

dao 层 UserRepository 直接用Jpa，实现了基本所有的简单数据库操作
```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findOneByEmail(String email);
    Optional<User> findOneById(String id);
}
```

## Web层
首页 “/“ 随便写一下就好了，我是写了几个到”/register”, “/login”的链接的。我把 login 和 register 都放在一个Controller 里。这是我的 LoginAndRegisterController
```java
@Controller
public class LoginAndRegisterController {

    private static final Logger LOGGER = LoggerFactory.getLogger(LoginAndRegisterController.class);
    private final UserService userService;
    private final UserCreateFormValidator userCreateFormValidator;

    @Autowired
    public LoginAndRegisterController(UserService userService, UserCreateFormValidator userCreateFormValidator) {
        this.userService = userService;
        this.userCreateFormValidator = userCreateFormValidator;
    }

    @InitBinder("form")
    public void initBinder(WebDataBinder binder) {
        binder.addValidators(userCreateFormValidator);
    }

    @RequestMapping(value = "/register", method = RequestMethod.GET)
    public String getRegisterPage(Model model) {
        model.addAttribute("form", new UserCreateForm());
        return "register";
    }

    @RequestMapping(value = "/register", method = RequestMethod.POST)
    public String handleUserCreateForm(Model model, @Valid @ModelAttribute("form") UserCreateForm form, BindingResult bindingResult) {
        LOGGER.debug("用户提交的表单: {}, 验证结果: {}", form, bindingResult);
        if(bindingResult.hasErrors()) {
            model.addAttribute("error", "出错啦");
            return "register";
        }
        try{
            userService.create(form);
        } catch (DataIntegrityViolationException e) {
            LOGGER.warn("保存时出错，邮箱已经注册了！", e);
            bindingResult.reject("email.exists", "Email already exists");
            return "register";
        }
        return "redirect:/hello";
    }

    @RequestMapping(value = "/login", method = RequestMethod.GET)
    public String getLoginPage(Model model, @RequestParam Optional<String> error) {
        model.addAttribute("error", error);
        return "login";
    }
}
```

这里还有比较重要的注册表单的验证类 UserCreateFormValidator, 在使用的时候还要用 @InitBinder 绑定一下。
```java
@Component
public class UserCreateFormValidator implements Validator {

    private final UserService userService;

    @Autowired
    public UserCreateFormValidator(UserService userService) {
        this.userService = userService;
    }

    @Override
    public boolean supports(Class<?> clazz) {
        return clazz.equals(UserCreateForm.class);
    }

    @Override
    public void validate(Object target, Errors errors) {
        UserCreateForm form = (UserCreateForm)target;
        validatePasswords(errors, form);
        validateEmail(errors, form);
    }

    private void validatePasswords(Errors errors, UserCreateForm form) {
        if (!form.getPassword().equals(form.getPasswordRepeated())) {
            errors.reject("password.no_match", "两次密码不一致!");
        }
    }

    private void validateEmail(Errors errors, UserCreateForm form) {
        if (userService.getUserByEmail(form.getEmail()).isPresent()) {
            errors.rejectValue("email", "email.exists", "该邮箱已被注册!");
        }
    }
}
```
这样注册功能其实是已经完成了。有注册就有登陆，登录之后问题有点复杂了，要保持登录状态，remember-me, 基于URL的安全保护和基于方法级的安全保护。

## CSRF
在现在版本里在 Sercurity 配置里开启了 @EnableWebSecurity，表单里会自动填充 csrf 如果没有的话，自己加一个就好了，我用thymeleaf 模板引擎里，加一句就好了。
```html
<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
```

## 认证配置
使用Sercurity Boot 里必须要配置的，不然的话，每次打开网页他就会一直叫你验证，可以说是，超级不友好了。SecurityConfig类: 
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailsService userDetailsService;

    /**定义安全策略*/
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                    //这里其实就是基于URL的安全配置
                    .antMatchers("/static/css/**", "/static/js/**", "/static/images/**").permitAll()  //静态文件不认证
                    .antMatchers("/", "/register").permitAll()  //首页和注册页面不认证
                    .antMatchers("/admin/**").hasAuthority("ADMIN") //只有管理员才能访问
                    .anyRequest().fullyAuthenticated()  //别的全都要认证
                .and()
                    .formLogin()    //通过表单验证
                    .loginPage("/login")    //未认证用户访问需要认证界面自动跳转到 "/login"
                    .failureUrl("/login?error")
                    .usernameParameter("email")
                    .permitAll()
                .and()
                    .logout()
                    .logoutUrl("/logout")
                    .deleteCookies("remember-me")   
                    .logoutSuccessUrl("/")
                    .permitAll()
                .and()
                    .rememberMe()
                    .tokenValiditySeconds(1209600); //remember-me的time-out默认我试了一下是14天。
    }

    /**定义认证用户信息获取来源，密码校验规则等*/
    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth
                .userDetailsService(userDetailsService)
                .passwordEncoder(new BCryptPasswordEncoder());

        //jdbcAuthentication从数据库中获取，但是默认是以security提供的表结构
        //usersByUsernameQuery 指定查询用户SQL
        //authoritiesByUsernameQuery 指定查询权限SQL
        //auth.jdbcAuthentication().dataSource(dataSource).usersByUsernameQuery(query).authoritiesByUsernameQuery(query);

        //注入userDetailsService，需要实现userDetailsService接口
        //auth.userDetailsService(userDetailsService);
    }
}
```

这个配置一开始让人感觉很头大，我把难受的地方都加了注释了。UserDetailsService这个接口是Spring Security用来验证当前用户，是系统给的。需要实现一下这个接口:

```java
@Service
public class CurrentUserDetailsService implements UserDetailsService {

    private static final Logger LOGGER = LoggerFactory.getLogger(CurrentUserDetailsService.class);
    private final UserService userService;

    @Autowired
    public CurrentUserDetailsService(UserService userService) {
        this.userService = userService;
    }

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        LOGGER.info("Authenticating user with email={}", email.replaceFirst("@.*", "@***"));
        User user = userService.getUserByEmail(email).orElseThrow(() -> new UsernameNotFoundException(String.format("User with email=%s was not found", email)));
        return new CurrentUser(user);
    }
}
```

UserDetails 是 Spring 用来记录当前用户，就是用户登录完之后，系统里保存就是当前这个用户的 UserDetails，所以可以在不同的用户端显示不同的内容。这个 UserDetails 对每个应用是不一样的，因为每个应用有不同的业务逻辑，所以继承一波UserDetails，写一个属于自己的 UserDetails。就是 CurrentUser: 
```java
public class CurrentUser extends org.springframework.security.core.userdetails.User {

    private User user;

    public CurrentUser(User user) {
        super(user.getEmail(), user.getPassword(), AuthorityUtils.createAuthorityList(user.getRole().toString()));
        this.user = user;
    }

    public User getUser() {
        return user;
    }

    public String getId() {
        return user.getId();
    }

    public Role getRole() {
        return user.getRole();
    }
}
```

如果访问”/user/{id}”, A 用户访问了 B 用户的 id，此时应该需要拦截，但是如果在 URL 是不方便设置的，可以用方法级的拦截。调用 getprincipal()就可得到当前用户的实例。
```java
@PreAuthorize("@currentUserServiceImpl.canAccessUser(principal, #id)")
@RequestMapping("/user/{id}")
public String getUserPage(@PathVariable Long id) {
    //
}
```

## 在视图中得到当前用户
编写一个用 @ControllerAdvice 和 @ModelAttribute 注释的类，就不必每次都去获取这个实例了。
```java
@ControllerAdvice
public class CurrentUserControllerAdvice {

    @ModelAttribute("currentUser")
    public CurrentUser getCurrentUser(Authentication authentication) {
        return (authentication == null) ? null : (CurrentUser) authentication.getPrincipal();
    }
}
```

[代码](https://github.com/pwxcoo/ProvokeQuestion/tree/974dda0bf9bda3b66ab16f8338209f95e9059459)