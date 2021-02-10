---
layout: post
title: 使用Jekyll和github搭建博客
category: Other
tags: 
    - Other
date: 2015-04-03
---
<p>经过两天的摸索，终于在github上成功搭建了一个简单的博客。对于一个懂一点git，会搭ruby环境，没做过网页的Android程序猿来说，能搭建起了，已经很满足了。</p>

<h5>搭建环境之前，首先要配置好ruby，bunlder</h5>

<p>新建文件夹</p>
```bash
$ mkdir myblog
```

<p>在根目录下添加一个Gemfile，输入以下内容</p>
```ruby
source 'https://rubygems.org'
gem 'github-pages'
```

<p>在根目录执行</p>
```bash
$ bundle install
```

<p>我的命令执行如下</p>
```bash
$ bundle install
Using RedCloth 4.2.9
Using i18n 0.7.0
Using json 1.8.2
Using minitest 5.5.1
Using thread_safe 0.3.5
Using tzinfo 1.2.2
Using activesupport 4.2.1
Using blankslate 2.1.2.4
Using hitimes 1.2.2
Using timers 4.0.1
Using celluloid 0.16.0
Using fast-stemmer 1.0.2
Using classifier-reborn 2.0.3
Using coffee-script-source 1.9.1
...
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
```

<p>至此，可以认为jekyll已经搭好了</p>

<p>之后的操作参考了这个帖子 <a href="http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html">搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门</a></p>
<p>参考上面的帖子，新建了_layout，_posts，index.html，先不要push到github</p>
<p>在根目录运行</p>
```bash
$ jekyll serve
```
<p>用浏览器打开<code>http://localhost:4000</code></p>
<p>如果能正常看到网页，整个blog就已经搭建成功了</p>
<p>虽然能够正常运行，但页面太简陋了，所以从github拷贝了一个模板，方法就是利用gitbub pages生成一个默认的页面，然后将这个页面修改成为模板。</p>

<p>上传到github的时候，只要把生成的静态网页上传就行</p>
```bash
$ cd _site
$ git init
$ git add --all
$ git commit "first commit"
$ git remote add origin "你的git Repositories地址"
$ git push -u origin master
```

<p>打开{username}.github.io，就能看到刚才写的博客</p>




