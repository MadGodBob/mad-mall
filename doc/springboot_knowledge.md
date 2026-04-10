## IDEA部署下载的依赖

```
Spring Web
Lombok
MySQL Driver
Spring Boot DevTools
```

springboot 4.0

## 加入mybatis-plus

```
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot4-starter</artifactId>
    <version>3.5.15</version>
</dependency>
```

在application.properties，加入

```
server.port=8090
spring.datasource.url=jdbc:mysql://mysql6.sqlpub.com:3311/madgod?useUnicode=true&useSSL=false&characterEncoding=utf8&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
spring.datasource.username=madgod
spring.datasource.password=MrfM9w2SJuxKUvlC
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

## 测试代码

在Application.java同根文件夹下创建包：entity	mapper	service	service/impl

创建User类	UserMapper接口	UserService接口	UserServiceImpl类

1. userMapper.selectList方法是继承于**MyBatis Plus**的库，自带数据库查询函数

```
@Data
public class User {
	@TableId(type = IdType.AUTO)  // 使用数据库自增策略
    private Integer id;
    private String no;
    private String name;
    private String password;
    private Integer sex;
    private Integer roleId;
    private String phone;
    @TableField("isValid")
    private String isValid;
}
```

```
@Mapper
public interface UserMapper extends BaseMapper<User> {
}
```

```
public interface UserService extends IService<User> {
    List<User> list();
}
```

```
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    @Resource
    private UserMapper userMapper;

    @Override
    public List<User> list() {
        return userMapper.selectList(null);
    }
}
```

##### 在controller中进行测试

```
@GetMapping("/list")
public List<User> list(){
    return userService.list();
}
```

## 使用代码编辑器生成代码

三种依赖分别是为了代码生成器、生成器模板、swagger调试

```
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.15</version>
</dependency>
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.32</version>
</dependency>
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.2.0</version>
</dependency>
```

新建common包，添加CodeGenerator.java

```
package com.madgod.library.common;

import org.apache.ibatis.annotations.Mapper;
import com.baomidou.mybatisplus.generator.FastAutoGenerator;
import com.baomidou.mybatisplus.generator.config.OutputFile;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;
import com.baomidou.mybatisplus.generator.model.ClassAnnotationAttributes;

import java.io.IOException;
import java.io.InputStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.util.Collections;
import java.util.Properties;

public class CodeGenerator {
    /*
        参数配置 表名 作者 工作目录 父类包名称 数据库url 数据库账号 数据库密码
    */
    public static String tableName = "user";
    public static String author = "madgod";
    public static String workspaceParent = "com.madgod.library";
    public static String MySQL_url;
    public static String usename;
    public static String password;

    private static Path resolveModuleRoot() {
        Path current = Paths.get(System.getProperty("user.dir")).toAbsolutePath().normalize();

        if (Files.exists(current.resolve(Paths.get("src", "main", "java")))) {
            return current;
        }

        Path springbootModule = current.resolve("springboot");
        if (Files.exists(springbootModule.resolve(Paths.get("src", "main", "java")))) {
            return springbootModule;
        }

        return current;
    }

    private static String pickProperty(Properties properties, String key) {
        String value = properties.getProperty(key);
        if (value == null || value.trim().isEmpty()) {
            return null;
        }
        return value.trim();
    }

    private static Properties loadDatasourceFromApplicationProperties(Path moduleRoot) {
        Properties properties = new Properties();

        // Prefer classpath resource; fallback to module file for direct run scenarios.
        try (InputStream classpathInput = CodeGenerator.class.getClassLoader()
                .getResourceAsStream("application.properties")) {
            if (classpathInput != null) {
                properties.load(classpathInput);
            }
        } catch (IOException ex) {
            throw new IllegalStateException("Failed to read classpath application.properties", ex);
        }

        Path propertiesPath = moduleRoot.resolve(Paths.get("src", "main", "resources", "application.properties"));
        if (properties.isEmpty() && Files.exists(propertiesPath)) {
            try (InputStream fileInput = Files.newInputStream(propertiesPath)) {
                properties.load(fileInput);
            } catch (IOException ex) {
                throw new IllegalStateException("Failed to read application.properties at: " + propertiesPath, ex);
            }
        }

        return properties;
    }

    public static void main(String[] args) {
        Path moduleRoot = resolveModuleRoot();
        Properties properties = loadDatasourceFromApplicationProperties(moduleRoot);
        MySQL_url = pickProperty(properties, "spring.datasource.url");
        usename = pickProperty(properties, "spring.datasource.username");
        password = pickProperty(properties, "spring.datasource.password");
        String javaOutputDir = moduleRoot.resolve(Paths.get("src", "main", "java")).toString();
        String xmlOutputDir = moduleRoot.resolve(Paths.get("src", "main", "resources", "mapper")).toString();

        FastAutoGenerator.create(MySQL_url, usename, password)
                .globalConfig(builder ->
                        builder.author(author)
                                .disableOpenDir()
                                .outputDir(javaOutputDir)
                )
                .packageConfig(builder ->
                        builder.pathInfo(Collections.singletonMap(OutputFile.xml, xmlOutputDir))
                                .parent(workspaceParent)
                                .entity("entity")
                                .mapper("mapper")
                                .service("service")
                                .serviceImpl("service.impl")
                )
                .strategyConfig(builder ->
                        builder.addInclude(tableName)
                                .enableSkipView()
                                .entityBuilder().enableLombok(new ClassAnnotationAttributes("@Data","lombok.Data"))
                                .mapperBuilder().mapperAnnotation(Mapper.class)
                                .controllerBuilder()
                                .mapperBuilder()
                                .mapperXmlTemplate("/templates/simple-mapper.xml")
                )
                .templateEngine(new FreemarkerTemplateEngine())
                .execute();
    }
}
```

在templates文件夹下新建xml的模板文件simple-mapper.xml.ftl

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="${package.Mapper}.${table.mapperName}">

    <!-- 基础结果映射 -->
    <resultMap id="BaseResultMap" type="${package.Entity}.${entity}">
        <#list table.fields as field>
            <#if field.keyFlag>
                <id column="${field.name}" property="${field.propertyName}" />
            <#else>
                <result column="${field.name}" property="${field.propertyName}" />
            </#if>
        </#list>
    </resultMap>

    <!-- 基础字段列表 -->
    <sql id="Base_Column_List">
        <#list table.fields as field>
            ${field.name}<#if field_has_next>, </#if>
        </#list>
    </sql>

</mapper>
```

## swagger调试

```
http://localhost:8090/swagger-ui/index.html
```

## 模糊匹配

```
@PostMapping("/listFuzzyByName")
public List<User> listFuzzyByName(@RequestBody User user){
    LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper();
    lambdaQueryWrapper.like(User::getName, user.getName());
    return userService.list(lambdaQueryWrapper);
}
```

## 分页

#### 使用方法：Mybatis的分页拦截器。此外还有1.编写分页mapper方法 2.自定义xml文件中的SQL语言

在Mybatis-plus 3.5.9后分页功能需要单独导入依赖

```
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-jsqlparser</artifactId>
    <version>3.5.15</version>
</dependency>
```

添加MybatisPlusConfig类

```
@Configuration
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

## 返回前端数据的封装

创建Result类，返回的数据包含HTTP状态码 提示信息 返回数据长度 返回数据

```
package com.madgod.library.common;

import lombok.Data;

@Data
public class Result<T> {
    private Integer code;
    private String message;
    private T data;

    // 构造函数
    private Result(Integer code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }

    // 成功 - 无数据
    public static <T> Result<T> success() {
        return new Result<>(200, "success", null);
    }
    // 成功 - 有数据
    public static <T> Result<T> success(T data) {
        return new Result<>(200, "success", data);
    }
    // 成功 - 自定义消息
    public static <T> Result<T> success(String message, T data) {
        return new Result<>(200, message, data);
    }

    // 失败 - 默认消息
    public static <T> Result<T> error() {
        return new Result<>(500, "error", null);
    }
    // 失败 - 自定义消息
    public static <T> Result<T> error(String message) {
        return new Result<>(500, message, null);
    }
    // 失败 - 自定义状态码和消息
    public static <T> Result<T> error(Integer code, String message) {
        return new Result<>(code, message, null);
    }
}


```

## 登录鉴权

导入JWT依赖

```
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.14.0</version>
</dependency>
```

新建工具

```
package com.madgod.library.utils;

import com.madgod.library.entity.User;
import com.madgod.library.service.IUserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;
import jakarta.servlet.http.HttpServletRequest;
import java.util.Calendar;

@Component
public class TokenUtils {
    @Autowired
    private IUserService userService;

    /**
     * 生成token
     * @param user
     * @return
     */
    public String genToken(User user) {
        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.HOUR, 2);
        return JWT.create()
                .withExpiresAt(calendar.getTime())
                .withAudience(user.getId().toString())
                .sign(Algorithm.HMAC256(user.getPassword()));
    }

    /**
     * 获取token中的用户信息
     * @return
     */
    public User getUser(String token) {
        try {
            if(token.isEmpty()){
                HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
                token = request.getHeader("token");
            }
            String aud = JWT.decode(token).getAudience().get(0);
            Integer userId = Integer.valueOf(aud);
            User user = userService.getById(userId);
            JWT.require(Algorithm.HMAC256(user.getPassword())).build().verify(token);
            return user;
        } catch (Exception e) {
            return null;
        }
    }
}
```

common新建拦截器,包含三个文件

```
package com.madgod.library.common;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.madgod.library.entity.User;
import com.madgod.library.utils.TokenUtils;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;

import java.io.IOException;
import java.nio.charset.StandardCharsets;

@Component
public class JwtInterceptor implements HandlerInterceptor {

    private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();

    private final TokenUtils tokenUtils;

    public JwtInterceptor(TokenUtils tokenUtils) {
        this.tokenUtils = tokenUtils;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 静态资源、Swagger、非controller方法 等直接放行
        if (!(handler instanceof HandlerMethod handlerMethod)) {
            return true;
        }

        String token = resolveToken(request);
        if (!StringUtils.hasText(token)) {
            writeUnauthorized(response, "未登录或token缺失");
            return false;
        }

        User user = tokenUtils.getUser(token);
        if (user == null) {
            writeUnauthorized(response, "token无效或已过期");
            return false;
        }

        // 给方法权限
        Privilege privilege = getPrivilege(handlerMethod);
        Integer role = user.getRole();
        if (privilege != null && (role == null || role < privilege.value())) {
            writeForbidden(response, "权限不足");
            return false;
        }

        request.setAttribute("currentUser", user);
        return true;
    }

    private Privilege getPrivilege(HandlerMethod handlerMethod) {
        // 寻找方法上的Privilege注解
        Privilege methodAnnotation = handlerMethod.getMethodAnnotation(Privilege.class);
        if (methodAnnotation != null) {
            return methodAnnotation;
        }
        return handlerMethod.getBeanType().getAnnotation(Privilege.class);
    }

    private String resolveToken(HttpServletRequest request) {
        // 获取相应token
        String authorization = request.getHeader("Authorization");
        if (StringUtils.hasText(authorization) && authorization.startsWith("Bearer ")) {
            return authorization.substring(7).trim();
        }
        return request.getHeader("token");
    }

    private void writeUnauthorized(HttpServletResponse response, String message) throws IOException {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setCharacterEncoding(StandardCharsets.UTF_8.name());
        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().write(OBJECT_MAPPER.writeValueAsString(Result.error(401, message)));
    }

    private void writeForbidden(HttpServletResponse response, String message) throws IOException {
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        response.setCharacterEncoding(StandardCharsets.UTF_8.name());
        response.setContentType("application/json;charset=UTF-8");
        response.getWriter().write(OBJECT_MAPPER.writeValueAsString(Result.error(403, message)));
    }
}
```

```
package com.madgod.library.common;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {
    // 注册拦截器

    private final JwtInterceptor jwtInterceptor;

    public WebConfig(JwtInterceptor jwtInterceptor) {
        this.jwtInterceptor = jwtInterceptor;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(jwtInterceptor)
                .addPathPatterns("/**") // 拦截所有请求
                .excludePathPatterns(   // 放行
                        "/user/register",
                        "/user/login",
                        "/error",
                        "/v3/api-docs/**",
                        "/swagger-ui/**",
                        "/swagger-ui.html"
                );
    }
}
```

```
package com.madgod.library.common;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 基于用户 role 的最小权限要求，role 越大权限越高。
 */
@Documented
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Privilege {
    int value() default 1;
}
```

*额外: 在swagger中添加token选项,在common新建

```
package com.madgod.library.common;

import io.swagger.v3.oas.models.Components;
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.security.SecurityRequirement;
import io.swagger.v3.oas.models.security.SecurityScheme;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        String schemeName = "bearerAuth";
        return new OpenAPI()
                .components(new Components().addSecuritySchemes(schemeName,
                        new SecurityScheme()
                                .name("Authorization")
                                .type(SecurityScheme.Type.HTTP)
                                .scheme("bearer")
                                .bearerFormat("JWT")))
                .addSecurityItem(new SecurityRequirement().addList(schemeName));
    }
}
```

