### 我们的愿景

1. 根据场景提供AWS服务的最佳实践
1. 通过AWS的服务为切入点，分享特定IT领域的知识
2. 给用户提供hands on的IT知识和能力

### 1. 在mac上安装hugo

```
# wget https://github.com/gohugoio/hugo/releases/download/v0.96.0/hugo_0.96.0_macOS-64bit.tar.gz
# tar xvfz hugo_0.96.0_macOS-64bit.tar.gz
# cp -p hugo /usr/local/bin
# hugo version
```

其他平台安装包可从以下链接下载：
https://github.com/gohugoio/hugo/releases

### 2. 常用命令

- 本地测试：hugo server --port 8080
- 创建章节：hugo new --kind chapter 10_introduction/_index.md
- 创建文章：hugo new 10_introduction/10_about.md
- 编译内容：hugo -D -d docs

### 3. 更多

关于hugo的更多介绍和命令请参考：
1. https://hosting-hugo-content.workshop.aws/30_develop-content/20_create-online-workshop.html
2. https://gohugo.io/documentation/
