---
layout: post
title: "如何安装octopress"
date: 2013-06-03 12:08
comments: true
categories: octopress
tags: octopress
---

很久没有搞Blog了，在接触了GitHub、Markdown等等之后，突然想到能否使用Markdown来写Blog。在经过了一些搜索和浏览之后，决定使用octopress。下面把自己安装的过程记录下来。

<!--more-->

## 环境准备

- 安装Git

- 安装Ruby

	- 下载地址：<http://rubyinstaller.org/downloads/>

	- 该指南中安装文件为：[Ruby 1.9.3-p429](http://rubyforge.org/frs/download.php/76952/rubyinstaller-1.9.3-p429.exe)
	
	- 安装过程中，选择添加至环境变量`Path`

- 安装Development Kit

	- 下载地址：<http://rubyinstaller.org/downloads/>

	- 该指南中安装文件为：[DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe](https://github.com/downloads/oneclick/rubyinstaller/DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe)
	
	- 解压至目录`C:/RubyDevKit`

- 创建目录

	该指南中，创建了目录`C:\Users\nicktang\Documents\GitHub`

## 安装Octopress

	cd C:\Users\nicktang\Documents\GitHub
	git clone git://github.com/imathis/octopress.git nicktang1982.github.com
	cd nicktang1982.github.com
	ruby --version 

	cd C:/RubyDevKit
	ruby dk.rb init
	ruby dk.rb install

更换gem的更新源
	
	gem sources -a http://ruby.taobao.org/
	gem sources -r http://rubygems.org/
	gem sources -l

修改nicktang82.github.com目录下Gemfile文件，将 http://rubygems.org/ 修改为 http://ruby.taobao.org/

安装依赖项

	cd C:\Users\nicktang\Documents\GitHub\nicktang1982.github.com
	gem install bundler
	bundle install

安装默认主题

	rake install

本地预览：执行以下命令之后，通过 http://localhost:4000 访问

	rake preview

## 部署

### GitHub User/Organization pages

在github.com，创建仓库：nicktang1982.github.com

执行：
	
	rake setup_github_pages

这将：

- 提示填写你GitHub Pages repository的url：`git@github.com:nicktang1982/nicktang1982.github.com.git`

- Rename the remote pointing to imathis/octopress from ‘origin’ to ‘octopress’

- 添加你的repository为默认的origin remote

- 活动分支从`master`变更为`source`

- 按照你的repository，配置你blog的url

- 在`_deploy`目录中安装`master`分支

> 使用GitHub的Git Shell时，有错误。 使用了默认安装的git。

接着执行：

	rake generate
	rake deploy

> 将生成你的blog，拷贝生成的文件至`_deploy/`目录，添加至git，commit并push至master分支。

为blog commit source

	git add .
	git commit -m 'your message'
	git push origin source

> 以后可以使用GitHub客户端进行提交

### GitHub Project pages(gh-pages)

	rake setup_github_pages

1. Ask you for the repository url for your project:`git@github.com:nicktang1982/nicktang1982.github.com.git`

2. Rename the remote pointing to imathis/octopress from ‘origin’ to ‘octopress’

3. Configure your blog for deploying to a subdirectory.

4. Set up a gh-pages branch for your project in the _deploy directory, ready for deployment.

		rake generate
		rake deploy

另外建立一个仓库用于存放blog的source

	git remote add origin (your repo url)
	git config branch.master.remote origin


## 配置

修改`_config.yml`文件：

``` html _config.yml
	url: http://nicktang1982.github.io
	title: Nick Tang's Blog
	subtitle: IT SA Programmer
	author: Nick Tang
	simple_search: http://google.com/search
	description:
```

## 发布

### 新帖子

	rake new_post["Hello World"]

`markdown`文件将创建至目录`nicktang1982.github.com\source\_posts`

	---
	layout: post
	title: "Hello World"
	date: 2011-07-03 5:59
	comments: true
	external-url:
	categories:
	---

你可以添加多个类别

	categories: [CSS3, Sass, Media Queries]
	
	categories:
	- CSS3
	- Sass
	- Media Queries

### 新页面

	rake new_page[about]

会在`nicktang1982.github.com\source\about`目录下创建`index.markdown`文件

更改`nicktang1982.github.com\source\_includes\custom`目录下`navigation.html`文件，以包含该页面

``` html navigation.html
	<ul class="main-navigation">
	  <li><a href="{{ root_url }}/">Blog</a></li>
	  <li><a href="{{ root_url }}/blog/archives">Archives</a></li>
	  <li><a href="{{ root_url }}/about">About</a></li>
	</ul>
```

### 生成和预览

	rake generate   # Generates posts and pages into the public directory
	rake watch      # Watches source/ and sass/ for changes and regenerates
	rake preview    # Watches, and mounts a webserver at http://localhost:4000

### 发布

	rake generate
	rake deploy

为blog commit source

	git add .
	git commit -m 'your message'
	git push origin source


## 中文问题

设置环境变量

	set LANG=zh_CN.UTF-8 
	set LC_ALL=zh_CN.UTF-8

并且markdown文件均以UTF-8编码保存


## 自定义主题

部分主题修改来自<http://blog.tinyxd.me/>

修改、替换`sass\custom`下文件

``` scss _font.scss
	$sans: "Hiragino Sans GB", "Microsoft YaHei", "WenQuanYi Micro Hei", sans-serif;
	$serif: "Hiragino Sans GB", "Microsoft YaHei", "WenQuanYi Micro Hei", sans-serif;
	$mono: "Source Code Pro", monospace;
	$heading-font-family: 'Lato', sans-serif;
	$header-title-font-family: 'Lato', sans-serif;
	$header-subtitle-font-family: 'Lato', sans-serif
```
	
```scss _styles.scss
	article blockquote {
	  text-align:justify;
	  margin-bottom: 1em;
	  font-size: 1em;
}
```

修改`source\_includes\custom`下`head.html`文件，使用的Google Web Font

``` html head.html
	<!--Fonts from Google"s Web font directory at http://google.com/webfonts -->
	<!--
	<link href="http://fonts.googleapis.com/css?family=PT+Serif:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css">
	<link href="http://fonts.googleapis.com/css?family=PT+Sans:regular,italic,bold,bolditalic" rel="stylesheet" type="text/css">
	-->
	<link href='http://fonts.googleapis.com/css?family=Lato:400,700,400italic,700italic' rel='stylesheet' type='text/css'>
	<link href='http://fonts.googleapis.com/css?family=Source Code Pro:400,700,400italic,700italic' rel='stylesheet' type='text/css'>
```

## 添加分类

### 添加category_list插件

创建文件`plugins/category_list_tag.rb`

``` ruby category_list_tag.rb
	module Jekyll
	  class CategoryListTag < Liquid::Tag
	    def render(context)
	      html = ""
	      categories = context.registers[:site].categories.keys
	      categories.sort.each do |category|
	        posts_in_category = context.registers[:site].categories[category].size
	        category_dir = context.registers[:site].config['category_dir']
	        category_url = File.join(category_dir, category.gsub(/_|\P{Word}/, '-').gsub(/-{2,}/, '-').downcase)
	        html << "<li class='category'><a href='/#{category_url}/'>#{category} (#{posts_in_category})</a></li>\n"
	      end
	      html
	    end
	  end
	end
	
	Liquid::Template.register_tag('category_list', Jekyll::CategoryListTag)
```

### 添加aside

创建文件`source/_includes/asides/category_list.html`

``` javascript category_list.html
	<section>
	  <h1>Categories</h1>
	  <ul id="categories">
	    {% category_list %}
	  </ul>
	</section>
```

修改`_config.yml`文件


## 添加标签云

项目路径

<https://github.com/robbyedwards/octopress-tag-pages>
 
<https://github.com/robbyedwards/octopress-tag-cloud>

### octopress-tag-pages

复制 `tag_generator.rb` 到 `/plugins` 目录

复制 `tag_index.htm`l 到 `/source/_layouts` 目录

复制 `tag_feed.xml` 到 `/source/_includes/custom/` 目录

### octopress-tag-cloud

复制 `tag_cloud.rb` 到 `/plugins` 目录

复制以下代码到 `/source/_includes/custom/asides/tags.html`

``` javascript tags.html
	<section>
	  <h1>Tags</h1>
	  <ul class="tag-cloud">
	{% tag_cloud font-size: 60-165%, limit: 8 %}
	  </ul>
	</section>

```

修改 `_config.yml` 文件，添加 `custom/asides/tags.html` 导航栏

新建文章时，增加 `tags` ，使用方法与 `categories` 类似

	tags: [CSS3, Sass, Media Queries] 

## 添加多说

注册多说，获得`short_name`

在`_config.yml`添加

``` yml _config.yml
	default_asides: [asides/category_list.html, asides/recent_posts.html, custom/asides/recent_comments.html]
	
	# Duoshuo Comments
	duoshuo_short_name: nicktang
	duoshuo_show_comment_count: true
	duoshuo_asides_num: 10      # 侧边栏评论显示条目数
	duoshuo_asides_avatars: 0   # 侧边栏评论是否显示头像
	duoshuo_asides_time: 0      # 侧边栏评论是否显示时间
	duoshuo_asides_title: 0     # 侧边栏评论是否显示标题
	duoshuo_asides_admin: 0     # 侧边栏评论是否显示作者评论
	duoshuo_asides_length: 18   # 侧边栏评论截取的长度
```

在`source/_layouts/post.html`中`disqus`代码

``` javascript
	{% if site.disqus_short_name and page.comments == true %}
	  <section>
	    <h1>Comments</h1>
	    <div id="disqus_thread" aria-live="polite">{% include post/disqus_thread.html %}</div>
	  </section>
	{% endif %}
```

下添加

``` javascript
	{% if site.duoshuo_short_name and page.comments == true %}
	  <section>
	    <h1>评论</h1>
	    <div id="comments" aria-live="polite">{% include post/duoshuo.html %}</div>
	  </section>
	{% endif %}
```

创建`source/_includes/post/duoshuo.html`

``` html duoshuo.html
	<!-- Duoshuo Comment BEGIN -->
	<div class="ds-thread"></div>
	<script type="text/javascript">
	  var duoshuoQuery = {short_name:"{{ site.duoshuo_short_name }}"};
	  (function() {
	    var ds = document.createElement('script');
	    ds.type = 'text/javascript';ds.async = true;
	    ds.src = 'http://static.duoshuo.com/embed.js';
	    ds.charset = 'UTF-8';
	    (document.getElementsByTagName('head')[0] 
	    || document.getElementsByTagName('body')[0]).appendChild(ds);
	  })();
	</script>
	<!-- Duoshuo Comment END -->
```

在`_includes/article.html`中代码

``` javascript
	{% if site.disqus_short_name and page.comments != false and post.comments != false and site.disqus_show_comment_count == true %}
	 | <a href="{% if index %}{{ root_url }}{{ post.url }}{% endif %}#disqus_thread">Comments</a>
	{% endif %}
```

下添加

``` javascript
	{% if site.duoshuo_short_name and page.comments != false and post.comments != false and site.duoshuo_comments == true %}
	  | <a href="{% if index %}{{ root_url }}{{ post.url }}{% endif %}#comments">Comments</a>
	{% endif %}
```

创建`_includes/custom/asides/recent_comments.html`

``` javascript recent_comments.html
	<section>
	<h1>Recent Comments</h1>
	<ul class="ds-recent-comments" data-num-items="{{ site.duoshuo_asides_num }}" data-show-avatars="{{ site.duoshuo_asides_avatars }}" data-show-time="{{ site.duoshuo_asides_time }}" data-show-title="{{ site.duoshuo_asides_title }}" data-show-admin="{{ site.duoshuo_asides_admin }}" data-excerpt-length="{{ site.duoshuo_asides_length }}"></ul>
	{% if index %}
	<!--多说js加载开始，一个页面只需要加载一次 -->
	<script type="text/javascript">
	  var duoshuoQuery = {short_name:"{{ site.duoshuo_short_name }}"};
	  (function() {
	    var ds = document.createElement('script');
	    ds.type = 'text/javascript';ds.async = true;
	    ds.src = 'http://static.duoshuo.com/embed.js';
	    ds.charset = 'UTF-8';
	    (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(ds);
	  })();
	</script>
	<!--多说js加载结束，一个页面只需要加载一次 -->
	{% endif %}
	</section>
```