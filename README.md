# 嵌入式笔记

一个记录嵌入式开发、单片机与硬件调试笔记的个人博客。

## 技术栈

- [Hugo](https://gohugo.io/) - 静态网站生成器
- [PaperMod](https://github.com/adityatelange/hugo-PaperMod) - 主题
- GitHub Pages - 部署托管

## 本地运行

```bash
git clone --recurse-submodules git@github.com:bamboosteps/myblog.git
cd myblog
hugo server -D
```

访问 `http://localhost:1313` 预览。

## 新建文章

```bash
hugo new posts/my-new-post.md
```

## 部署

推送到 `main` 分支后，GitHub Actions 自动构建部署到 GitHub Pages。
