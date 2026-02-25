## 一、使用 Docker 部署 Sub-Store

### ⚠️ 安全提醒

如果你**部署在公网环境**，**务必修改后端路径**。
否则后端接口可能被扫描并盗用，导致订阅泄露。

---

### 1️⃣ 使用 `docker run` 启动容器（推荐）

```bash
docker run -d \
  --name substore \
  --restart always \
  -p 3001:3001 \
  -e SUB_STORE_CRON="55 23 * * *" \
  -e SUB_STORE_FRONTEND_BACKEND_PATH="/super-random-path" \
  -v $(pwd)/data:/opt/app/data \
  xream/sub-store
```

#### 参数说明

| 参数                                | 作用               |
| --------------------------------- | ---------------- |
| `3001:3001`                       | Sub-Store 前端端口   |
| `SUB_STORE_CRON`                  | 自动更新订阅的定时任务      |
| `SUB_STORE_FRONTEND_BACKEND_PATH` | **后端真实路径（务必修改）** |
| `/opt/app/data`                   | 数据持久化目录          |

---

## 二、反向代理配置（以 Caddy 为例）

Sub-Store 前端运行在 `3001` 端口，建议通过域名反代访问。

### Caddyfile 示例

```caddyfile
sub-domain.example.com {
    reverse_proxy :3001
}
```

配置完成后，访问：

```text
https://sub-domain.example.com
```

即可进入 Sub-Store 前端界面。

---

## 三、初始化 Sub-Store 后端地址

首次进入前端时，需要手动添加 **后端地址**。

后端地址格式为：

```text
https://你的域名/你设置的后端路径
```

例如本文中的示例为：

```text
https://sub-domain.example.com/super-random-path
```

> ⚠️ 该路径必须与 `SUB_STORE_FRONTEND_BACKEND_PATH` 完全一致

---

## 四、组合订阅与节点管理

### 1️⃣ 添加上游订阅

你可以添加：

* 各类机场订阅
* 自建节点订阅
* 其他 Sub-Store 实例

全部加入后，创建一个 **组合订阅**，统一管理。

---

### 2️⃣ 批量重命名节点（强烈推荐）

不同机场的节点命名规则混乱，甚至存在重名问题。
Sub-Store 支持 **脚本操作**，可对节点进行批量整理。

#### 推荐重命名脚本

在**编辑订阅 → 脚本操作**中粘贴以下地址：

```text
https://raw.githubusercontent.com/Keywos/rule/main/rename.js
```

该脚本可自动统一节点命名格式，显著提升可读性。

---

### 3️⃣ 节点整理完成后

此时你将得到：

* 命名规范
* 去重清晰
* 适合规则分流的节点列表

可以继续根据个人偏好进行节点筛选、排序或分组。

---

## 五、生成 Mihomo（Clash Meta）配置

当前节点列表**不包含任何规则**，需要生成完整配置文件。

---

### 1️⃣ 创建新的 Mihomo 配置文件

进入 **Sub-Store → 文件管理 → 新建配置**

设置如下：

* **来源**：组合订阅
* **订阅名称**：选择你的组合订阅
* **配置类型**：Mihomo

---

### 2️⃣ 使用 JavaScript 覆写规则（推荐）

在 **脚本操作** 中填入你自己的覆写规则。

#### 推荐覆写规则（找的别人现成的）

项目地址（JavaScript 覆写）：

```text
https://raw.githubusercontent.com/powerfullz/override-rules/refs/heads/main/convert.js
```

---

### 3️⃣ 该覆写规则的主要特点

* 集成：

  * `SukkaW/Surge`
  * `Cats-Team/AdRules`
* 广告拦截与隐私保护更精细
* 新增实用分流规则：

  * Truth Social
  * E-Hentai
  * TikTok
  * 加密货币相关服务
* 移除冗余规则集，减少性能负担
* 使用 `Loyalsoldier/v2ray-rules-dat` **完整版 GeoSite / GeoIP**
* IP 规则全部添加 `no-resolve`：

  * 避免本地 DNS 查询
  * 提升访问速度
  * 防止 DNS 泄露
* **动态节点国家识别与分组**：

  * 仅为真实存在的国家/地区生成代理组
  * 节点变化时自动调整
  * 无需手动维护

示例效果：

> 如果你的订阅中没有韩国节点，最终配置中就不会出现「韩国节点」代理组。

---

## 六、生成并使用订阅链接

配置完成后：

1. 点击 **生成分享链接**
2. 将链接导入到：

   * Clash Meta
   * Mihomo Party
   * Surge（需转换）
   * 其他mihomo客户端