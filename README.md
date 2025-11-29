# [ProgrammerNotes](https://github.com/One0neSix6/ProgrammerNotes)

这是 [个人博客](blog.shangwanjun.com) 的源码仓库，使用 **Hexo** 作为静态博客框架，通过 **Netlify** 构建部署，并使用 **Cloudflare** 进行全站加速与域名托管。文章与资源通过 **GitHub** 进行版本管理。

---

## 🛠 技术栈

| 模块           | 技术                                                         |
| -------------- | ------------------------------------------------------------ |
| 博客编写       | **Typora + PicGo**                                           |
| 博客框架       | **Hexo**（Node.js 静态博客生成器）                           |
| 代码托管       | **GitHub**                                                   |
| 自动构建部署   | **Netlify**（自动构建 + CI/CD）                              |
| CDN / 域名解析 | **Cloudflare**                                               |
| 主题           | [hexo-theme-matery](https://github.com/blinkfox/hexo-theme-matery) |
| 对象存储       | **腾讯云**                                                   |

---

## 🚀 构建与部署流程

整套博客系统的构建与发布过程如下：

1. **写作与开发**
   - 在本地使用 Hexo 撰写文章、调试主题
   - 生成的静态资源由 Hexo 输出到 `public/`

2. **推送到 GitHub**
   - 所有 Markdown 内容、配置文件、主题、自定义脚本等均托管在 GitHub 仓库中

3. **Netlify 自动部署**
   - GitHub 推送后触发 Netlify Hook
   - Netlify 会自动构建
   - 构建完成后自动更新生产环境
   
4. **Cloudflare 全站加速**
   - 域名 `blog.shangwanjun.com` 指向 Netlify
   - Cloudflare 提供 CDN、DNS、SSL、缓存优化等功能
   - 保证访问速度与稳定性

## ✨ 功能特点

- 使用 PicGo 一键上传图片
- 使用 Hexo 生成静态页面
- 支持代码高亮 / 文章分类 / 标签
- 使用 Netlify 自动部署，零运维
- Cloudflare CDN 全球加速
- GitHub 保存全部文章历史记录
- 支持自定义主题与插件扩展

------

## 📬 联系方式

如需交流或反馈，可访问博客联系站长：

- 博客主页：**https://blog.shangwanjun.com**

------

## 📝 License

```txt
MIT License

Copyright (c) 2025 One0neSix6

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

