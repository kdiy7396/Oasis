---
title: First blog 
date: 2026-01-03 18:00:00
tags: [生活，碎碎念]
categories: [日常］
---
# blog部署日记
这是我第一个blog，在AI的帮助下，我成功的部署了，如你所见，blog页面依旧比较原始。相信我，它会慢慢发展起来的。
## 准备工作
部署这个blog需要用到以下软件及网站：
1. [Github]账号(https://github.com/)
2. 使用Github账号登录[vercel](https://vercel.com/)
3. 下载并安装 [Node.js](https://nodejs.org/) (选 LTS 版本)
4. 下载并安装 [Git](https://git-scm.com/)
5. 属于你的域名,eg.[本blog的域名](https://blog.novabrain.edu kg)
6. 配置好DNS,eg.[cloudflare](https://dash.cloudflare.com/login)

## 让我们开始吧
### 1.本地搭建blog
```markdown
[!abstract]CMD或PowerShell执行命令安装hexo
```
``` bash
npm install -g hexo-cli

# ==== 初始化博客 ====
hexo init myblog
# "myblog"文件名更改成你的
cd myblog
npm install
```
### 2.安装主题（Butterfly)
```markdown
[!abstract]换上大名鼎鼎的Butterfly 主题，让你的Blog更美观。
```
```bash
# === 在博客根目录下执行 ===
npm install hexo-theme-butterfly hexo-renderer-pug hexo-renderer-stylus --save

```
```markdown
[!abstract]安装完后，您需要修改根目录下的_config.yml 文件。
```
```bash
# 修改 theme参数
theme: landscape 
# 改为
theme: butterfly
```
### 3.上传代码到 GitHub
```markdown
[!abstract] Vercel 是通过读取 GitHub 仓库来自动发布的，所以我们需要把代码传上去。
- 先在 GitHub 新建仓库：
	- 登录 GitHub，点击右上角 + -> New repository。
	- 起个名字（比如 my-blog），设为 Public (公开)，不要勾选 "Add a README"。
	- 点击 Create repository。
- 在本地新建的文件夹根目录推送代码：
```
```bash
git init
git add .
git commit -m "First commit"
git branch -M main
# 下面这行指令，在 GitHub 仓库创建成功的页面里能直接复制到
git remote add origin https://github.com/$github用户名/$仓库名.git
git push -u origin main

```

### 4.在 Vercel 上部署
#### **使用Github账号登入vercel时，可能遇到风控“注册vercel时报错“您的账户需要进一步验证。 请填写此表格以获取更多帮助。”**
[! notes]- 遇到风控检查以下内容
```markdown
1. GitHub 账号是否为新注册或未创建仓库等使用迹象；
2. IP 地址风险：如果您注册时开启了代理（VPN），且该代理 IP 被其他人滥用过，Vercel 会连带封锁，可以使用[ping0.cc](https://www.ping0.cc)检测一下；
3. 邮箱后缀：某些国内邮箱（如 163, qq）容易被风控系统标记；
4. 风控后填写表格时不要使用接码平台或GV等虚拟号，使用真实的手机号，手机号是vercel风控检查的重要手段之一。
```
#### 1. 登录 [Vercel Dashboard](https://vercel.com/dashboard)。

#### 2.点击 "Add New..." -> "Project"。
#### 3.在左侧列表中找到您刚才上传的 GitHub 仓库 (my-blog)，点击 "Import"。
[! notes ] 你可能需要再登陆一下Github
#### 4.配置构建选项：
- Framework Preset (框架预设)：一般vercel 会自动识别Hexo，如未识别手动选择即可
- Output Directory: 默认为 public (不用改)。
#### 5.点击 "Deploy" 按钮。
[! notes ] 约一两分钟后，当你看到屏幕上方出现🌈，说明你的blog上线了。
### 5.绑定域名
[!notes] 如果xxx.vercel.app未被屏蔽可以直接使用，当然也可以换个域名，用自己的域名。
- 在 Vercel 的项目页面 -> **Settings** -> **Domains**
- 输入您购买的域名，点击 Add，复制生成的域名。
- 去[cloudflare](https://dash.cloudflare.com/login)添加一个CNAME记录，目标内容填写上步生成的域名，取消
[!notes]- DNS代理
```markdown
就是小黄云
```
- 等几分钟，你的blog应该可以直接访问了。
