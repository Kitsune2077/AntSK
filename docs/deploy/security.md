---
sidebar_position: 2
---

# 安全加固指南

> ⚠️ **AntSK 的默认配置面向本地开发与内网试用。** 将其部署到生产环境前，请务必完成本文列出的全部整改项。
>
> 本文对应安全基线，建议在部署评审中逐项确认。

## 一、启动前必做（P0）

### 1.1 设置高强度登录口令

AntSK **不在文档中提供固定的默认弱口令**。生产部署必须在**首次启动前**完成口令设置。

**方式一：配置文件**

修改 `src/AntSK/appsettings.json`：

```json
"Login": {
  "User": "<你的账号>",
  "Password": "<你的高强度口令>"
}
```

**方式二：环境变量注入（推荐）**

容器化与 CI/CD 场景建议使用环境变量，避免口令写入版本库：

```bash
export Login__User="antsk_admin"
export Login__Password="<高强度口令>"
```

**口令强度要求（建议）**

- 长度 ≥ 16 位
- 同时包含大写字母、小写字母、数字与特殊字符
- 不包含项目名、公司名、常见词（如 admin、password、antsk）
- 不与团队其他系统复用
- 通过密码管理器生成与保存

> 🚨 **禁止使用弱口令**：admin、123456、test、password、12345678 等。
> 系统在启动时若检测到仍在使用默认口令，会在日志中输出醒目安全告警：
> `[SECURITY WARNING] AntSK is running with the default Login credential...`

### 1.2 不要将实例直接暴露到公网

AntSK 默认监听 `http://*:5000`（见 `appsettings.json` 的 `urls` 配置）。

- 生产环境应部署在**内网**或**反向代理**之后
- 如需对外提供服务，仅暴露反向代理端口，不要直接映射 5000
- 通过防火墙 / 安全组限制来源 IP

### 1.3 启用 HTTPS

通过 Nginx / Caddy 等反向代理终止 TLS。Nginx 示例：

```nginx
server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate     /etc/nginx/ssl/your-domain.com.crt;
    ssl_certificate_key /etc/nginx/ssl/your-domain.com.key;

    location / {
        proxy_pass         http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection "upgrade";
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```

> 注意：Blazor Server 依赖 WebSocket，必须正确转发 `Upgrade` 与 `Connection` 头，否则页面会不断重连。

## 二、加固项（P1）

### 2.1 收紧 CORS 策略

AntSK 默认的 CORS 策略允许任意来源：

```csharp
builder.Services.AddCors(options => options.AddPolicy("Any",
    b => b.AllowAnyMethod()
          .AllowAnyHeader()
          .SetIsOriginAllowed(_ => true)   // ⚠️ 允许任意来源
          .AllowCredentials()));
```

**生产环境请改为白名单**：

```csharp
builder.Services.AddCors(options => options.AddPolicy("Any",
    b => b.AllowAnyMethod()
          .AllowAnyHeader()
          .WithOrigins("https://your-trusted-domain.com")
          .AllowCredentials()));
```

### 2.2 API 调用方最小权限

- 为每个接入方分配**独立的 SecretKey**，不要共用
- 按 RBAC 角色授予最小必要权限（默认角色：`AntSKAdmin` 管理员、`AntSKUser` 普通用户）
- 定期审计并回收不再使用的 key

### 2.3 保护 Swagger 接口文档

AntSK 默认在启动时启用 Swagger UI。生产环境建议：

- 关闭 Swagger UI，或
- 通过反向代理限制 `/swagger` 与 `/swagger/v1/swagger.json` 的访问来源

### 2.4 关闭 `DetailedErrors`

`appsettings.json` 中的 `DetailedErrors: true` 会向客户端暴露详细异常信息，生产环境应设为 `false`：

```json
"DetailedErrors": false
```

## 三、运维与审计（P2）

### 3.1 日志与审计

- 定期审计登录日志与对话记录（路径见 `Serilog` 的 `WriteTo.File.path` 配置）
- 隔离日志目录的访问权限，避免越权读取
- 建议接入集中式日志平台（如 Seq、ELK、OpenTelemetry Collector）

### 3.2 数据备份

需要备份的内容：

| 内容 | 说明 |
|------|------|
| 关系型数据库 | 用户、应用、知识库元数据、对话记录 |
| 向量存储 | pgvector 表数据 / Qdrant 数据 / Disk 向量目录 |
| 模型与文件目录 | `FileDir.DirectoryPath` 指向的目录 |
| 配置文件 | `appsettings.json`（注意口令已外置为环境变量） |

建议制定定期备份策略并**演练恢复流程**。

### 3.3 及时更新

- 关注 GitHub 仓库的安全公告与 Release
- 定期升级到最新稳定版本
- 升级前先在测试环境验证

### 3.4 容器安全（Docker 部署）

- 不要以 root 身份运行容器
- 限制容器资源（CPU / 内存 / GPU）
- 挂载目录最小化，模型目录与数据目录分离
- 不要将数据库端口（PostgreSQL 5432）映射到公网

## 四、自查清单

部署上线前逐项确认：

- [ ] 已修改默认登录口令，使用 ≥16 位高强度口令
- [ ] 口令通过环境变量注入，未写入版本库
- [ ] 实例未直接暴露公网，位于反向代理或内网之后
- [ ] 已启用 HTTPS 并正确转发 WebSocket
- [ ] CORS 来源已收紧为可信域名白名单
- [ ] 每个 API 调用方使用独立 SecretKey，权限最小化
- [ ] Swagger UI 已关闭或限制访问来源
- [ ] `DetailedErrors` 已设为 `false`
- [ ] 日志目录权限已隔离，并接入集中式日志（可选）
- [ ] 数据库、向量存储、模型目录已配置定期备份
- [ ] 已确认当前版本为最新稳定版

---

**相关文档**：[常见问题 FAQ](../introduce/faq.md) ｜ [如何开始](./start.md) ｜ [配置说明](./settings.md)
