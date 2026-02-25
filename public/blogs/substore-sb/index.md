### 1. 打开您部署的 sub-store([没部署sub-store的和要mihomo配置的看这篇](https://www.nodeseek.com/post-587420-1))

 打开 Sing-box 面板首页。
---

### 2. 进入 “文件管理” 页面

在面板底部或侧边栏，点击 **文件管理**。

---

### 3. 新建文件

点击页面左上角的 **+** 按钮来新建一个文件。

---

### 4. 填入文件名称

在弹出的对话框中输入任意名称。

---

### 5. 打开 GitHub 配置模板仓库

访问：

```
https://github.com/kj163kj/singbox_proxy_config
```

选择对应你客户端版本的的配置文件。

---

### 6. 选择对应配置模板

找到与你的 singbox 版本匹配的配置文件夹，打开你需要的配置文件，然后点击 **Raw** 查看它的原始内容。

---

### 7. 复制原始文件的 URL

当你打开了 Raw 文件之后，**从浏览器地址栏完整复制该 URL**，例如：

```
https://raw.githubusercontent.com/…
```

这是你需要引入到 Sub Store 的链接。

---

### 8. 在 Sub Store 中填写链接

回到 Sub Store 的界面，将刚才复制的 URL 填入到 **来源下面的原程链接位置** 

---

### 9. 执行脚本操作

在 Sub Store 中找到 **脚本操作**

---

### 10. 填写脚本链接

在脚本操作中，将以下模板 JS 脚本链接（本例）填入对应位置：

```
https://raw.githubusercontent.com/xream/scripts/main/surge/modules/sub-store-scripts/sing-box/template.js#name=tx&outbound=%F0%9F%95%B3%E2%84%B9%EF%B8%8F%F0%9F%9A%80%20%E8%8A%82%E7%82%B9%E9%80%89%E6%8B%A9%F0%9F%95%B3%E2%84%B9%EF%B8%8F%F0%9F%87%AD%F0%9F%87%B0%20%E9%A6%99%E6%B8%AF%F0%9F%8F%B7%E2%84%B9%EF%B8%8F%E6%B8%AF%7Chk%7Chongkong%7Ckong%20kong%7C%F0%9F%87%AD%F0%9F%87%B0%F0%9F%95%B3%E2%84%B9%EF%B8%8F%F0%9F%87%B9%F0%9F%87%BC%20%E5%8F%B0%E6%B9%BE%F0%9F%8F%B7%E2%84%B9%EF%B8%8F%E5%8F%B0%7Ctw%7Ctaiwan%7C%F0%9F%87%B9%F0%9F%87%BC%F0%9F%95%B3%E2%84%B9%EF%B8%8F%F0%9F%87%AF%F0%9F%87%B5%20%E6%97%A5%E6%9C%AC%F0%9F%8F%B7%E2%84%B9%EF%B8%8F%E6%97%A5%E6%9C%AC%7Cjp%7Cjapan%7C%F0%9F%87%AF%F0%9F%87%B5%F0%9F%95%B3%E2%84%B9%EF%B8%8F%F0%9F%87%B8%F0%9F%87%AC%20%E6%96%B0%E5%8A%A0%E5%9D%A1%F0%9F%8F%B7%E2%84%B9%EF%B8%8F%5E(%3F!.*(%3F%3Aus)).*(%E6%96%B0%7Csg%7Csingapore%7C%F0%9F%87%B8%F0%9F%87%AC)%F0%9F%95%B3%E2%84%B9%EF%B8%8F%F0%9F%87%BA%F0%9F%87%B8%20%E7%BE%8E%E5%9B%BD%F0%9F%8F%B7%E2%84%B9%EF%B8%8F%E7%BE%8E%7Cus%7Cunitedstates%7Cunited%20states%7C%F0%9F%87%BA%F0%9F%87%B8&type=%E6%9C%AC%E5%9C%B0%E8%AE%A2%E9%98%85
```

> 链接中带 `#name=…&outbound=…` 可以根据实际需要修改名称和节点选择内容（如国家/地区筛选关键字）。

---

### 11. 编辑参数

运行后，在脚本操作底部会出现一组 **参数编辑**：

```text
参数名     | 参数值
--------------------
name       | 填入你sub-store订阅名
type       | sub-store订阅类型
...
```
![image](https://tc.airv.us/rest/2026/01/Ugb0v2k.jpeg)
这里的hkvless，us，tx就是要填入name区域的参数，而本地订阅就是填入type部分的参数
如果有**多个**单条订阅，那么就要添加多个脚本，单个脚本不能填入多个订阅，也不能填入聚合订阅
根据实际需要修改这些参数值以定制你的 Sing-box 配置输出。


---

### 12. 导入

完成设置后，可以选择导出生成的配置，或直接导入到Sing-box客户端使用