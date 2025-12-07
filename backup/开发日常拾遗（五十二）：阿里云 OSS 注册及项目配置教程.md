# 阿里云 OSS 注册及项目配置教程

本文档介绍如何从零开始 **注册阿里云 OSS、创建 Bucket、获取 AccessKey 并在项目中完成 OSS 配置**，适用于前后端分离项目、微服务项目（Nacos 配置）等场景。

------

# 一、注册并开通 OSS 服务

## 1. 登录阿里云官网

使用已有账号登录或注册新账号。

## 2. 搜索 OSS

在控制台左上搜索框输入 **对象存储 OSS**，进入 OSS 控制台页面。

## 3. 开通 OSS 服务

首次使用需点击 **“立即开通”**，流程简单，**开通免费**（含试用期）。

------

# 二、创建 Bucket（存储空间）

## 1. 进入 Bucket 创建页面

OSS 控制台 → 左侧菜单 → **Bucket 列表** → “创建 Bucket”

## 2. 基本配置

| 配置项          | 推荐设置                                 | 说明           |
| --------------- | ---------------------------------------- | -------------- |
| **Bucket 名称** | <你的bucket-name>               | 全局唯一       |
| **地域**        | 选择靠近服务器的区域（例如：华南1·深圳） | 访问更快       |
| **读写权限**    | 公共读                                   | 适用于图片访问 |
| **备份**        | 定时备份（前期免费试用）                 | 防止数据丢失   |
| 其他设置        | 默认即可                                 | —              |

## 3. 点击“确认创建”

创建完成后即可在 Bucket 列表中看到新建的存储空间。

------

# 三、获取 AccessKey（程序访问密钥）

## 1. 进入 AccessKey 管理页面

右上角头像 → **AccessKey 管理**

## 2. 创建 AccessKey

生成后会显示：

- **AccessKeyId**

- **AccessKeySecret（只显示一次）**

   **务必安全保管！不可泄露！**

### 建议

- 可以创建新 AccessKey 并禁用旧 Key
- 定期更换 AccessKey，提升安全性

------

# 四、在项目中配置 OSS 参数（Nacos）

如果创建了新的 Bucket 或新的 AccessKey，需要同步更新项目配置。

## 1. 需要修改的 OSS 配置项

以下配置需要在 Nacos 中进行修改：

| 配置项        | 含义                 |
| ------------- | -------------------- |
| `bucket-name` | 要上传的 Bucket 名称 |
| `access-key`  | AccessKeyId          |
| `secret-key`  | AccessKeySecret      |
| `endpoint`    | OSS 服务访问节点     |
| `regionId`    | OSS 所在区域         |

## 2. 查询 endpoint 与 regionId

阿里云官方查询地址：
 https://help.aliyun.com/zh/oss/user-guide/regions-and-endpoints

选择你的 Bucket 所在区域即可得到：

- 示例 `endpoint`: `oss-cn-shenzhen.aliyuncs.com`
- 示例 `regionId`: `cn-shenzhen`

------

# 五、Nacos 具体修改示例

## 1. parking-manager-test 配置示例

```
oss:
  bucket-name: <你的bucket-name>
  endpoint: oss-cn-shenzhen.aliyuncs.com
  regionId: cn-shenzhen
  access-key: <你的AccessKeyId>
  secret-key: <你的AccessKeySecret>
  prefix: https://<你的bucket-name>.oss-cn-shenzhen.aliyuncs.com/
```

## 2. ws 空间配置示例

```
oss:
  bucket-name: <你的bucket-name>
  endpoint: oss-cn-shenzhen.aliyuncs.com
  regionId: cn-shenzhen
  access-key: <你的AccessKeyId>
  secret-key: <你的AccessKeySecret>
  prefix: https://<你的bucket-name>.oss-cn-shenzhen.aliyuncs.com/
```

------

# 六、配置说明（必读）

| 配置项                            | 用途说明                                             |
| --------------------------------- | ---------------------------------------------------- |
| **prefix**                        | 用于前端访问 OSS 资源的 URL 前缀（通常为公共读地址） |
| **endpoint**                      | OSS API 接入地址（服务端使用）                       |
| **bucketName**                    | 上传目标 Bucket                                      |
| **accessKeyId / accessKeySecret** | 服务端上传文件时的认证秘钥                           |
| **regionId**                      | Bucket 所在区域 ID                                   |

**注意!!!**  **accessKeySecret 是机密信息，不可提交到 Git 仓库。
 建议放在 Nacos / 环境变量 / 配置中心中管理。**

------

# 七、常见注意事项

1. **前端只需要 prefix**, 不需要 AccessKey。
2. 服务端上传必须使用包含密钥的 OSS 客户端配置。
3. 若 Key 泄露需立即禁用并创建新 Key。
4. Bucket 地域必须与项目服务器同区域或相近区域（网络性能更优）。
5. 若修改 Bucket 或 AccessKey，记得同步更新项目配置并重启服务。