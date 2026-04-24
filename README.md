# newBlog

基于 `Hexo 5` 的个人博客项目，当前使用主题 `Butterfly`。

## 技术栈

- `Hexo 5.4.2`
- `Pug` 模板
- `Stylus` 样式
- `hexo-theme-butterfly`

## 本地开发

### 1. 安装依赖

```bash
npm install
```

### 2. 启动本地预览

```bash
npm run server
```

启动后访问：

- [http://localhost:4000/newBlog/](http://localhost:4000/newBlog/)

注意：

- 根路径 `http://localhost:4000/` 会返回 `404`
- 因为项目在 [_config.yml](/Users/zehong.li/newBlog/_config.yml:14) 中配置了 `root: /newBlog/`

### 3. 生成静态文件

```bash
npm run build
```

生成结果位于 `public/` 目录。

### 4. 清理缓存与重新生成

```bash
npm run clean
npm run build
```

## 常用命令

```bash
npm run server   # 本地预览
npm run build    # 生成静态文件
npm run clean    # 清理缓存
npm run deploy   # 部署到 GitHub Pages（需要先配置 deploy）
```

## 项目结构

```text
newBlog/
├── _config.yml              # Hexo 主配置
├── package.json             # 项目脚本与依赖
├── source/                  # 博客内容
│   ├── _posts/              # 文章
│   ├── about/               # 关于页
│   ├── categories/          # 分类页
│   ├── tags/                # 标签页
│   └── _data/               # 页面数据
├── themes/
│   ├── butterfly/          # 当前主题
│   │   ├── _config.yml      # 主题默认配置
│   │   ├── layout/          # Pug 模板
│   │   └── source/          # 主题静态资源
│   └── pure/               # 旧主题，当前不再启用
│       ├── _config.yml      # 主题配置
│       ├── layout/          # EJS 模板
│       └── source/
│           └── css/
│               ├── style.css
│               └── custom.css
└── public/                  # Hexo 生成后的静态文件
```

## 内容维护说明

### 文章维护

文章默认放在：

- `source/_posts/`

建议每篇文章至少包含这些 front-matter：

```yaml
---
title: 文章标题
date: 2026-04-23 18:00:00
categories:
  - 分类名
tags:
  - 标签1
  - 标签2
---
```

示例：

```markdown
---
title: Hexo 使用笔记
date: 2026-04-23 18:00:00
categories:
  - 前端
tags:
  - Hexo
  - 博客
---

正文内容从这里开始。
```

说明：

- `title`：文章标题
- `date`：发布时间
- `categories`：分类
- `tags`：标签

### 新增文章

可以直接在 `source/_posts/` 下新建 `.md` 文件，也可以使用 Hexo 命令：

```bash
npx hexo new "文章标题"
```

生成后的文章会出现在 `source/_posts/` 目录。

### 页面维护

页面内容目前放在：

- `source/about/`
- `source/categories/`
- `source/tags/`
- `source/links/`
- `source/repository/`

如果要新增独立页面，可以执行：

```bash
npx hexo new page "page-name"
```

例如：

```bash
npx hexo new page "about"
```

### 分类与标签

分类和标签主要来自文章 front-matter：

```yaml
categories:
  - 源码学习
tags:
  - vue3
  - 响应式
```

对应聚合页已经存在：

- `source/categories/index.md`
- `source/tags/index.md`

### 导航菜单维护

顶部/侧边导航来自主题配置：

- [_config.butterfly.yml](/Users/zehong.li/newBlog/_config.butterfly.yml:1)

对应字段：

```yml
menu:
  首页: / || fas fa-home
  归档: /archives/ || fas fa-archive
  分类: /categories/ || fas fa-folder-open
  标签: /tags/ || fas fa-tags
  友链: /links/ || fas fa-link
  关于: /about/ || fas fa-user
```

如果你新增了页面，也可以在这里补一个菜单入口。

### 友链与数据文件

项目中有一些页面数据放在：

- `source/_data/link.yml`
- `source/_data/gallery.yml`

Butterfly 友链页读取 `source/_data/link.yml`；旧 `pure` 主题使用过的 `links.yml` 可以保留作参考。

### 修改内容后的推荐流程

```bash
npm run clean
npm run build
npm run server
```

然后访问：

- [http://localhost:4000/newBlog/](http://localhost:4000/newBlog/)

## 主题与样式修改

当前主题为 `Butterfly`，项目级主题配置在：

- [_config.butterfly.yml](/Users/zehong.li/newBlog/_config.butterfly.yml:1)

当前主题源码位于：

- [themes/butterfly/](/Users/zehong.li/newBlog/themes/butterfly/_config.yml:1)

旧的 `pure` 主题文件仍保留在仓库中，主要入口如下：

- 主题配置：[\_config.yml](/Users/zehong.li/newBlog/themes/pure/_config.yml:1)
- 页面骨架：[layout.ejs](/Users/zehong.li/newBlog/themes/pure/layout/layout.ejs:1)
- 头部：[header.ejs](/Users/zehong.li/newBlog/themes/pure/layout/_common/header.ejs:1)
- 文章卡片：[article.ejs](/Users/zehong.li/newBlog/themes/pure/layout/_partial/article.ejs:1)
- 分页：[pagination.ejs](/Users/zehong.li/newBlog/themes/pure/layout/_partial/pagination.ejs:1)
- 原主题样式：[style.css](/Users/zehong.li/newBlog/themes/pure/source/css/style.css:1)
- 自定义覆盖样式：[custom.css](/Users/zehong.li/newBlog/themes/pure/source/css/custom.css:1)

建议：

- 优先改 `_config.butterfly.yml` 做主题配置，不要直接修改 `themes/butterfly/_config.yml`
- 如果只是调导航、侧栏、搜索、封面图、评论等开关，基本都在 `_config.butterfly.yml`
- 如果需要深度改模板，再修改 `themes/butterfly/layout/`

## 当前已做的 UI 调整

本轮已完成的重点改动：

- 主题已从 `pure` 切换到 `Butterfly`
- 新增项目级 Butterfly 配置
- 调整分类、标签、友链等页面 front-matter 以适配 Butterfly
- 保留旧 `pure` 主题文件，便于后续参考或回退

## 部署说明

当前仓库远程地址为：

- [https://github.com/lizehongss/newBlog](https://github.com/lizehongss/newBlog)

主配置中已经使用了 GitHub Pages 子路径形式：

- `url: http://lizehongss.github.io/newBlog`
- `root: /newBlog/`

这说明项目是按“仓库名页面”方式部署的，目标地址通常是：

- [https://lizehongss.github.io/newBlog/](https://lizehongss.github.io/newBlog/)

### 方案一：使用 Hexo deploy 部署到 GitHub Pages

这是最贴近当前项目结构的方式。

#### 1. 安装部署插件

```bash
npm install hexo-deployer-git --save
```

#### 2. 修改根配置中的 deploy

编辑 [_config.yml](/Users/zehong.li/newBlog/_config.yml:78)，把：

```yml
deploy:
  type: ''
```

改成：

```yml
deploy:
  type: git
  repo: https://github.com/lizehongss/newBlog.git
  branch: gh-pages
```

#### 3. 生成并部署

```bash
npm run clean
npm run build
npm run deploy
```

#### 4. 在 GitHub Pages 中确认发布分支

GitHub 仓库设置中：

1. 打开 `Settings`
2. 进入 `Pages`
3. Source 选择 `Deploy from a branch`
4. Branch 选择 `gh-pages`

部署完成后，一般通过下面地址访问：

- [https://lizehongss.github.io/newBlog/](https://lizehongss.github.io/newBlog/)

### 方案二：手动发布 `public/` 到静态托管

如果你不想使用 `hexo deploy`，也可以手动发布 `public/` 目录。

流程：

1. 执行 `npm run clean && npm run build`
2. 将 `public/` 下生成的静态文件上传到托管平台
3. 确保托管路径对应 `/newBlog/`

适用平台：

- GitHub Pages
- Vercel
- Netlify
- Nginx 静态站点

### 部署前检查项

每次部署前建议确认：

- [_config.yml](/Users/zehong.li/newBlog/_config.yml:14) 中的 `url` 和 `root` 是否正确
- 本地访问 [http://localhost:4000/newBlog/](http://localhost:4000/newBlog/) 是否正常
- 文章、分类、标签页是否都能正常打开
- 主题改动是否已经重新执行 `npm run build`

### 如果部署后资源路径错乱

最常见原因是 `root` 配置不匹配。

当前项目是：

```yml
url: http://lizehongss.github.io/newBlog
root: /newBlog/
```

如果你未来换成自定义域名并挂在站点根路径，需要把它改成：

```yml
url: https://your-domain.com
root: /
```

否则页面可能会出现：

- CSS/JS 加载失败
- 图片 404
- 首页能打开但子页面样式丢失

## 预览时的注意事项

- `npm run server` 是前台进程，终端关闭后服务会停止
- 如果浏览器突然无法访问本地页面，通常是本地预览进程已经退出，重新执行：

```bash
npm run server
```

## 后续建议

- 补充部署方式，例如 GitHub Pages 或其他静态托管平台
- 增加一份文章发布流程说明
- 如果后续持续改主题，建议把 `custom.css` 进一步拆分为：
  - `mobile.css`
  - `pagination.css`
  - `article.css`
