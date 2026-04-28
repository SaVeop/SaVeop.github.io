#### **一、静态变量与静态方法的理解**

##### 1. **静态（`static`）成员的定义和访问**：

- **静态变量（`static` field）** 和 **静态方法（`static` method）** 是属于 **类** 本身的，而不是属于 **类的实例**。因此，它们是共享的，所有实例对象都访问相同的静态成员。

##### 2. **静态成员的访问方式**：

- 静态变量和静态方法可以通过 **类名** 来访问。例如：`ClassName.variable` 或 `ClassName.method()`。
- 即使没有创建类的实例，我们也可以通过类名直接访问静态成员。
- 静态变量可以通过实例对象访问，但通常不推荐，因为这容易引起混淆，且静态成员属于类，不属于某个实例。

##### 3. **静态方法的限制**：

- **静态方法** 只能访问 **静态成员**（包括静态变量和静态方法），不能直接访问实例变量和实例方法。
- **实例方法** 则可以访问 **静态成员** 和 **实例成员**，因为实例方法是由对象实例调用的。

##### 4. **推荐访问方式**：

- **推荐通过类名** 来访问静态变量和静态方法，因为这能清晰表明该变量/方法是属于类的，而不是某个对象实例。

##### **示例代码**：

```java
public class MyClass {
    // 静态变量
    public static int staticCount = 0;

    // 实例变量
    public int instanceCount = 0;

    // 静态方法
    public static void staticMethod() {
        // 只能访问静态变量和静态方法
        System.out.println("Static count: " + staticCount);
    }

    // 实例方法
    public void instanceMethod() {
        // 可以访问静态变量和实例变量
        System.out.println("Static count: " + staticCount);  // 访问静态变量
        System.out.println("Instance count: " + instanceCount);  // 访问实例变量
    }

    public static void main(String[] args) {
        // 通过类名调用静态方法
        MyClass.staticMethod();

        // 实例化对象后，可以调用实例方法
        MyClass obj = new MyClass();
        obj.instanceMethod();
    }
}
```

------

#### **二、Redis 与 Spring Data Redis 使用**

##### 1. **Redis 数据类型：`Set` 和 `Hash`**：

- Redis 提供了多种数据类型，其中`Set` 和 `Hash`是常用的无序集合。
  - **Set**：一个集合，不包含重复元素，适用于存储不重复的数据。
  - **Hash**：一个键值对集合，其中的每个元素都有一个字段（`field`）和一个值（`value`），支持根据字段来存取数据。

##### 2. **`Set` 和 `Hash` 的区别**：

- **`Set`**：无序集合，不包含重复元素，存储的是唯一的值。底层实现可以使用哈希表（`hash table`）。
- **`Hash`**：类似于 `Map` 的结构，是键值对集合，其中 **字段**（`field`）是唯一的，但同一个字段的 **值**（`value`）可以重复。

##### 3. **Redis 操作示例**：

- 使用 `RedisTemplate`进行数据操作：
  - `redisTemplate.opsForValue()`：操作 Redis 中的字符串（String）。
  - `redisTemplate.opsForHash()`：操作 Redis 中的哈希表（Hash）。
  - `redisTemplate.opsForSet()`：操作 Redis 中的集合（Set）。

------

#### **三、问题解析：`ClassCastException` 问题**

- **问题描述**： 在使用 `RedisTemplate` 设置和获取值时，出现了 `java.lang.ClassCastException`，原因是 Redis 默认将值存储为字符串，而你尝试将其转化为整数。

- **解决方案**： 使用 `ValueOperations` 来设置和获取数据时，确保获取的数据类型与存储的数据类型匹配。例如，将整数存储为字符串后，在获取时需要将其转换回正确的类型：

  ```java
  // 存储整数
  valueOperations.set("name", 100);
  
  // 获取并转化为字符串
  String name = String.valueOf(valueOperations.get("name"));
  System.out.println(name);
  ```

------

#### **四、Spring Data Redis 与 Redis 操作**

##### 1. **Spring Data Redis 的配置**：

- 使用 Spring Boot 集成 Redis 时，只需要在 `pom.xml` 文件中添加 Redis 相关的依赖，并配置 Redis 连接信息即可。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

- 在 `application.yml` 文件中配置 Redis 连接信息：

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: yourpassword
    timeout: 2000ms
```

##### 2. **示例：设置和获取店铺的营业状态**：

- **设置营业状态**：

```java
@PutMapping("/{status}")
@ApiOperation("设置营业状态")
public Result setStatus(@PathVariable Integer status){
    log.info("设置店铺状态为:{}", status == 1 ? "营业": "打烊");
    redisTemplate.opsForValue().set(KEY, status);
    return Result.success();
}
```

- **获取营业状态**：

```java
@GetMapping("/status")
@ApiOperation("获取营业状态")
public Result<Integer> getStatus(){
    Integer status = (Integer) redisTemplate.opsForValue().get(KEY);
    log.info("当前营业状态为:{}", status == 1 ? "营业": "打烊");
    return Result.success(status);
}
```

- **总结**：以上例子展示了如何使用 `RedisTemplate` 存储和获取数据。通过 `opsForValue()` 操作 Redis 中的值，使用 `KEY` 作为键，存储和获取 `Integer` 类型的数据。

------

#### **五、总结与思考**：

- **`static` 关键字的应用**：
  - `static` 成员属于类，而非类的实例。可以通过类名来访问静态成员，推荐这种方式。
  - 静态方法可以访问静态变量和静态方法，但不能直接访问实例变量和实例方法。
  - 实例方法可以访问静态变量、静态方法以及实例变量、实例方法。
- **Redis 数据类型与 Spring Data Redis**：
  - Redis 提供了多种数据类型，包括字符串、集合（`Set`）、哈希表（`Hash`）等，每种类型适用于不同的场景。
  - 使用 Spring Boot 集成 Redis 时，可以通过 `RedisTemplate` 进行数据存取操作，简化了与 Redis 的交互。