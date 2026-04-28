#### **1. 静态资源加载路径**

- **问题**：`addResourceHandlers` 方法配置的静态资源路径（如 `classpath:/META-INF/resources/`）在项目中找不到。
- 解答：
  - 这些路径属于依赖中的资源文件，通常存在于 Maven 依赖的 jar 包内。
  - 可以在 IDE 的 `External Libraries` 中找到对应路径。

------

#### **2. Spring 报错：`No mapping for GET /favicon.ico`**

- **原因**：浏览器在请求页面时会默认尝试加载 `/favicon.ico`。
- 解决方案：
  - 提供一个 `favicon.ico` 文件，并通过静态资源路径或 Controller 配置返回。
  - 或者忽略 `/favicon.ico` 请求，通过 Web 配置进行处理。

------

#### **3. DTO、VO、Entity 的区别**

- **DTO**（数据传输对象）：用于服务之间的数据传输，通常不包含业务逻辑。
- **VO**（视图对象）：用于封装展示层的数据，直接用于页面展示。
- **Entity**（实体类）：与数据库表一一对应，包含数据结构和持久化相关内容。

现实类比：

- **DTO**：快递盒，用于运输。
- **VO**：展示柜，用于展示商品。
- **Entity**：仓库中的商品，是最真实的物品存储。

------

#### **4. Query 参数直接封装为对象**

- **现象**：Spring MVC 的方法参数可以直接接收 Query 参数并封装成对象，而不需要 `@RequestParam`。
- **原因**：Spring 的数据绑定机制（通过 `WebDataBinder`）会自动将请求参数匹配到对象属性，前提是参数名和属性名一致。

**示例**：

```java
@GetMapping("/employee")
public String getEmployee(Employee employee) {
    System.out.println(employee); // 自动绑定
    return "success";
}
```

------

#### **5. 端口号：80 vs 8080**

- **80 端口**：默认的 HTTP 端口，浏览器访问时可省略。
- **8080 端口**：常用的开发或测试端口，需明确指定。
- **区别**：80 端口需要特权，非 root 用户无法直接绑定。

------

#### **6. 修改 TODO 的颜色**

- **问题**：希望修改 IDEA 中 `TODO` 的显示颜色。
- 解决方案：
  - 打开 **File → Settings → Editor → Color Scheme → General**。
  - 找到 `Code → TODO Default`，选择颜色并保存。

------

#### **7. MD5 加密**

- **问题**：字符串 `123456` 的 MD5 加密值是什么？
- **答案**：`e10adc3949ba59abbe56e057f20f883e`。
- **特点**：MD5 是不可逆的哈希算法，用于验证数据完整性。

------

#### **8. JVM 类加载的逻辑**

- **问题**：类中包含 `static` 成员时，JVM 会提前加载该类吗？
- 结论：
  - 如果类中有任何一个 `static` 成员（变量或方法），JVM 会在首次使用时加载该类。
  - 这包括初始化静态变量、执行静态代码块等。

------

#### **9. MyBatis 的 `<where>` 标签**

- **问题**：`<where>` 标签中的条件是否需要手动加 `and`？

- 解答：

  - 不需要手动加 `and`，MyBatis 会根据条件智能地拼接 SQL。

  - 例如：

    ```xml
    <where>
        <if test="name != null">
            name like concat('%', #{name}, '%')
        </if>
    </where>
    ```

------

#### **10. ThreadLocal 的使用场景**

- **问题**：在 AOP 的 `around` 方法中获取 JWT 并解析用户 ID，是否可以直接传递到 Service 层？
- 建议：
  - 如果直接在 Service 层解析 JWT，不建议使用 `HttpServletRequest` 或 `HttpServletResponse`，因为 Service 层应与 HTTP 解耦。
  - 推荐方案：在拦截器中解析 JWT，并将用户信息存入 `ThreadLocal`，供后续业务逻辑使用。

------

#### **11. 类型转换：`String` 与 `Object`**

- **问题**：为什么 `Object` 类型可以直接转换为 `String`？
- 原因：
  - Java 的 `Object` 类有一个 `toString()` 方法，默认会返回对象的字符串表示。
  - 因此，`Object.toString()` 是安全的，但实际内容取决于对象的实现。

------

#### **12. Spring 日志中的连接池警告**

- **现象**：`discard long time none received connection` 警告。
- **原因**：Druid 连接池检测到长时间未使用的连接，并将其丢弃。
- 解决方案：
  - 调整连接池配置，例如 `minIdle` 和 `maxIdle`，确保连接池大小适合业务需求。

------

#### **13. 时间单位转换**

- **问题**：`7200000` 毫秒是多少分钟？
- **答案**：`7200000 / 1000 / 60 = 120` 分钟，即 2 小时。

------

#### **14. 泛型与成员访问**

- **问题**：`T result = (T)e.value;` 这种写法中的 `e.value` 是什么？

- 解答：

  - `e.value` 表示访问对象 `e` 的字段 `value`。

  - 这是典型的字段访问操作，`value` 是 `e` 的一个成员变量。

  - 示例：

    ```java
    class Entry<T> {
        T value;
    }
    
    Entry<String> e = new Entry<>();
    String result = e.value;
    ```