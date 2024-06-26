<!--
    需要填充的占位符：
    
    README.md
    
        数据库中文系列文档：文档中文名
        {nameEn}：文档英文名
        {urlEn}：文档原始链接
        db-docs：域名前缀
        飞龙：负责人名称
        wizardforcel：负责人 Github 用户名
        562826179：负责人 QQ
        db-docs-zh：ApacheCN 的 Github 仓库名称
        db-docs-zh：DockerHub 仓库名称
        db-docs-zh：PYPI 包名称
        db-docs-zh：NPM 包名称
    
    CNAME
    
        db-docs：域名前缀

    index.html
    
        数据库中文系列文档：文档中文名
        #DAA520：显示颜色
        db-docs-zh：ApacheCN 的 Github 仓库名称

    asset/docsify-footer.js
    
        db-docs-zh：ApacheCN 的 Github 仓库名称
-->

# 数据库中文系列文档

> 协议：[CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/)
> 
> 真相一旦入眼，你就再也无法视而不见。——《黑客帝国》

* [在线阅读](https://db-docs.flygon.net)

## 下载

### Docker

```
docker pull apachecn0/db-docs-zh
docker run -tid -p <port>:80 apachecn0/db-docs-zh
# 访问 http://localhost:{port} 查看文档
```

### NPM

```
npm install -g db-docs-zh
db-docs-zh <port>
# 访问 http://localhost:{port} 查看文档
```

## 赞助我

![](https://img-blog.csdnimg.cn/20200112005920729.png)
