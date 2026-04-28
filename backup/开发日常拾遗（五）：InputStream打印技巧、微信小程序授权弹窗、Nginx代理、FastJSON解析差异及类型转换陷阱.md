#### **1. 打印 `InputStream` 的内容**

**问题背景：**
在代码中获取 HTTP 响应内容时，想打印 `InputStream` 内容，但直接打印流对象只输出其地址信息。

```java
InputStream content = response.getEntity().getContent();
System.out.println(content); // 输出：java.io.BufferedInputStream@xxxx
```

**解决方案：**

1. 通过 `BufferedReader` 读取内容：

   ```java
   InputStream content = response.getEntity().getContent();
   BufferedReader reader = new BufferedReader(new InputStreamReader(content));
   StringBuilder result = new StringBuilder();
   String line;
   while ((line = reader.readLine()) != null) {
       result.append(line);
   }
   System.out.println(result.toString());
   ```

2. 使用 Apache Commons IO 工具类：

   - 依赖：`org.apache.commons:commons-io`

   - 示例代码：

     ```java
     String result = IOUtils.toString(content, StandardCharsets.UTF_8);
     System.out.println(result);
     ```

------

#### **2. 微信小程序 `button` 点击没有弹窗**

**问题背景：**
点击微信小程序的按钮，使用 `wx.getUserProfile` 方法获取用户信息时，未出现授权弹窗。

**原因分析：**

- 微信小程序的基础库版本较新，废除了用户信息授权的弹窗逻辑。
- 官方文档建议将基础调试库切换至 **2.27** 或更低版本。

**解决方案：**

- 修改微信开发者工具的调试基础库版本：
  点击右上角的设置按钮，选择基础库版本，降级为 2.27 及以下。

- 调用代码中使用 `wx.getUserProfile`获取用户信息：

  ```javascript
  wx.getUserProfile({
      desc: '用于完善会员资料',
      success: (res) => {
          console.log(res.userInfo);
      }
  });
  ```

------

#### **3. Nginx 代理与前端请求端口的关系**

**问题背景：**
使用 Nginx 作为前端代理，Nginx 监听 80 端口，后端服务监听 8080 端口，前端请求时应该使用哪个端口？

**解答：**

- 请求流程：
  1. 前端发送请求至 80 端口（Nginx）。
  2. Nginx 根据配置文件将请求反向代理到后端服务（8080）。
- 前端请求端口：
  - 前端应始终请求 80 端口。
  - 后端 8080 端口对外隐藏，前端无需感知。

------

#### **4. FastJSON 的 `parse` 与 `parseObject` 区别**

**区别概述：**

1. **`JSON.parse(String text)`**：

   - 作用：将 JSON 字符串解析为 `Object` 类型。

   - 适用场景：仅需要 JSON 的基础解析，不关心具体类型。

   - 示例：

     ```java
     Object obj = JSON.parse("{\"name\":\"Tom\"}");
     System.out.println(obj); // 输出 LinkedHashMap
     ```

2. **`JSON.parseObject(String text, Class<T> clazz)`**：

   - 作用：将 JSON 字符串解析为指定类型的 Java 对象。

   - 适用场景：需要将 JSON 数据映射为具体的实体类。

   - 示例：

     ```java
     class User {
         private String name;
         // getter & setter
     }
     User user = JSON.parseObject("{\"name\":\"Tom\"}", User.class);
     System.out.println(user.getName()); // 输出 Tom
     ```

------

#### **5. `toString()` 在类型转换中的作用**

**问题背景：**
在以下代码中，`toString()` 是否可以省略？

```java
Long empId = Long.valueOf(claims.get(JwtClaimsConstant.EMP_ID).toString());
```

**分析：**

- `claims.get()` 返回的是 `Object` 类型。
- 不调用 `toString()` 时：
  - 如果返回值是 `String` 类型：`Long.valueOf((String) object)` 可以正常运行。
  - 如果返回值是非 `String` 类型（如 `Integer`）：会报错 `NumberFormatException`。
  - 如果返回值是 `null`：会抛出 `NullPointerException`。
- **建议：** 使用 `toString()` 是更安全的做法，确保转换时不会因类型不匹配出错。

