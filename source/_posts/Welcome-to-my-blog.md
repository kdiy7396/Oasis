---
title: First Blog
date: 2026-01-03 18:00:00
tags:
  - 生活
  - 碎碎念
categories:
  - 日常
---

# Blog 部署日记

这是我的第一个 Blog。  
在 AI 的帮助下，我终于把它成功部署了。正如你现在看到的这样，页面还比较原始，但这只是开始。

我相信，它会慢慢发展成我理想中的样子。

---

## 一、准备工作

在开始部署之前，你需要准备好以下软件和服务：

1. 一个 **GitHub 账号**  
   👉 https://github.com/

2. 使用 GitHub 账号登录 **Vercel**  
   👉 https://vercel.com/

3. **Node.js**（推荐 LTS 版本）  
   👉 https://nodejs.org/

4. **Git**  
   👉 https://git-scm.com/

5. 一个属于你自己的 **域名**  
   示例：  
   👉 [本 Blog 域名](https://blog.novabrain.edu.kg)

6. 已配置好的 **DNS 服务**  
   推荐使用 Cloudflare  
   👉 https://www.cloudflare.com/

---

## 二、本地搭建 Blog（Hexo）

> 以下命令可在 **CMD / PowerShell / Terminal** 中执行

### 1. 安装 Hexo CLI

```bash
npm install -g hexo-cli
```
### 2. 初始化博客项目
```bash
hexo init myblog
# “myblog” 可替换为你自己的文件夹名称
cd myblog
npm install
```
> 执行完成后，一个最基础的 Hexo Blog 就已经在本地生成了。

## 三、安装并启用 Butterfly 主题
> Butterfly 是目前非常流行且维护活跃的 Hexo 主题之一，功能与颜值都在线。
1. 安装主题及依赖
在 博客根目录 下执行：
```bash
npm install hexo-theme-butterfly hexo-renderer-pug hexo-renderer-stylus --save
```
2. 修改主题配置
打开根目录下的 _config.yml，找到：
```yaml
theme: landscape
```
修改为
```yaml
theme: butterfly
```
保存即可。
## 四、将 Blog 上传至 GitHub
> Vercel 是通过读取 GitHub 仓库来自动构建和发布的，因此这一步是必须的。
1. 新建 GitHub 仓库
- 登录 GitHub
- 点击右上角 “+” → “New repository”
### 仓库名示例：my-blog
- 选择 Public
- 不要勾选 “Add a README”
- 点击 Create repository
2. 推送本地代码到 GitHub
> 在博客根目录执行：
```bash
git init
git add .
git commit -m "First commit"
git branch -M main
git remote add origin https://github.com/你的GitHub用户名/仓库名.git
git push -u origin main
```
## 五、在 Vercel 上部署 Hexo
1. 关于 Vercel 风控的说明
### 使用 GitHub 注册 Vercel 时，可能会遇到提示：
“您的账户需要进一步验证”
> 常见原因包括：
- GitHub 账号过新，几乎没有使用记录
- 注册或登录时使用了 高风险 IP / VPN
- 使用国内邮箱（如 qq.com、163.com）
- 填写风控表单时使用虚拟手机号
- 可使用 [ping0.cc](https://www.ping0.cc) 检测当前 IP 风险
2. 正式部署流程
- 登录 Vercel Dashboard
- 点击 “Add New…” → “Project”
- 选择刚才创建的 GitHub 仓库，点击 Import
- 构建配置说明：
  - Framework Preset：一般会自动识别为 Hexo
  - Output Directory：public（默认即可）
- 点击 Deploy
>[!success]
大约 1～2 分钟后，如果你看到页面顶部出现 🌈，说明部署成功了。
## 六、绑定自定义域名
1. 在 Vercel 中添加域名
- 项目页面 → Settings → Domains
- 输入你的域名，点击 Add
- 复制 Vercel 提供的目标域名
2. 在 Cloudflare 中配置 DNS
- 添加一条 CNAME 记录
- 目标指向刚才复制的 Vercel 域名
- 关闭 DNS Proxy（小黄云）
>[!note] 等待几分钟后即可上线。
