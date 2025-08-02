### 1. **Association 和 Collection 标签的区别**

在 MyBatis 的 `resultMap` 中，`association` 和 `collection` 是两个重要的标签，用于映射数据库查询结果到 Java 对象中，但它们的作用不同：

- **`association`**：用于映射一对一关系。即一个对象（主对象）与另一个对象（关联对象）之间的关系。这个标签表示某个字段是另一个 Java 对象的属性，通常用于从查询中提取单一的对象。
  - **示例**：当查询结果中包含了一个主对象的属性，并且该属性本身也是一个 Java 对象时，`association` 标签用于映射这个属性。
- **`collection`**：用于映射一对多关系。即一个对象与多个对象之间的关系。通常用于当查询结果中，主对象拥有多个子对象时，使用 `collection` 来表示这种关系。通过 `collection` 标签，能够将多个结果行映射为一个集合类型的属性（如 `List` 或 `Set`）。
  - **示例**：当查询结果中，一个父节点关联了多个子节点时，可以使用 `collection` 来将这些子节点映射到父节点对象中的集合属性。

#### 示例：`association` 与 `collection` 的使用

##### 代码示例：`association` 和 `collection` 配置

```xml
<resultMap id="treeNodeResultMap" type="com.xuecheng.content.model.dto.TeachplanDto">
    <!-- 一级数据映射 -->
    <id column="one_id" property="id"/>
    <result column="one_pname" property="pname"/>
    <result column="one_parentid" property="parentid"/>
    <result column="one_grade" property="grade"/>
    <result column="one_mediaType" property="mediaType"/>
    <result column="one_startTime" property="startTime"/>
    <result column="one_endTime" property="endTime"/>
    <result column="one_orderby" property="orderby"/>
    <result column="one_courseId" property="courseId"/>
    <result column="one_coursePubId" property="coursePubId"/>
    <!-- 一级中包含多个二级数据, 一对多映射 -->
    <collection property="teachPlanTreeNodes" ofType="com.xuecheng.content.model.dto.TeachplanDto">
        <id column="two_id" property="id"/>
        <result column="two_pname" property="pname"/>
        <result column="two_parentid" property="parentid"/>
        <result column="two_grade" property="grade"/>
        <result column="two_mediaType" property="mediaType"/>
        <result column="two_startTime" property="startTime"/>
        <result column="two_endTime" property="endTime"/>
        <result column="two_orderby" property="orderby"/>
        <result column="two_courseId" property="courseId"/>
        <result column="two_coursePubId" property="coursePubId"/>
        <association property="teachplanMedia" javaType="com.xuecheng.content.model.po.TeachplanMedia">
            <id column="teachplanMeidaId" property="id"/>
            <result column="mediaFilename" property="mediaFilename"/>
            <result column="mediaId" property="mediaId"/>
        </association>
    </collection>
</resultMap>
```

##### 查询语句：将一对多关系映射到 `TeachplanDto` 中

```sql
select one.id            one_id,
       one.pname         one_pname,
       one.parentid      one_parentid,
       one.grade         one_grade,
       one.media_type    one_mediaType,
       one.start_time    one_startTime,
       one.end_time      one_endTime,
       one.orderby       one_orderby,
       one.course_id     one_courseId,
       one.course_pub_id one_coursePubId,
       two.id            two_id,
       two.pname         two_pname,
       two.parentid      two_parentid,
       two.grade         two_grade,
       two.media_type    two_mediaType,
       two.start_time    two_startTime,
       two.end_time      two_endTime,
       two.orderby       two_orderby,
       two.course_id     two_courseId,
       two.course_pub_id two_coursePubId,
       m1.media_fileName mediaFilename,
       m1.id             teachplanMeidaId,
       m1.media_id       mediaId
from teachplan one
         INNER JOIN teachplan two on one.id = two.parentid
         LEFT JOIN teachplan_media m1 on m1.teachplan_id = two.id
where one.parentid = 0
  and one.course_id = 117
order by one.orderby,
         two.orderby
```

------

### 2. **Collection 标签专门用来处理一对多关系**

`collection` 标签专门处理一对多关系。在数据库查询结果中，父节点和多个子节点的映射需要使用 `collection` 标签来处理。**每一行返回的结果都包含父节点和对应的子节点**，但是 MyBatis 通过查询的 JOIN 结果将这些子节点映射到父节点对象的集合属性上。

- **返回结果**：每行记录是父节点信息和对应的子节点信息一起返回的，MyBatis 会根据定义的 `resultMap`，将这些子节点包装到父节点对象中的集合属性（例如 `List`）中。

#### 示例：

假设查询结果是类似下面这样的：

```reStructuredText
268,1.配置管理,0,1,,,,1,117,,269,1.1 什么是配置中心,268,2,,,,1,117,,01-Nacos配置管理-内容介绍.avi
268,1.配置管理,0,1,,,,1,117,,270,1.2 Nacos简介,268,2,,,,2,117,,16-Nacos配置管理-课程总结.avi
```

- 第一行是父节点信息（`268`），第二行是子节点信息（`269`）。这些数据通过 `collection` 标签被映射到父节点的 `teachPlanTreeNodes` 属性中。

------

### 3. **类成员变量是否需要初始化**

在 Java 中，类成员变量（实例变量）并不一定需要在定义时初始化，但如果是 `final` 类型或是某些需要初始化的特定情况，才必须初始化。例如：

- **非 `final` 成员变量**：可以在构造函数中初始化，也可以在声明时赋予默认值。
- **`final` 成员变量**：必须在声明时或者构造函数中进行初始化。

#### 示例：

```java
public class Example {
    private String name;  // 非final成员变量，可以不初始化
    private final int age = 30;  // final成员变量，必须初始化

    public Example() {
        // 构造函数中可以初始化非final变量
        name = "John";
    }
}
```

------

### 4. **Throw Exception 的可侵入性**

`throw Exception` 是可侵入性的，主要是因为它会导致调用栈中所有方法的签名发生变化。具体来说：

- 当一个方法抛出异常时，所有调用这个方法的地方都必须处理这个异常，**要么捕获（`catch`），要么声明抛出（`throws`）**。
- 这种强制要求处理异常的方式增加了代码的耦合性。特别是异常的传播可能会导致代码变得冗长且复杂。

#### 示例：

```java
public void method1() throws Exception {
    method2();
}

public void method2() throws Exception {
    throw new Exception("An error occurred");
}
```

这里，`method1` 和 `method2` 都需要声明抛出异常，增加了代码的复杂性。

------

### 5. **沙箱测试的定义**

沙箱测试（Sandbox Testing）是一种在隔离环境中进行的测试方法，常用于评估软件、系统或代码的安全性。沙箱是一个封闭的环境，它与真实系统完全隔离，用于测试潜在的安全风险或不稳定因素。

- **隔离性**：沙箱环境与生产环境分开，避免测试过程中的风险影响到生产环境。
- **安全性**：用于测试未知的软件或代码，确保其没有恶意行为。

#### 常见应用：

- **网络安全**：测试恶意代码或漏洞。
- **软件功能测试**：验证不稳定或未经过充分验证的代码。

------

### 6. **A 依赖 B 时配置文件问题**

当 A 依赖 B 时，A **通常不能自动获得** B 的配置文件。A 需要显式地引用或加载 B 的配置文件。

- **A 和 B 的配置是分开的**：即使 A 依赖 B，A 不会自动继承 B 的配置文件。
- **显式加载配置**：A 需要在自己的配置文件中或者代码中显式加载 B 的配置。

#### 解决方法：

1. **Spring 环境**：通过 `@PropertySource` 或 `@Import` 加载 B 的配置。
2. **非 Spring 环境**：手动加载配置文件，例如使用 `Properties` 类读取配置。