### 1. Serializable 和 Lombok的关系

- 实现 `Serializable` 接口和 Lombok 没有直接关系。
- Lombok 使用 `@Data` 等模块生成 getter、setter、toString、hashCode、equals等方法。
- `Serializable` 的目的是实现对象的序列化。在跨系统、存储、应用跨 JVM 对象传递时需要。
- Lombok 并不会自动实现 `Serializable`，需要手动声明。

------

### 2. Spring MVC 中的 `@PathVariable`

**功能：**

- 将 URL 路径中的动态参数值绑定到方法参数上。

**使用场景：**

1. 路径中包含动态参数：
   - 例如一个路径 `/order/{id}`，`{id}` 是动态参数。

```java
@GetMapping("/order/{id}")
public Order getOrderById(@PathVariable("id") Long id) {
    return orderService.findById(id);
}
```

1. 动态参数名称和方法参数不同：
   - 需要使用 `@PathVariable("id")`
2. 多个路径变量：

```java
@GetMapping("/order/{orderId}/detail/{detailId}")
public OrderDetail getOrderDetail(@PathVariable("orderId") Long orderId,
                                   @PathVariable("detailId") Long detailId) {
    return orderService.getOrderDetail(orderId, detailId);
}
```

**什么情况不用绑定：**

1. 动态参数名称和方法参数名称一致，可以省略值的指定：

```java
@GetMapping("/order/{id}")
public Order getOrderById(@PathVariable Long id) {
    return orderService.findById(id);
}
```

1. 没有动态参数时：

- 例如静态路径 `/orders`，不需要 `@PathVariable`，直接运行方法。

------

### 3. SQL 错误解析

#### **错误 1：SQL Syntax Error**

- 原因：SQL 中缺少 `AND` 关键字或者缺少条件之间的连接。

**错误 SQL例子：**

```sql
select count(0) from (
   select * from orders
   WHERE status = ?
         user_id = ?  -- 缺少 AND
   order by order_time desc
) tmp_count;
```

**修正后：**

```mysql
select count(0) from (
   select * from orders
   WHERE status = ?
         AND user_id = ?  -- 添加 AND
   order by order_time desc
) tmp_count;
```

#### **错误 2：RedisConnectionException**

- 原因：Redis 服务器没有启动，系统无法连接 Redis。
- 解决方法：
  1. 检查 Redis 服务器是否启动：`redis-server`
  2. 确认 Redis 运行状态。
  3. 确认当前使用的 Redis 给定端口和地址正确。

------

### 4. Spring Boot @RequestParam 和 @PathVariable

| 对比点     | @PathVariable                 | @RequestParam                 |
| ---------- | ----------------------------- | ----------------------------- |
| 取值源地点 | URL 路径动态参数              | Query参数或表单参数。         |
| 示例       | `/order/{id}`                 | `/order?id=123`               |
| 抽象性     | 适用于动态路径参数绑定        | 适用于请求参数绑定。          |
| 代码示例   | `@PathVariable("id") Long id` | `@RequestParam("id") Long id` |
| 选填情况   | 默认为必填，没有值会报错      |                               |