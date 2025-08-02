# 综合笔记：从Spring MySQL连接问题到QPS解释

## **1. Docker容器中MySQL服务的开机自启**

### 方法：

在 Docker 中，设置 MySQL 容器开机自启可以通过以下方式实现：

1. 使用 `--restart` 参数：

   ```bash
   docker run --name mysql-container -e MYSQL_ROOT_PASSWORD=123456 -d --restart always mysql:latest
   ```

   - `--restart always`：表示无论容器是否正常退出，都自动重启。

2. 修改已存在容器的重启策略：

   ```bash
   docker update --restart always mysql-container
   ```

## **2. concat 的含义**

### 定义：

`concat` 是 "concatenate" 的缩写，意为 "连接" 或 "拼接"。在编程中，常用于将字符串、数组等数据连接在一起。

### 示例：

- **字符串拼接（Java）**：

  ```java
  String result = "Hello".concat(" World");
  System.out.println(result); // 输出：Hello World
  ```

- **数据库拼接（SQL）**：

  ```sql
  SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;
  ```

## **3. SDK 的定义和含义**

### 英文全称：

**Software Development Kit**，即软件开发工具包。

### 含义：

- 提供了一组开发工具、库、文档等，便于开发者构建特定平台上的应用程序。

### 常见示例：

- **Android SDK**：用于开发 Android 应用。
- **Java SDK（JDK）**：包含 Java 编译器、运行环境等。

## **4. 本地文件管理器右键终端打开功能**

### **Windows 系统**

1. 打开注册表编辑器（`regedit`）。

2. 导航到：

   ```reStructuredText
   Computer\HKEY_CLASSES_ROOT\Directory\Background\shell
   ```

3. 添加新项：

   - 新建项：`Open Terminal Here`

   - 在该项下创建 `command` 子项，设置默认值为：

     ```bash
     cmd.exe /k cd %V
     ```

## **5. Sentinel 的含义**

### 定义：

- "Sentinel" 在中文中意为 "哨兵"。
- 在分布式系统中，Sentinel 是一款用于流量控制、熔断降级、系统负载保护的组件。

### 核心功能：

1. **流量控制**：QPS 限流、线程数控制。
2. **熔断降级**：当服务不可用时触发。
3. **热点参数限流**：根据参数值限流。

## **6. QPS 的定义与含义**

### 英文全称：

**Queries Per Second**，每秒查询率。

### 含义：

- 表示系统每秒能够处理的请求数。
- 是衡量系统吞吐量的重要指标。

### 示例：

- 假设一个接口在 1 分钟内被调用了 600 次，则 QPS 为：

  ```
  QPS = 总请求数 / 时间（秒） = 600 / 60 = 10
  ```

## **7. IntelliJ IDEA 日志换行设置**

### 操作步骤：

1. 打开 IntelliJ IDEA。
2. 进入 "设置 > 编辑器 > 常规 > 控制台"。
3. 勾选 "Wrap text"（自动换行）。

### 结果：

日志输出将根据控制台窗口大小自动换行，避免内容超出窗口。