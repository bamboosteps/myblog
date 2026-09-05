# 嵌入式笔记

> 记录单片机、RTOS、通信协议与硬件调试过程中的踩坑与心得。

📖 **在线阅读**：[bamboosteps.github.io/myblog](https://bamboosteps.github.io/myblog/)

## 内容方向

- **各类单片机** — STM32 等主流 MCU 的入门与实战
- **通信协议** — UART、I2C、SPI 等总线协议的对比与选型
- **RTOS** — FreeRTOS 使用心得，裸机 vs RTOS 的取舍
- **硬件设计** — 电路调试、供电设计、信号完整性等硬件侧经验
- **调试技巧** — 没有高级工具时的土办法与排查思路

## 技术栈

| 组件 | 说明 |
|------|------|
| [Hugo](https://gohugo.io/) | 静态网站生成器（Extended 版本） |
| [PaperMod](https://github.com/adityatelange/hugo-PaperMod) | 简洁主题 |
| [GitHub Actions](.github/workflows/deploy.yml) | 自动构建 |
| GitHub Pages | 部署托管 |

字体方案：[Inter](https://rsms.me/inter/)（拉丁）+ [霞鹜文楷](https://github.com/lxgw/LxgwWenKai)（中文）

## 本地开发

**前置要求**：安装 [Hugo Extended](https://gohugo.io/installation/)（>= 0.165.0）和 Git。

```bash
# 克隆仓库（含主题子模块）
git clone --recurse-submodules git@github.com:bamboosteps/myblog.git
cd myblog

# 启动本地预览（含草稿）
hugo server -D
```

访问 `http://localhost:1313` 预览。

## 写作

```bash
# 新建文章
hugo new posts/my-new-post.md
```

在生成的 front matter 中填写标题、标签和摘要，正文使用 Markdown 编写。完成后将 `draft: true` 改为 `false` 即可发布。

## 部署

推送到 `main` 分支后，GitHub Actions 自动构建并部署到 GitHub Pages，无需手动操作。

## 目录结构

```
myblog/
├── content/posts/      # 文章 Markdown 源文件
├── assets/css/extended/ # 自定义样式
├── layouts/            # 自定义模板
├── themes/PaperMod/    # 主题（Git submodule）
├── hugo.toml           # 站点配置
└── .github/workflows/  # CI/CD
```

## License

文章内容保留所有权利。代码部分（配置、模板、样式）采用 MIT License。
