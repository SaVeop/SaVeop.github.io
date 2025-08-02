### **1. -Dserver.port 参数问题**

**问题：** 在 IDEA 的运行配置中，如何指定端口参数？

1. 在运行配置的 "程序参数 (Program Arguments)" 中指定：

   ```reStructuredText
   --server.port=8083
   ```

2. 在运行配置的 "虚拟机选项 (VM Options)" 中使用：

   ```reStructuredText
   -Dserver.port=8083
   ```

**总结：**

- `--server.port` 是 Spring Boot 的参数，通过程序参数指定。
- `-Dserver.port` 是 JVM 的系统属性，通过虚拟机选项指定。

------

### **2. Spring 的 RestTemplate 的 exchange 方法**

**问题：** `exchange` 方法是什么？

1. `exchange` 是 Spring 的 **RestTemplate** 提供的一个高级方法，用于发起 HTTP 请求。

   - 它支持自定义 HTTP 方法（如 GET、POST、PUT、DELETE）。
   - 允许通过 `HttpEntity` 设置请求头、请求体。
   - 返回 `ResponseEntity`，可以解析响应头、状态码和响应体。

2. 示例代码：

   ```java
   RestTemplate restTemplate = new RestTemplate();
   String url = "https://api.example.com/data";
   HttpHeaders headers = new HttpHeaders();
   headers.set("Authorization", "Bearer token");
   HttpEntity<String> entity = new HttpEntity<>(null, headers);
   ResponseEntity<String> response = restTemplate.exchange(url, HttpMethod.GET, entity, String.class);
   System.out.println(response.getBody());
   ```

------

### **3. IDEA 的快捷键 Ctrl+H**

**功能：**

- **Ctrl + H** 在 IDEA 中用于查看 **类的继承结构**。
- 打开后会显示当前类的所有父类和子类。

------

### **4. AntPath 简介**

**问题：** 什么是 AntPath？

1. **AntPath** 是一种路径匹配的方式，主要在 Spring 的 `RequestMapping` 中用于定义动态 URL 路径。

2. 特点：

   - 支持通配符 `*` 和 `**`。
   - 支持路径参数 `{}`。

3. 示例：

   ```java
   @GetMapping("/user/{id}")
   public String getUser(@PathVariable String id) {
       return "User ID: " + id;
   }
   ```

------

### **5. Spring Boot 项目启动失败**

**问题描述：**

```bash
Parameter 0 of constructor in com.hmall.gateway.filters.AuthGlobalFilter required a bean of type 'com.hmall.gateway.config.AuthProperties' that could not be found.
```

**原因：**

1. Spring 在创建 `AuthGlobalFilter` 时，发现需要一个 `AuthProperties` 的 Bean。
2. 但项目中没有定义 `AuthProperties` 或者未将其标注为 Spring 管理的 Bean。

**解决方法：**

1. 确保 `AuthProperties` 类上添加了注解：

   ```java
   @Component
   @ConfigurationProperties(prefix = "auth")
   public class AuthProperties {
       private String key;
       // Getters and Setters
   }
   ```

2. 在配置类中启用配置绑定：

   ```java
   @EnableConfigurationProperties(AuthProperties.class)
   @Configuration
   public class AppConfig {
   }
   ```

------

### **6. Filter 和 Interceptor 的范围**

**问题：** 两者的范围如何对比，如何记忆？

1. **Filter（过滤器）**：
   - 属于 Servlet 规范，处理的是 HTTP 请求。
   - 范围广泛，可操作请求和响应，甚至可在进入 Spring 之前拦截。
2. **Interceptor（拦截器）**：
   - 属于 Spring 框架，处理的是 Spring MVC 的请求。
   - 只作用于进入 Controller 的请求。

**记忆方法：**

- **过滤器 (Filter)** 是广义的，像一张网过滤所有请求。
- **拦截器 (Interceptor)** 是特定的，像门卫拦截进门的请求。

------

### **7. exchange.mutate() 的含义**

**问题：** `mutate` 方法的作用是什么？

- `exchange.mutate()` 是 Spring WebFlux 中 `ServerWebExchange` 的方法(副本的意思)。
- 用于修改当前请求或响应的属性，例如请求头、响应头、路径等。

**示例：**

```java
exchange.mutate()
       .request(request -> request.header("X-Custom-Header", "value"))
       .build();
```

------

### **8. Spring Factories 与 @Enable 的区别**

**问题：** 为什么有时候需要写 `spring.factories`，有时候使用 `@Enable`？

1. **spring.factories**：
   - 是 Spring Boot 提供的自动装配机制，用于声明自动配置类。
   - 无需用户显式声明，Spring Boot 自动加载。
2. **@Enable**：
   - 提供用户选择性地启用某些功能。
   - 比如 `@EnableScheduling`、`@EnableCaching` 等。

**总结：**

- `spring.factories` 是自动装配，不需要用户干预。
- `@Enable` 是让用户显式控制是否启用某些功能。

------

### **9. 判断项目是否有 Spring MVC**

**方法：**

1. 检查 Maven 或 Gradle 依赖中是否有 `spring-webmvc`。

2. 查看项目中是否有 `@RestController`、`@Controller` 注解。

3. 查看启动日志，是否有类似：

   ```reStructuredText
   Mapped URL path [/example] onto handler
   ```

------

### **10. Spring Boot 与 Spring MVC 的关系**

**问题：** `@SpringBootApplication` 是 Spring MVC 的一部分吗？

- **不是**。`@SpringBootApplication` 是 Spring Boot 的注解，用于标记启动类，自动加载配置。
- Spring MVC 是 Web 开发框架，需要单独引入 `spring-webmvc`。

------

### **11. 依赖的作用范围**

**问题：** `<scope>provided</scope>` 的含义是什么？

- **作用：** 指定依赖在编译和测试阶段可用，运行时不包含，也不会传递给下游模块。
- 常见场景：
  - Servlet API、JSP 等由服务器提供的依赖。

------

### **12. 引用 `spring-boot-starter-web` 是否意味着有 Spring MVC？**

**答案：** 是的。

- `spring-boot-starter-web` 包含了 `spring-webmvc`，自动加载 Spring MVC 所需的所有组件。

------

### 13. JwtProperties不是已经用注解@ConfigurationProperties(prefix = "hm.jwt")成为容器里的bean了吗?为什么在SecurityConfig里面还要再加上@EnableConfigurationProperties(JwtProperties.class)注解,是不是要保证JwtProperties得在它之前生成bean,这样它才可以用?

#### **解释** :

- `@ConfigurationProperties` 和 `@EnableConfigurationProperties `的作用 `@ConfigurationProperties(prefix = "hm.jwt")`：将配置文件中的属性绑定到 `JwtProperties `类的字段上。 

  `@EnableConfigurationProperties(JwtProperties.class)`：确保` JwtProperties` 被注册为` Spring` 容器中的一个` Bean`。 

- **为什么需要` @EnableConfigurationProperties` ?**

- 即使` JwtProperties `使用了 `@ConfigurationProperties `注解，它并不会自动成为` Spring`容器中的 `Bean`。你需要通过  `@EnableConfigurationProperties `或者在某个 `@Configuration` 类中显式声明` JwtProperties` 为` Bean` 来确保它被注册到`Spring `容器中。 

  `@EnableConfigurationProperties` 可以确保 `JwtProperties `在应用启动时就被加载并注册为` Bean`，从而可以在其他配置类或组件中直接注入使用。 

- **顺序问题** 

- `@EnableConfigurationProperties(JwtProperties.class) `并不是为了保证` JwtProperties `在` SecurityConfig `之前生成 `Bean`，而是确保 `JwtProperties` 成为 `Spring` 容器中的` Bean`，这样 `SecurityConfig `才能正确地依赖注入` JwtProperties`。 

- **总结** 

- `@EnableConfigurationProperties(JwtProperties.class)` 是必要的，因为它确保` JwtProperties` 被注册为` Spring `容器中的 `Bean`，从而可以在 `SecurityConfig `中正常使用。 

  这并不是为了控制生成顺序，而是为了确保 `JwtProperties `成为可注入的` Bean`。