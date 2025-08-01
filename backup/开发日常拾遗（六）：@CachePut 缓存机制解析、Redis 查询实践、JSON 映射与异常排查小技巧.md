### 1. **Spring Cache 与 `@CachePut` 注解**

- **`@CachePut` 注解** 用于方法执行后，将方法的返回值放入缓存中，而不是从缓存中获取数据。它可以确保缓存中的数据与方法返回值同步更新。
- 关键点：`@CachePut` 会在方法执行完之后才执行，缓存的值是方法的返回值，而不是方法参数的值。例如，在保存 `User` 时，`@CachePut(cacheNames = "userCache", key = "#user.id")` 会根据执行完的方法返回值，将数据存储到缓存中。
- Spring 缓存的机制是方法执行完毕后将返回值放入缓存中，避免了直接在方法开始时就对缓存进行操作，从而确保缓存的数据是最新的。

### 2. **Redis 操作与缓存配置**

- **Spring Data Redis**：Spring Data Redis 是 Spring 提供的对 Redis 数据库的支持，能够简化与 Redis 交互的过程。
- Redis 键查询使用：可以通过 `keys` 命令在 Redis 中查询指定模式的键，例如 `keys "prefix:*"` 查询所有以 `prefix:` 开头的键，但在生产环境中不推荐使用 `keys`，因为它可能导致性能问题。
- 使用 Redis 存储数据时，Spring Data Redis 会自动处理数据的序列化和反序列化。Spring 提供了缓存注解，如 `@Cacheable`, `@CachePut`, `@CacheEvict` 等，这些注解可以简化 Redis 缓存的操作，自动处理缓存数据。

### 3. **MyBatis 中的 `@CachePut` 和参数映射**

- **MyBatis 映射文件中的 `parameterType` 和 `resultType`**：`parameterType` 用于指定方法参数的类型，`resultType` 用于指定查询结果的类型。在 XML 映射文件中，`parameterType="Dish"` 代表方法参数为 `Dish` 类型，`resultType="Dish"` 代表返回值为 `Dish` 类型。
- **`@CachePut` 注解中的缓存键**：在 `@CachePut` 中，`key = "#user.id"` 会将方法的返回值 `user` 的 `id` 作为缓存的键。这个缓存键在方法执行完后生成，确保缓存的数据是最新的。

### 4. **"sub" 作为动词的含义**

在某些语境中，**"sub"** 作为动词时有 **"删除"** 或 **"替代"** 的意思。具体来说：

- **"substitute"** 的缩写形式，“sub” 常常表示 **替代**，例如在命令行或脚本中，常用于替换文本或变量。
- 在某些编程环境中，**sub** 用于表示将一个元素从列表或数组中移除，或者在特定情况下表示进行 **删除操作**。

比如，在编程中，常见的 `sub` 操作是替换或删除的操作：

- **替换 (substitute)**：用新的值替换旧的值。
- **删除 (sub)**：某些框架或语言中使用 `sub` 来表示移除、删除操作。

**举例**：

- 在 **Perl** 或 **Ruby** 中，`sub` 作为关键字用于定义函数或方法，但在某些上下文中，它也可以表示删除或替代的操作。
- 在正则表达式中，`sub` 是替换操作的简写。

​       这个词的具体意义取决于使用的上下文。在一般的自然语言中，**"sub"** 主要是 **"替代"** 或 **"替换"** 的意思，但在编程或命令行操作中，它有时也用作表示 **删除**。

### 5. **JSON 解析错误：`HttpMessageNotReadableException`**

- 出现 **`HttpMessageNotReadableException`** 错误时，通常是因为前端发送的 JSON 数据格式错误，Spring 无法解析请求体中的数据。常见原因包括 JSON 格式错误（如缺少逗号、引号不匹配等）或 `Content-Type` 错误。
- 解决方法：
  1. 确保前端发送的 JSON 数据格式正确。
  2. 使用工具（如 JSONLint）验证 JSON 格式。
  3. 检查请求头中的 `Content-Type` 是否正确设置为 `application/json`。
  4. 确保请求体中的 JSON 数据完整，没有丢失。

### 6. **Spring 的 `@RequestBody` 自动映射**

- **`@RequestBody` 注解** 使 Spring 自动将 HTTP 请求体中的 JSON 数据转换为 Java 实体类对象。Spring 使用 **`HttpMessageConverter`**（通常是 Jackson）进行数据的序列化和反序列化。
- 例如，前端发送 JSON 数据，Spring 会自动将其映射到对应的实体类参数中，方法的参数就变成了请求体中的数据。
- 关键点：
  1. `@RequestBody` 自动将 JSON 数据绑定到方法参数。
  2. 前端需要正确设置 `Content-Type: application/json`。
  3. 请求体中的字段名需要与 Java 类中的属性匹配。

### 7. **Redis 键查询与 `keys` 命令的注意事项**

- Redis 提供了 `keys` 命令用于匹配指定模式的所有键，但在生产环境中不推荐使用 `keys` 命令，因为它会遍历整个 Redis 数据库，可能对性能造成影响。
- 推荐使用 Redis 的 **`SCAN`** 命令，它是一个增量式的键扫描方法，不会对 Redis 服务器的性能造成严重影响，适用于大规模的数据查询。