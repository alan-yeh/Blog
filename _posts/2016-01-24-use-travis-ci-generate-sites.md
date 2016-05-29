---
layout: post
date: 2016-01-24
title: 利用Travis CI生成网页，支持Tags/Categorys
tags: Jekyll
---
　　现在大多使用Jekyll，并且支持Tags/Categorys的博客，都是将所有博客标题链接放到同一个页面中。博客量少的时候，弊端并不明显；但是博客量大的时候，同一个页面下的博客标题太多，就会给人一种目不暇接的感觉。大多数人第一时间想到就是利用分页。

　　想要将Tags/Categorys生成分页功能，有几个思路。由于jekyll在运行时，将所有博客生成静态页面，利用query param来分页显示博客，虽然可行，但不可取（因为虽然可以利用javascript动态隐藏一些标题，但实际上还是将所有标题放在同一个页面了）。利用自定义插件生成额外的静态页面，以支持Tags/Categorys的真正分页效果，是比较好的方案，而且Jekyll也支持[使用自定义插件](http://jekyllrb.com/docs/plugins/)，但很可惜，GitHub Pages却不支持。

　　[GitHub Pages](https://pages.github.com/)是由[Jekyll](https://github.com/jekyll/jekyll)提供技术支持的，但考虑到安全因素，所有的Jekyll自定义插件都被禁用了(禁用所有存放在_plugins目录下的插件，但pages是支持部分使用gem加载的插件的)。Jekyll官方提供的解决方案是先在本地生成静态页面，然后将生成好的页面Commit到GitHub上。但是这样做的体验并不好，所以Jekyll的[GitHub Issues](https://github.com/jekyll/jekyll/issues)下有很多discussion关于要求Jekyll提供官方支持Tags、Category的插件，以支持更好体验的博客系统。但是2年过去了，官方仍然没有提供相关插件。于是，有的网友就利用一些『奇技淫巧』来实现自动生成分页的功能。

　　[Travis CI](https://travis-ci.org/)，是一个在线的、分布式的持续集成服务，用来构建及测试在GitHub托管的代码。Travis CI会在你每次提交代码的时候，自动运行一些脚本。利用这个特性，我们可以在Travis CI上，利用脚本将托管在GitHub的博客生成静态页面，然后再提交到另一个代码仓库中（也就是USERNAME.github.io）上，这样就可以自动完成布署了。

##步骤
###.travis.yml文件
　　在根目录添加.travis.yml。.travis.yml用于配置项目，告诉Travis CI项目语言是什么，需要的环境，执行的脚本等等。在这个文件中，还可以添加一些其它的环境变量。利用Travis CI生成网页之后，我们需要将生成好的文件提交到你的GitHub下的USERNAME.github.io仓库，这样就可以完成动态生成了。提交代码需要权限，这里使用了[GitHub Token](https://github.com/settings/tokens)。明文放置GitHub Token是不安全的，所以需要将它加密一下。

```ruby
# 进入你项目所在目录
$ cd YOUR_BLOG_DIR

# 生成Travis CI密文，其中USERNAME是你的GitHub的用户名，GIT_EMAIL是你的GitHub所用的邮箱，YOUR_TOKEN是在上面生成的用户凭证
$ travis encrypt ‘GIT_NAME="USERNAME" GIT_EMAIL="YOUR_EMAIL" GH_TOKEN=YOUR_TOKEN’
```
　　以上具体.travis.yml配置可以参考[源代码](https://github.com/Poi-Son/jasper/blob/master/.travis.yml)。

### Rakefile
　　在.travis中，执行Rake脚本下的的site:deploy，所以在Rakefile，需要实现相关的生成网站逻辑。

　　具体思路大概是这样的。首先，Travis CI会clone一份代码仓库出来，同时根据.travis.yml文件为我们将环境搭建好。接着我们利用在.travis.yml中配置的Token，clone一份YOURUSERNAME.github.io的代码到项目生成的路径。然后执行`jekyll build`命令，生成静态html文件到YOURUSERNAME.github.io中，然后`git commit`到你的代码仓库，完成整个过程。

　　以上代码可以参考[源代码](https://github.com/Poi-Son/jasper/blob/master/Rakefile)

## 总结
　　利用Travis CI，现在仅需要将你的代码提交到GitHub上，然后稍等几份钟之后，你就可以看到USERNAME.github.io的代码已经更新了，自然你的博客也就更新了。

　　GitHub上的[jasper](https://github.com/Poi-Son/jasper)项目是通过改造原作者[Fábio Madeira](https://github.com/biomadeira)的[jasper](https://github.com/biomadeira/jasper)项目而来，感谢作者。