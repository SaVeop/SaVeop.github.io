### **1. 关于 `lambdaQuery()` 的参数引用问题**

- **问题描述**： `lambdaQuery().like(userName != null && userName != "", UserQuery::getName, userName)` 中，`UserQuery::getName` 报错，提示无法从 static 上下文引用非 static 方法。
- **原因**：
  - `UserQuery::getName` 是一个实例方法引用，必须在能够获取实例的上下文中使用。
  - 如果无法找到对应实例的上下文，编译器会报错。
- **对比**：
  - **为什么 `User::getUsername` 可以**：
    因为 `lambdaQuery()` 流式 API 会传入每个实例作为上下文，可以调用实例方法。
  - **`UserQuery::getName` 报错**：
    因为编译器认为上下文中无法提供实例。

------

### **2. 关于 MyBatis-Plus 查询结果为空**

- **问题描述**：
  - 使用 `List<User> users = listByIds(ids);`，即使没有查到数据，`users` 是空列表，而不是 `null`。
  - 使用 `User user = getById(id);`，如果查不到数据，则返回 `null`。
- **原因**：
  - `listByIds(ids)` 的行为：
    - 它总是返回一个列表，即使查询不到数据，也会返回一个空列表（`new ArrayList<>()`）。
    - 符合 Java 集合 API 的惯例，避免 `NullPointerException`。
  - `getById(id)` 的行为：
    - 如果查不到数据，直接返回 `null`，表示无此记录。

------

### **3. 集合是否为 `null` 或空**

- **`new ArrayList<>()` 是空集合，不是 `null`**：
  - 空集合：`集合对象不为 null，但没有元素`。
  - `null` 集合：集合对象本身未初始化，无法调用任何方法。
- **集合可能为 `null` 的情况**：
  - 手动赋值为 `null`。
  - 数据库返回的对象未初始化或未正确赋值。

------

### **4. 关于 `throw` 和 `return` 后的代码**

- 结论：
  - `throw` 和 `return` 之后的代码**不会执行**。
  - 它们会中断当前方法的执行，直接返回或抛出异常。

------

### **5. 实例方法引用与流的处理**

- **`User::getId` 的行为**：
  - 它是实例方法引用的一种简化形式。
  - 作用相当于 `user -> user.getId()`。
  - 因为流会为每个元素提供上下文，因此可以直接使用实例方法引用。
- **实例方法引用的三种形式**：
  1. **静态方法引用**：`ClassName::staticMethod`。
  2. **特定对象的实例方法引用**：`instance::method`。
  3. **任意对象的实例方法引用**：`ClassName::method`（如 `User::getId`）。

------

### **6. 按 `List<AddressVO>` 中的 `userId` 分类**

- 实现代码：

  ```java
  Map<Long, List<AddressVO>> groupedByUserId = addressList.stream()
      .collect(Collectors.groupingBy(AddressVO::getUserId));
  ```

- 方法引用的行为：

  - `AddressVO::getUserId` 是任意对象的实例方法引用，应用于流中每个元素。

------

### **7. 枚举类的设计**

- 示例代码：

  ```java
  public enum UserStatus {
      NORMAL(1, "正常"),
      FREEZE(2, "冻结");
  
      private final int value;
      private final String desc;
  
      UserStatus(int value, String desc) {
          this.value = value;
          this.desc = desc;
      }
  }
  ```

- 功能：

  - 枚举定义了一组固定的常量，具有描述和值。
  - 可以通过 `value` 或 `desc` 获取常量对应的信息。
  - 提高代码可读性

------

### **8. 关于 `paginationInnerInterceptor.setMaxLimit(1000)`**

- **问题描述**：

  - `paginationInnerInterceptor.setMaxLimit(1000);` 提示类型不匹配，`setMaxLimit` 方法需要 `Long` 类型。
  - 为什么 `1000`（`int` 类型）不能自动升级为 `Long`？

- **原因**：

  - `int` 是基本类型，`Long` 是包装类，二者之间不会自动转换。

  - 自动装箱仅在 `int ↔ Integer` 或 `long ↔ Long` 之间生效。

  - 必须显式转换为 `Long`：

    ```java
    paginationInnerInterceptor.setMaxLimit(1000L);
    ```

------

### **9. `paginationInnerInterceptor.setMaxLimit` 的意义**

- **功能**： 限制分页查询时每页最大返回记录数，防止一次性加载过多数据导致内存溢出。

- 示例：

  ```java
  paginationInnerInterceptor.setMaxLimit(1000L); // 每页最多返回 1000 条数据
  ```

------

### 10. **MyBatis-Plus 不更新为 null 的字段**

默认情况下，MyBatis-Plus 不会更新为 `null` 的字段。这是因为框架对 `null` 值的处理机制默认跳过。

**相关设置：**

- 如果需要将 `null` 值也更新到数据库，可以在 MyBatis-Plus 的全局配置中设置：

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-not-delete-value: "1"
      logic-delete-value: "0"
      update-strategy: NOT_NULL  # 改为 NULL 即可让 null 值也被更新
```

**关键配置解释：**

- `**update-strategy**`**：**
  - `NOT_NULL`（默认值）：仅更新非 `null` 值的字段。
  - `NULL`：包括 `null` 值的字段也会更新。

------

### 11. `**call-setters-on-nulls**`

- `**call-setters-on-nulls**`**（MyBatis 原生配置）**：
  - 控制从数据库查询出的字段如果为 `null`，是否调用实体类的 `setter` 方法。
  - **影响方向：数据库 -> 实例对象。反过来的暂时没有全局设置,只能在wrapper之类的设置**

如果你需要从数据库查询出的 `null` 值被赋给实体类的字段，同时也允许实体类的 `null` 值更新到数据库，你需要同时配置这两者。

------

### 12. **@RequestParam 与实体类接收查询参数**

- 如果在 Spring 中通过实体类接收查询参数，那么字段名称需要与请求中的参数名称一一对应。
- **无需显式声明** `**@RequestParam**`，因为 Spring 会自动将查询参数绑定到实体类的字段。

**示例：**

```java
@GetMapping("/users")
public List<UserVO> getUsers(UserQueryDTO queryDTO) {
    // queryDTO 自动绑定查询参数
    return userService.queryUsers(queryDTO);
}
```

**请求示例：**

```javascript
GET /users?name=John&age=25
```

------

### 13. **BaseMapper 和 Service 中的 LambdaQueryWrapper 和 lambdaQuery()**

- **BaseMapper 的特点：**
  - 包含基础的 CRUD 方法，如 `selectById()`，`selectBatchIds()` 等。
  - 不直接提供 `lambdaQuery()` 方法，但可以通过 `LambdaQueryWrapper` 来使用。
- **Service 的特点：**
  - 包含继承自 BaseMapper 的所有功能，并扩展了更多方法。
  - 提供了 `lambdaQuery()` 方法，简化了构造查询条件的流程。

**示例对比：**

```java
// BaseMapper
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(User::getAge, 25);
List<User> users = userMapper.selectList(wrapper);

// Service
List<User> users = userService.lambdaQuery().eq(User::getAge, 25).list();
```

- `LambdaQueryWrapper` 和 `lambdaQuery()` 都可以用于查询条件的构建，主要区别在于 `lambdaQuery()` 是 Service 独有的便捷方法。

------

### 14. **BeanUtil.copyProperties 的行为**

- **概念：**
  - 将源对象的属性值复制到目标对象中。
  - 只会复制 **属性名称和类型匹配** 的字段。
- **作用范围：**
  - 如果源对象中有某些字段，而目标对象中没有对应字段，则这些字段不会被复制。
  - 如果目标对象中有某些字段，而源对象中没有对应字段，则这些字段保持默认值。

**示例：**

```java
public class User {
    private String name;
    private Integer age;
    private String email;
    // Getters and Setters
}

public class UserVO {
    private String name;
    private Integer age;
    // Getters and Setters
}

User user = new User();
user.setName("John");
user.setAge(25);
user.setEmail("john@example.com");

UserVO userVO = new UserVO();
BeanUtil.copyProperties(user, userVO);
// userVO 仅复制 name 和 age，email 不会被复制。
```


