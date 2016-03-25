---
layout: post
title: "搭建博客"
description: ""
category: blog
tags: [jekyll, css]
---
一直想写一个博客，来来回回换了好几个地方，慢慢也荒废了，后来看到 Jekyll，简单看了一下，并在 2015 年简单搭建了起来，今天重新配置了一下，部分记录如下。（后续再优化）

### 必须材料如下：

#### 1. 官网

http://jekyllrb.com/ 基本照做即可

#### 2. Github

在 github 上新建一个约定的项目，然后把本地的 jekyll 工程上传即可

#### 3. 域名

Github 默认是 `github.io` 的二级域名，可以自己绑定域名到该域名，参见 CNAME

#### 4. 预览

搭建完成后，在本地目录，运行 `jekyll server --watch`（--watch 保证有改动后自动编译），然后在浏览器登录 `http://localhost:4000/` 即可

### 遇到的其他问题

#### 1. Theme

找可用的主题用了很久，最后暂时选择了 dbyll，更新主题的方式，只需要把对方的 git 项目 clone 到本地，然后替换 _post 目录下的所有博客正文，修改自己的配置就可以了。

#### 2. 修改配置

1. 在运用 dbyll 后，修改自己的头像，发现原来的 css 是直接连接的外链的图片，于是换了自己的本地图片到 `asset/media/` 下，使用方式是{/assets/media} 注意是 assets。
2. 结果图片太大了，于是又修改 `asset/css/style.css` 文件，新加了侧边栏的: `#sidebar-avator`，设置了 width、height。在这里还查了 css 名字前面的 .(id) 和 #(clsass) 的区别。
3. 同样是 css，对于图片来说，`border-radius` 是四个角的圆角的值，正方形显示圆形的话，要用 50%，圆角用 px。
4. 开始时主页的 post 显示不出来，运行时提示：
    
    > Deprecation: You appear to have pagination turned on, but you haven't included the `jekyll-paginate` gem. Ensure you have `gems: [jekyll-paginate]` in your configuration file.
    
    这时需要运行:
    
    ```
    sudo gem install jekyll-paginate
    ```

    然后修改 _config，增加：
    
    ```
    gems: [jekyll-paginate]
    ```
5. 想快速新建 post，发现 Jekyll-Bootstrap 有一个工具: Rakefile，copy 过来之后，直接运行：
{% highlight bash %}
rake post title="titlename"
{% endhighlight %}

#### 3. 最后，学习了 MIT 开源协议

MIT 是和 BSD 一样宽范的许可协议，作者只想保留版权，而无任何其他了限制。也就是说，你必须在你的发行版里包含原许可协议的声明，无论你是以二进制发布的还是以源代码发布的。

还有一些修改 markdown 出现的问题，后期统一再整理 markdown 相关问题吧。

