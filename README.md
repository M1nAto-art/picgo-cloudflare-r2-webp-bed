# 🚀 PicGo-Cloudflare-R2-WebP-Bed: Cloudflare R2 极速免费 WebP 自动化图床搭建与终极闭坑指南

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

建站、写博客、Markdown 深度用户的终极省钱与性能优化方案！

本方案旨在引导你使用 **PicGo 客户端**，无缝对接 **Cloudflare R2** 免费对象存储（每月 10GB 免费额度，全网免流量费），并在上传瞬间**全自动将图片转换为 WebP 格式并极致高压缩**。

针对国内用户配置此流程时必然遭遇的 `imagemin` 插件安装报错、二进制包下载死锁、Cloudflare S3 权限配置错乱等“高血压”问题，本项目给出了**百分之百跑通的完全体闭坑解决方案**。

---

## ✨ 为什么选择这套方案？

- 💰 **彻底免费**：Cloudflare R2 每月提供 10GB 存储额度，**A/B类请求全部免费配额，最重要的是公网出流量（Download）完全免费！** 配合 CF CDN，一毛钱不用花就能拥有顶级大厂的高防 CDN 节点加速。
- ⚡ **无感自动 WebP 压缩**：本地拖入 5MB 的原生 4K 截图，PicGo 瞬间在后台全自动压缩成 200KB 的超清 WebP 并完成上传，Markdown 直接拿链接，网站加载速度直接飙升 500%！
- 🛡️ **专治各种安装不服**：全量收录了解决本地 Node 环境报错、网络死锁的底层黑魔法，拒绝鬼打墙。

---

## 🛠️ 第一阶段：Cloudflare R2 后台绝不翻车配置

要让 PicGo 能顺利把图砸进 R2，必须拿到正确的 S3 密钥：

1. **新建存储桶（Bucket）**：
   - 登录 Cloudflare 后台 -> `R2` -> `创建存储桶`。
   - 命名你的桶（例如：`my-img-bed`），地区选择 `自动` 即可。
2. **绑定自定义域名（核心，否则无法公网访问）**：
   - 点进建好的桶 -> `设置` -> 找到 `公开访问`。
   - 点击 `连接域`，输入你在 CF 解析的专属二级域名（例如：`img.yourdomain.com`），让 CF 全自动完成配置。
3. **获取 S3 API 凭证（至关重要 ⚠️）**：
   - 回到 R2 首页，点击右侧的 **`管理 R2 API 令牌`** -> `创建 API 令牌`。
   - 权限选择：**`管理员读写（Admin Read and Write）`**（千万别选只读！）。
   - TTL 选择 `永久`。
   - 点击创建后，**立刻用记事本抄下以下三个核心参数**（刷新页面就再也看不到了！）：
     - `访问令牌 ID` (对应 PicGo 的 AccessKeyId)
     - `机密访问令牌` (对应 PicGo 的 SecretAccessKey)
     - `为 S3 客户端管辖的终结点 URL` (简称为 Endpoint)

---

## 📦 第二阶段：PicGo 客户端配置与插件逆天闭坑

### 1. 基础图床插件安装
1. 打开 PicGo，进入 `插件设置`，搜索并安装 **`s3`** 插件（推荐使用 `s3-repository` 插件，对 R2 兼容性极好）。
2. 配置 S3 插件参数：
   - **应用密钥 ID**：填入刚才复制的 `访问令牌 ID`。
   - **应用密钥**：填入刚才复制的 `机密访问令牌`。
   - **桶名**：填入你在 R2 创建的存储桶名字（如 `my-img-bed`）。
   - **自定义连接地址 (Endpoint)**：填入刚才复制的 `终结点 URL`。
   - **自定义域名**：填入你绑定的自定义域名（带上协议头，如 `https://img.yourdomain.com`）。

---

### 2. WebP 自动化压缩插件安装（高血压重灾区 ⚡）

要实现自动转 WebP 并在本地高压缩，我们需要安装 **`web-transformer`** 插件。**直接在 PicGo 界面点击安装大概率会因为二进制墙卡死报错！请使用以下硬核姿势安装：**

#### 🔥 终极闭坑通关步骤：
1. **Windows 用户**打开 PowerShell / **Mac 用户**打开终端，通过全局换源，强行打通二进制文件的下载通道。直接按顺序运行以下三行命令：
   ```bash
   # 1. 切换国内清华/淘宝源
   npm config set registry [https://registry.npmmirror.com](https://registry.npmmirror.com)
   
   # 2. 设置 imagemin 底层二进制文件的国内专属镜像（核心灵魂！绕过外网死锁）
   npm config set gif鼓勵_binary_host_mirror [https://npmmirror.com/mirrors/giflib](https://npmmirror.com/mirrors/giflib)
   npm config set jpegtran_binary_host_mirror [https://npmmirror.com/mirrors/jpegtran-bin](https://npmmirror.com/mirrors/jpegtran-bin)
   npm config set optipng_binary_host_mirror [https://npmmirror.com/mirrors/optipng-bin](https://npmmirror.com/mirrors/optipng-bin)
   npm config set pngquant_binary_host_mirror [https://npmmirror.com/mirrors/pngquant-bin](https://npmmirror.com/mirrors/pngquant-bin)
   ```
2. **肉身强行切进 PicGo 插件运行底层目录**：
   - Windows 路径：`cd $env:APPDATA\picgo`
   - Mac 路径：`cd ~/.config/picgo`
3. **在底层直接用命令强制灌入插件**：
   ```bash
   npm install picgo-plugin-web-transformer --legacy-peer-deps
   ```
4. 运行成功后，彻底重启 PicGo 客户端。此时在 `插件设置` 里，你会发现 `web-transformer` 已经乖乖听话、完好无损地亮起来了！

---

## ⚙️ 第三阶段：`web-transformer` 完美参数推荐

点击 `web-transformer` 插件的齿轮进入配置，为了达到画质毫无肉眼损失、体积缩减到极致的平衡，请无脑抄以下作业：

```json
{
  "convertFormat": "webp",    // 强制转换为完全体 WebP 格式
  "isConvert": true,          // 开启自动转换
  "quality": 75,              // 推荐画质设定为 75（学术界公认的 WebP 最佳性价比性价比甜点区，体积暴缩，画质无感）
  "appendedString": ""        // 后缀留空，保持原文件名清爽
}
```

---

## 🏁 享用起飞成果

全部对齐之后，将 PicGo 的默认图床切换为你的 S3 对应存储桶。

现在，你随便截一张几兆大的 PNG 巨图，往 PicGo 悬浮窗里一丢——本地 Nginx 级内核全自动将其踩碎成 100 多 KB 的 WebP，紧接着无感空投至 Cloudflare R2 全球 CDN 铁冷库。 Markdown 链接瞬间到手，整套网站图床链路直接进入全自动赛博现代化时代！

---

## 📄 许可证

本项目基于 **[MIT License](LICENSE)** 协议开源。
欢迎各位被 `imagemin` 折腾过的站长、极客老哥们提交 PR 补充更多操作系统的底层闭坑环境魔改法。
