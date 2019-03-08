---
title: GitHub Pages + Hexo 搭建博客
date: 2018-11-01 08:29:03
categories:
- DevelopTools
tags:
- GitHub
- Hexo
- NexT
- 博客
---

在决定写博客之后，我们首要面临的问题就是如何完成个人博客的初始化工作。此处，个人选择了 GitHub Pages + Hexo 的博客搭建方案。对博客方案选择过程有兴趣的或者有自身明确需求的读客可以阅读 [博客的需求](#1) 和 [博客搭建的选择](#2) 两部分以决定是否阅读[本博客的方案](#3)，若决定使用与本博客同样的解决方案，可以直接阅读[本博客的方案](#3)。

 <!--more-->

### <span id="1">博客的需求</span>

对个人博客而言，我将需要具备的能力按重要性排列如下：

- 方便快捷编辑修改的能力
- 数据的存放
- 内容获取的便利性
- 其他方面



#### 方便快捷编辑修改的能力

我将方便快捷编辑修改的能力放置于个人博客需求的首位。

因为与其他网站不同，个人博客在追求形式美的同时，平时更多关注于内容的撰写与发表。具有较为理想的快速编辑与修改能力就显的很有必要，也减少了编写博客的阻力（编写能力弱无形中增加了写博客的惰性）。

其次，博客的内容在于他人的沟通交流或者自身理解程度的加深，需要回去修改或增加内容，所以快速定位博客及编辑修改的能力也要突出。

最后，编写功能操作越简单越好，推送发布的操作越少越好。降低每次写新博客或者编辑旧博客的复杂程度。



#### 数据的存放

对个人博客而言，数据存放是一件严肃认真的事情。数据的存放会包含很多方面，比如数据存放的位置，存放的平台，存放的格式等等；存放完还有数据是否自己可以完全掌控，是否容易迁移等等各种各样的问题。

对个人博客而言，一般而言，以文字为主，佐以少量图片或语音调色（可使用外链），因此一般数据量不大。访问的频率也不会太高。但写博客是一件长期的事情，所以要考虑数据存储的稳定性，数据备份迁移等行为不宜频率太高。

此处，我个人倾向于选择数据可以完全自己掌握的存储方式，如博客园，CSDN等博客网站，或如知乎，公众号等平台，首先会对你发表的内容进行审核，因此发表修改编辑有环节，不能所写所见；其次，数据导出或迁移甚至在无网络情况下，本地无法浏览更改；最后，这些数据的所有权归属于平台，因此在博客选择时我排查了上述类型的博客撰写方式。



#### 内容获取的便利性

个人博客的撰写目的当然是为了方便他人的阅读与交流，因此要方便他人的阅读，方便交流。当然，也可以通过获取方式对阅读人群进行初步筛选，如常见的新浪微博偏娱乐碎片化的信息记录交流；公众号稍微长一点，也是碎片化的信息；各种课程网站就比较长且完整。



#### 其他方面

除上述之外，可能还有很多原因或考虑促成你选择适合自己的博客。比如美观时尚要求，比如内容要求，经常分享自己的拍照视频等等各种各样的需求。按需选择适合自己的博客是促使可以长期坚持下去的首要条件，若是不合适的博客，在几次尝试撰写博客，或者在各大流行的博客间转换几次之后失去了兴趣，也就失去了写博客的初衷。因此，愿每个勇于尝试博客搭建的人都能很快选出适合的博客并坚持下去。



### <span id="2">博客搭建的选择</span>

该部分大部分内容会参考自[ONEGEE 博客 - 怎么选择和快速搭建个人博客](https://onegee.space/post/buildblog/) ，本人尝试过的方式会进行额外说明与补充。

该博主将博客按发布形式分为了三种：个人主页注册、静态网站生成与内容管理系统。



#### 个人主页注册

个人主页注册是指在现有的博客网站、论坛或社区上注册个人主页。优点是没有技术门槛，注册即用；拥有成熟的平台支撑，方便推广。缺点是风格单一，自定义程度低，还有许多形式与内容的限制。适合嫌麻烦不喜折腾而又不反感条条框框，对数据存储无感且不轻易迁移数据的人。



##### [SegmentFault](https://segmentfault.com/)

中文领域最大的编程问答交流社区平台，其所属杭州堆栈科技有限公司创立于2012年，目标是覆盖和服务中国软件开发者和 IT 信息从业者，充分利用在各个平台所能获得的各种技术创新机会为其开发产品应用和服务。

可以理解为中文的 StackOverFlow 社区，技术交流平台成熟。网站提供了文章专栏板块，有审核机制，支持 Markdown 文法、标签、评论、智能目录等。颜值中等的简洁风格。

因此，整体上 SegmentFault 平台计算机相关专业为主要阅读者。



##### [简书](https://www.jianshu.com/)

简书自身定位为国内最优质的创作社区，2013年4月上线公测版本，正式开放注册。任何人可以在其上进行创作，相互交流。内容偏重于文字。同支持 Markdown 文法、标签、评论功能。颜值中等干净。

简书创始人[简书个人主页](https://www.jianshu.com/u/y3Dbcz)，知乎ID[简叔](https://www.zhihu.com/people/linlis/activities)。对简书或其CEO个人有兴趣的可以去关注。

##### [知乎](https://www.zhihu.com/)

中文互联网知名知识社交平台，创立于2011年1月26日，产品形态模仿自美国类似网站 [Quora](https://www.quora.com/)。用户通过问答等交流方式建立连接，偏娱乐化和专业知识，目前该公司包括知乎、知乎群组、知乎日报与公益壹点通四款应用。

提供的文章版本可以用作于博客撰写，提供 Markdown 文法、标签、评论功能。颜值中等偏大气的风格。

对知乎用户获取较为方便，但非知乎用户获取困难。大部分互联网人士了解该平台。

##### [CSDN](https://www.csdn.net/)

CSDN 全称是 Chinese Software Developer Network，创建于1999年，自身定位是中国专业 IT 社区，为中国的软件开发者提供知识传播、在线学习、职业发展等全生命周期服务。官方数据，截止2018年6月，CSDN 拥有2500+ 万技术会员，论坛发帖数 1000+ 万，技术资源 700+ 万，博客文章1300+ 万，新媒体矩阵粉丝数量 430+ 万。

老牌技术论坛，支持 Markdown 文法、标签、评论功能，文章管理方式传统。但目前 CSDN 的交互体验比较差，常被吐槽。颜值正常偏下水平。

##### [博客园](https://www.cnblogs.com/)

博客园创立于2004年1月，面向开发者的知识分享社区。自身定位为开发者打造纯净的技术交流区，推动并帮助开发者通过互联网分享知识。

老牌技术论坛，申请博客需人工审核，上班时间10分钟左右。支持 Markdown 文法、标签、评论、RSS、相册、文件等功能，文章管理方式传统。Logo 有多种选择，颜值正常，老式网站风格。

##### 其他

注册形式的博客还有很多，如[网易](http://blog.163.com/)(很快将迁移至 [LOFTER](http://www.lofter.com))、[新浪](http://blog.sina.com.cn/)、[搜狐博客](http://blog.sohu.com/)等，甚至还有已经停止维护的博客网站，且大多数定位也不是技术类博客，此处没有介绍。此外，如微信订阅号、公众号，知乎问答、StackOverFlow或Quora、甚至百度贴吧等以问答形式完成博客的撰写也是不错的形式。



#### 静态网站生成

通常是指由 Jekyll、Hugo 或 Hexo 等技术生成静态网站，然后上传至 GitHub Pages、Coding Pages 等托管平台免费展示。具有一定的技术门槛，需要了解 Markdown 文法，简单 Linux 命令，域名解析，对要托管的平台如[GitHub](https://github.com/) 或 [Coding](https://coding.net/) 有一定了解。

该类型的博客撰写或修改流程大致如下

- 本地以特定表头格式写博客，放于指定文件夹中
- 执行命令快速生成完整的静态网站
- 通过 Git 管理工具将文件上传至代码托管平台

该种博客搭建方式具有搭建快速、自定义程度高、主题丰富、技术更新迭代快、社区活跃的优势，同时具有一定的入坑门槛，适合有一定技术基础或喜欢折腾的用户，不同技术配置间迁移成本低。



##### [Hexo](https://hexo.io/zh-cn/)

一种基于 Node.js 的快速、简洁、高效的博客框架，[GitHub代码库](https://github.com/hexojs/hexo)有 24k+ 的 Star（截止2018年11月12日）。安装过程顺利，配置、发布人性化，社区活跃，对技术不熟英文不好的人同样友善。主题多，选择空间大。可以通过插件形式支持博客所需功能。

##### [Hugo](http://gohugo.io/)

一种基于 Go 语言实现的站点生成器，[GitHub代码库](https://github.com/gohugoio/hugo)有30K+的 Star（截止2018年11月12日）。安装过程较为顺利，中文社区不是很活跃。主题多，适合有一定技术基础有更高品味要求的用户。

##### [Jekyll](https://www.jekyll.com.cn/)

GitHub 官方推荐的将纯文本转化为静态网站和博客的站点生成器。GitHub 联合创始人 Tom Preston-Werner 使用 Ruby 语言编写，在 [GitHub代码库](https://github.com/jekyll/jekyll)有35K+的 Star（截止2018年11月12日）。官方的加持，使得可以不依赖本地环境配置，直接在网站生成，本地环境配置较为麻烦。主题较好，相较于 Hexo 与 Hugo 较少。



#### 内容管理系统

内容管理系统指带有后台管理的博客系统，需要配置服务器、数据库以及域名管理，在此基础上安装内容管理系统。相较于静态网站生成而言，是动态博客，有前台后台之分，后台负责写作、发布、系统配置等。

这种方式具有贴心的后台管理功能，意味着具有出色的文章管理，相册管理，文件管理，而且在数据库基础上可以实现用户管理以及高清大图上传等，可以内置搜索、评论等常用功能。但同时，丰富的管理功能背后需要用户较高的技术基础，如 Web 相关的服务器知识，数据库知识等。与当下用户体验当道和扁平化时代相比，丰富又臃肿。

因此，对个人用户而言，使用该方式搭建博客，安全稳定，一次上手之后基本无需迁移，但可能需要有服务器的开销花费。若有多人维护，频繁更新的需求可以考虑该方式搭建博客。

##### [WordPress](https://cn.wordpress.org/)

一个开源的基于 PHP 和 MySQL 的个人发布系统。根源和开发可以追溯到2001年，社区活跃，遵循 GPL 协议。

具有较高的市场占有率，博客只是其功能之一，可以搭建企业级网站。中文友好，中文社区活跃。

##### [ghost](https://ghost.org/)

基于Node.js 实现的开源，旨在为新闻媒体构建开源解决方案。社区活跃，相比于 WordPress 而言，简洁大气，专为写作生产力的极致博客系统，便捷，可以随时随地撰写编辑博客，尤其在不同电脑上。WordPress 良好替换品，有一定搭建门槛。颜值在所有例子中最高（为颜值牺牲了一些功能）。



#### 建议

**新手村指南**：如果是新手，对于以上的技术门槛一窍不通，但是又想要主题精美的个人博客网站，建议从[Markdown](https://daringfireball.net/projects/markdown/)语言开始学起（半天入门，一天出师）。之后可以选择现有平台，简单上手，也可以稍微了解一些基本的命令行知识和 Git 操作，跟随各种教程，从生成静态网站入门快速搭建博客，完全不花钱。

对与我情况类似的读者，出身计算机相关专业或从事相关工作，可以考虑自己动手搭建。

**首推 hexo**。性价比最高，中文友好，快速上线，贴心配置，免费高颜值。

**其次 WordPress**。满足多人维护，资料繁多等需求，虽然门槛高较高，体量较大，且有额外花销，但稳定，可以对网址数据全掌握。

**最后**，内容高于形式，入坑需谨慎 。

个人博客最终选择了**GitHub Pages + Hexo + NexT** 的博客解决方案，博客地址：[码农驿站](https://chenwenjia1991.github.io/)

### <span id="3">本博客的方案</span>

个人搭建技术博客，对颜值要求中等，期望数据完全掌握，由于一个人维护，又不希望维护的成本过高，因此选定了 Hexo 技术方案。其丰富的插件支持足以满足我对个人博客的需求。

在托管平台选取时，天然选择了 GitHub，即程序员交友网站，方便阅读和沟通交流。

GitHub Pages + Hexo + NexT 的方案搭建过程如下，其中可能遇到的问题我会在相应位置提及，但整体而言，较少遇到，搭起来很快。

参考了较多他人的博客与官方文档，资料如下

- [GitHub + Hexo 搭建个人网站详细教程 - From 知乎](https://zhuanlan.zhihu.com/p/26625249)
- [Hexo 官网](https://hexo.io/zh-cn/)

#### 环境准备

这里的环境准备包括两部分，GitHub 的托管平台配置与本地环境的配置。与博客相关度不大的部分在这里介绍将会较为简略。

##### GitHub 配置

GitHub 是目前最流行的代码仓库，得到了很多大公司与项目的青睐，为使得项目更方便的被人理解，需要项目的介绍页面甚至完整的技术文档，于是 GitHub Pages 服务应运而生，其不仅可以方便的为项目建立介绍站点，也可以用来建立个人博客。

GitHub Pages 属于轻量级的博客系统，配置简单；支持 Markdown 文法，编辑简单迅速；无需自己搭建服务器，GitHub 给每个站免费提供了 300MB 的空间（对文字而言足够）；可以绑定自己的域名。

配置流程大致如下：

- 购买、绑定独立域名
- 配置和使用 GitHub

1. 注册账号
2. 本地安装 Git Bash
3. 配置 SSH Keys 设置实现免密登陆
4. 测试联通成功并添加账号等相关信息

- GitHub Pages 建立博客 - 注个人博客必须使用与 GitHub 用户名一样的名字，格式为 GitHubName.github.io
- 绑定域名到GitHub Pages

详细 GitHub Pages 配置过程可参考 使用 [GitHub Pages 建立独立博客 - beiyunyun的博客](http://beiyuu.com/github-pages)，若与我情况类似，重新建立了新的 GitHub 账号以搭建博客，可能需要了解[多个 GitHub 账号配置 SSH Key](https://www.jianshu.com/p/477444ada71e)。

##### Hexo 本地环境配置

Hexo 是基于 Node.js 的博客框架，因此需要首先安装 Node.js，[Node.js 的安装包下载地址](https://nodejs.org/en/download/)

安装成功后，通过命令行测试结果如下

```bash
~ $ node -v
v8.4.0
~ $ npm -v
6.1.0
```

Hexo 安装较为简单，命令如下所示

```bash
~ $ npm install hexo-cli -g
```

如上操作成功后，本地的环境基本准备完成，后续进行 Hexo 相关配置说明。



#### Hexo 配置

[Hexo 官网技术文档](https://hexo.io/zh-cn/docs/)详细说明了如何上手，强烈推荐快速浏览一遍，可以对 Hexo 使用有一个大概了解，知道有哪些功能，方便后续功能的添加维护和更新。

此处基本按照 Hexo 搭建的流程进行配置。

##### 建站

安装 Hexo 完成后，执行如下命令，Hexo 将在指定的文件夹中新建所需文件

```bash
~ $ hexo init <blog-folder>  # 初始化名为 blog 的博客，可自行设置博客名
~ $ cd <blog-folder>
<blog-folder> $ npm install
```

新建完成后，指定文件夹下的目录结构如下所示

```
.
├── _config.yml  # 网站的配置信息
├── package.json  # 应用程序信息，默认安装了 EJS，Stylus 和 Markdown Renderer，可自由移除
├── scaffolds  # 模板文件夹，Hexo 根据该文件建立文件
├── source  # 存放用户资源的地方，Markdown 和 HTML 文件被解析并放到 public 文件夹，其他被拷贝过去
|   ├── _drafts
|   └── _posts
└── themes  # 主题文件夹
```

网站的配置在 _config.yml 文件中，在此可以配置大部分的参数，常用及此次修改位置有网站、网址、目录、文章、分类&标签、日期时间格式、分页、扩展（包括主题）等几个部分，配置文件中对应修改如下。

###### 网站

```yaml
# Site
title: 码农驿站  # 网站名，会在标签页上显示
subtitle: 一枚码农的自述以供交流娱乐  # 副标题，网站名下面
description: Just For Fun  # 描述，主要用于SEO，告诉搜索引擎一个关于您站点的简单描述
keywords: Code
author: 陈文嘉
language: zh-Hans
# 应填写 Asia/Shanghai，填写 CN 会报错 TypeError: Cannot read property 'utcOffset' of null
timezone: Asia/Shanghai
```

###### 网址

```yaml
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://chenwenjia1991.github.io
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
```

###### 目录

```yaml
# Directory
source_dir: source  # 资源文件夹
public_dir: public  # 公共文件夹，用于存放生成的站点文件
tag_dir: tags  # 标签文件夹
archive_dir: archives  # 归档文件夹
category_dir: categories  # 分类文件夹
code_dir: downloads/code  # Include Code 文件夹
i18n_dir: :lang  # 国际化文件夹
skip_render:  # 跳过指定文件的渲染，可使用 glob表达式来匹配路径
```

Tips: 刚接触 Hexo，此部分一般不做修改。

###### 文章 ######

{% codeblock lang:yaml %}
# Writing
# 新文章的文件名称，此处修改问 年-月-日-title.md 的方式方便检索
new_post_name: :year-:month-:day-:title.md # File name of new posts
# 预设布局
default_layout: post
# 中英文间加入空格 - 建议文章中自己添加
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
#  代码块设置
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:
{% endcodeblock %}

###### 分类 & 标签

博客添加分类和标签页，参考了 [Hexo 使用攻略 - 添加分类及标签 From linlif 博客](https://linlif.github.io/2017/05/27/Hexo%E4%BD%BF%E7%94%A8%E6%94%BB%E7%95%A5-%E6%B7%BB%E5%8A%A0%E5%88%86%E7%B1%BB%E5%8F%8A%E6%A0%87%E7%AD%BE/)，添加新的页面也是如此，此处以添加 “categories” 页面为例，主要流程为

1. 创建 “categories” 页面并添加 type 属性

```bash
<blog_folder> $ hexo new page categories
```

成功后提示

```
INFO  Created: ~/Documents/blog/source/categories/index.md
```

正该 index.md 文件中，添加字段 type: "categories"

```yaml
<blog_folder> $ cat source/categories/index.md
---
title: categories
date: 2018-10-25 23:22:17
comments: false
type: "categories"
---
```

1. 文章中添加 “categories” 属性即可，如下

```yaml
---
title: blog_name
date: ****
categories:
- category_name
---
```

对于今后的文章，基本都会在撰写时填写分类&标签，可以通过修改 scaffolds/post.md 文件，在 tags: 上添加 categories: 后保存，之后执行 hexo new ** 产生的新文件就有分类选项了。

该文件是产生新博客时的模板，可以通过此文件设置默认的博客页面。

**可能产生的问题**

问题描述：GitHub Pages 结构混乱，而本地正常

解决方案：tag、category. 在主题的配置中必须选至少一个，即文档中存在的页面需要在配置文件打开，未打开出现上述异常。

###### 扩展

这里主要指主题，此处选择了 NexT 主题



##### NexT 主题配置

[NexT](http://theme-next.iissnan.com/) 主题现已支持十种语言，有四种外观，五套代码高亮主题，配置简单而丰富，已支持多种常见第三方服务，社区较为活跃，[GitHub代码库](https://github.com/iissnan/hexo-theme-next)有13K+的 Star（截止2018年11月13日）。官网的文档足够全面，基本可以满足个人博客的所有需求。

##### NexT 安装

Hexo 安装主题的方式简单粗暴，只需将主题文件拷贝至站点目录 themes 目录下，然后修改配置文件中的 theme 配置即可。

从刚才的 [GitHub代码库](https://github.com/iissnan/hexo-theme-next) 位置拉取最新代码，或下载稳定版代码解压缩到站点 themes 目录下，并将解压后的文件夹名改为 next。

###### NexT 启用

与其他 Hexo 主题启用方式一致，在 Hexo 配置文件中，theme 字段值修改为 next。

```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

切换主题后验证主题是否正确启用之前，最好使用 `hexo clean` 命令清除 Hexo 缓存。

###### 验证主题

启动 Hexo 本地站点并开启调试模式，命令是 `hexo s --debug` 。

服务启动过程中，注意观察命令行输出是否有任何异常信息，若碰到问题，这些信息可帮助更好的定位问题。

当命令行输出如下提示时，可以使用浏览器访问`http://localhost:4000`检查站点是否正常运行。

```bash
INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.
```

当看到的站点外观与下图所示类似时说明已成功安装 NexT 主题。这是默认的 Schema -- Muse。
![avatar](https://blogimages-1252423296.cos.ap-beijing.myqcloud.com/blog_pictures/11%E6%9C%88/11-1-NexT_Default.png "Next 默认样式")

###### 选择 Schema

Schema 是 NexT 提供的一种特性，以提供多种不同的外观，几乎所有配置均可以在不同 Scheme 之间功用。目前支持了四种 Scheme，在希望启用的 scheme 前去掉注释 `#`即可。

```yaml
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------
# Schemes
#scheme: Muse  # 默认主题，NexT 的最初版，黑白主调，大量留白
#scheme: Mist  # Muse 的紧凑版本，整洁有序的单栏外观
scheme: Pisces  # 双栏，小家碧玉似的清新，此处我采用的外观
#scheme: Gemini  # 与 Pisces 类似
```

###### 菜单设置

1. 设定菜单内容，对应 menu 字段，设置格式为 `item name: link || icon name`。其中`item name`是名称，并不会直接显示在页面上，而是用于匹配图标及翻译。`link`目标连接。`icon name`是 FontAwesome Icon 的名字。

   通过下述我的配置，即打开了`home`、`tags`、`categories` 与 `archives`，其他部分待打开进行设置。

   ```yaml
   # ---------------------------------------------------------------
   # Menu Settings
   # ---------------------------------------------------------------
   # When running the site in a subdirectory (e.g. domain.tld/blog), remove the leading slash from link value (/archives -> archives).
   # Usage: `Key: /link/ || icon`
   # Key is the name of menu item. If translate for this menu will find in languages - this translate will be loaded; if not - Key name   will be used. Key is case-senstive.
   # Value before `||` delimeter is the target link.
   # Value after `||` delimeter is the name of FontAwesome icon. If icon (with or without delimeter) is not specified, question icon will be loaded.
   menu:
     home: / || home  # 主页
     # about: /about/ || user  # 关于页面
     tags: /tags/ || tags  # 标签页
     categories: /categories/ || th  # 分类页
     archives: /archives/ || archive  # 归档页
     # schedule: /schedule/ || calendar
     # sitemap: /sitemap.xml || sitemap  # 站点地图
     # commonweal: /404/ || heartbeat  # 公益 404
   ```

2. 设置菜单的显示文本。上述步骤1中的名称并不会用于界面上的展示。Hexo 在生成时实用该名字查找对应的翻译，并提取显示文本。这些翻译文本放置在 NexT 主题目录下的 `languages/{language}.yml`（`{language}`为自己所使用的语言）。

   ```yaml
   menu:
     home: 首页
     archives: 归档
     categories: 分类
     tags: 标签
     about: 关于
     search: 搜索
     schedule: 日程表
     sitemap: 站点地图
     commonweal: 公益 404
   ```

3. 设定菜单项的图标，对应字段是`menu_settings`。

   ```yaml
   # Enable/Disable menu icons / item badges.
   menu_settings:
     icons: true
     badges: false
   ```

   注：在菜单图标开启的情况下，如果菜单项与菜单未匹配（没有设置或者无效的 Font Awesome 图标名字） 的情况下，NexT 将会使用 ? 作为图标。

##### 图标配置&头像修改

主题配置文件中的`favicon` 中，内容如下

```yaml
# ---------------------------------------------------------------
# Site Information Settings
# ---------------------------------------------------------------
# To get or check favicons visit: https://realfavicongenerator.net
# Put your favicons into `hexo-site/source/` (recommend) or `hexo-site/themes/next/source/images/` directory.
# Default NexT favicons placed in `hexo-site/themes/next/source/images/` directory.
# And if you want to place your icons in `hexo-site/source/` root directory, you must remove `/images` prefix from pathes.
# For example, you put your favicons into `hexo-site/source/images` directory.
# Then need to rename & redefine they on any other names, otherwise icons from Next will rewrite your custom icons in Hexo.
favicon:
  small: /images/blog_logos/16x16.png
  medium: /images/blog_logos/32x32.png
  apple_touch_icon: /images/blog_logos/apple-icon-180x180.png
  safari_pinned_tab: /images/blog_logos/logo.svg
  android_manifest: /images/blog_logos/manifest.json
  ms_browserconfig: /images/blog_logos/browserconfig.xml
```

主题配置文件中的`avatar` 中，内容如下

```yaml
# Sidebar Avatar
avatar:
  # in theme directory(source/images): /images/avatar.gif
  # in site  directory(source/uploads): /uploads/avatar.gif
  # You can also use other linking images.
  url: /images/blog_logos/avatar.gif
  # If true, the avatar would be dispalyed in circle.
  rounded: true
  # The value of opacity should be choose from 0 to 1 to set the opacity of the avatar.
  opacity: 1
  # If true, the avatar would be rotated with the cursor.
  rotated: true
```

其图像建议放入博客的 source/images 中，这样将来修改或更换该图片均不会失效。

网站 Logo、头像等的制作网上有很多工具，此处不做过多说明。

###### 集成评论模块

Hexo NexT 主题下，支持较多的评论系统，具体可参看[NexT 第三方服务集成](http://theme-next.iissnan.com/third-party-services.html)。除评论系统外，还可以通过第三方服务增加数据统计与分析功能，内容分享服务，搜索服务，数学公式显示，Facebook SDK 支持，Google 站点管理工具等服务。

官方推荐的评论模块有 DISQUES、Facebook Comments、HyperComments、网易云跟帖、LiveRe几种。参考了知乎问题[Hexo(NexT主题)评论系统哪个好？](https://www.zhihu.com/question/267598518)，根据其推荐，选择了 [Gitalk](https://github.com/gitalk/gitalk/blob/master/readme-cn.md) 作为此处评论系统。

Gitalk 是一个基于 GitHub Issue 和 Preact 开发的评论插件，使用 GitHub 登陆，支持多语言，支持个人或组织，无干扰模式（设置 `distractionFreeMode` 为 `true` 开启），支持快捷键（`cmd`|`ctrl` + `enter`）提交。

此处工作参考了博客[Hexo NexT 主题中集成 Gitalk 评论系统 - From asdfv1929‘s Home](https://asdfv1929.github.io/2018/01/20/gitalk/)，配置流程如下

1. GitHub 中注册新应用，[注册链接](https://github.com/settings/applications/new)，填写内容如下

   ```yaml
   Application name # 应用名称
   Homepage URL    # 网址，此处填 https://chenwenjia1991.github.io/
   Application description # 应用描述~
   Authorization callback URL # 授权回调网址， https://chenwenjia1991.github.io/
   ```

   点击注册后，保存 Client ID 和 Client Secret，在后面的配置中会用到

2. 主题配置文件中配置 Gitalk，具体参数含义可参考[详细参数列表](https://github.com/gitalk/gitalk#options)

   ```yaml
   # Gitalk
   # Introduction: https://github.com/gitalk/gitalk/blob/master/readme-cn.md
    gitalk:
      enable: true
      githubID: chenwenjia1991 # GitHub 账号
      repo: Gitalk-Comment # 存储评论的仓库，可新建或使用旧项目，只写项目名称即可，项目需开启 issue
      ClientID: ** # 上述记录的 Client ID
      ClientSecret: ** # 上述 Client Secret
      adminUser: chenwenjia1991 #指定可初始化评论账户
      perPage: 15 # 每页显示的最大评论数
      distractionFreeMode: true # 全屏遮罩
   ```

3. 通过 MD5 加密 ID 以解决 Label 长度不能超过50的问题，出现的具体问题描述及解决方案参考 [Gitalk Issues 115](https://github.com/gitalk/gitalk/issues/115)，将[JS脚本文件](https://github.com/blueimp/JavaScript-MD5/blob/master/js/md5.min.js)拷贝至 `source/js/src/md5.min.js`。

4. 主题中进行相关配置

   - 主题目录下`/layout/_third-party/comments`文件夹中

     - 创建 gitalk.swig 文件并添加以下内容

     ```css
     {% if page.comments && theme.gitalk.enable %}
       <link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
       <script src="https://unpkg.com/gitalk/dist/gitalk.min.js"></script>
     
       <script src="/js/src/md5.min.js"></script>  # <— 添加的上述脚本的路径
     
       <script type="text/javascript">
             var gitalk = new Gitalk({
               clientID: '{{ theme.gitalk.ClientID }}',
               clientSecret: '{{ theme.gitalk.ClientSecret }}',
               repo: '{{ theme.gitalk.repo }}',
               owner: '{{ theme.gitalk.githubID }}',
               admin: ['{{ theme.gitalk.adminUser }}'],
               id: md5(location.pathname),  # <— 使用上述脚本中的函数加密
               distractionFreeMode: '{{ theme.gitalk.distractionFreeMode }}'
             })
             gitalk.render('gitalk-container')
            </script>
     {% endif %}
     ```

     - 修改` index.swig`，在其末尾添加如下内容，将上述文件注册

     ```js
     {% include 'gitalk.swig' %}
     ```

   - 修改主题目录中 `/layout/_partials/comments.swig`文件，在 endif 语句前前添加如下内容

     ```js
     {% elseif theme.gitalk.enable %}
     <div id="gitalk-container"></div>
     ```

   - 主题目录下`/source/css/_common/components/third-party`文件夹中

     - 新建 gitalk.styl，内容如下

     ```css
     .gt-header a, .gt-comments a, .gt-popup a
     border-bottom: none;
     .gt-container .gt-popup .gt-action.is--active:before
     top: 0.7em;
     ```

     - 修改`third-party.styl`文件，末尾添加如下代码
     
     ```styl
     @import "gitalk" if hexo-config('gitalk.enable');
     ```

5. 重新生成静态网页并推送至GitHub Pages `hexo clean && hexo g && hexo d`

##### 站点统计

使用了[不蒜子统计](https://busuanzi.ibruce.info/)，配置简单，在主题配置文件中设置`busuanzi_count`的`enable`的值为`true`即可。

###### 数学公式支持 ######

主题配置文件中，设置`mathjax`的值为`true`即可。借助于 MathJax 显示数学公式。
配置文件中如下所示
{% codeblock lang:yaml %}
 # Math Equations Render Support
 math:
   enable: true

   # Default(true) will load mathjax/katex script on demand
   # That is it only render those page who has 'mathjax: true' in Front Matter.
   # If you set it to false, it will load mathjax/katex srcipt EVERY PAGE.
   # 默认为 ture。若为 false 每个页面均会载入该脚本（影响加载速度）
   per_page: true

   engine: mathjax
{% endcodeblock %}

在上述配置下，插入数学公式的博客 Front-matter 中均需要打开 `mathjax` 开关，如下所示。
{% codeblock lang:yaml %}
# Math Equations Render Support
---
title: test_mathjax
date: 2018-12-05 12:01:30
tags:
mathjax: true
--
{% endcodeblock %}

###### 图床

在图片较多时，将图片上传至 GitHub Pages 已不再合适，毕竟有300MB的大小限制，此时考虑图床。此处参考[国内外部分可用图床推荐对比-YiCH_ 简书](https://www.jianshu.com/p/9dbef7ae6e3b)，[嗯，图片就交给它了-少数派图床推荐](https://sspai.com/post/40499)等相关资料，选取了腾讯云 COS 做图床，存储空间 50GB，外网下行 10GB，基本够用 - 有防盗链设置。基本满足了我们的需求。

###### 站内搜索服务

官网提供了 Swiftype、微搜索、Local Search、Algolia几种，其中 Swiftype 与 Algoliia 开始收费项目，故舍去。 此处选择了 Local Search，两种实现方式，一是本地建立索引；二是采用第三方线上服务。同样修改完重新生成网站并推送。

```yaml
# 1. 安装 hexo-generator-searchdb，站点根目录下执行如下命令
blog_path $ npm install hexo-generator-searchdb --save
# 2. 编辑站点配置文件，新增如下内容
# Local Search
search:
   path: search.xml
   field: post
   format: html
   limit: 10000
# 3. 主题配置文件中，启动本地搜索，且可以修改相关配置
# Local search
local_search:
    enable: true
```

#### Markdown 本地编辑器

个人目前博客撰写环境为 MacOS Mojave 10.14.1，因此主要查找尝试了 Mac 下的 Markdown 文本编辑器，参考知乎问题 [Mac 上最好的 Markdown 文本编辑器是什么](https://www.zhihu.com/question/22700184)，其中 Typora 推荐数较高，优点是所见即所得，将写作和预览合二为一了。

##### [Typora](https://typora.io/)

Typora 是一款由 Abner Lee 开发的轻量级 Markdown 编辑器，适用于 OS X、Windows 和 Linux 三种操作系统，免费软件。与其他 Markdown 编辑器不同的是采用所见即所得的编辑方式实现了即时预览功能，也可切换至源代码编辑模式。

在编辑时，除了通过传统的 Markdown 代码的方式来实现富文本之外，Typora 支持通过菜单栏或者鼠标右键选取命令的方式来实现富文本，也支持通过快捷键的方式插入。Typora 也支持通过以 TeX 的格式来插入行间公式和行内公式。在完成编辑后导出文件时，Typora 支持以 PDF 或 HTML 的形式导出，如果安装了 [Pandoc](https://zh.wikipedia.org/wiki/Pandoc)，也能够以Word、RTF、MediaWiki、LaTeX 等形式导出。Typora 提供有几种主题，并支持通过自定义 CSS 的方式进行个性化定制。

目前博客的编写我使用该软件完成，主要由于该目录独立，又不需要在多台电脑间同步。

##### 印象笔记 & 马克飞象

[印象笔记](https://www.yinxiang.com/)是我目前使用的笔记记录工具，两个月前开始支持 Markdown 文法，但整体来说，还处于测试阶段，用户体验有待提高。[马克飞象](https://maxiang.io/)是基于印象笔记的 Markdown 编辑工具，优点是数据存入印象笔记，因此用户可直接通过印象笔记阅读，但无法修改，流畅性好，稳定；缺点马克飞象的维护团队只有一个人，因此相比较与其他产品未完全成熟，如目前不支持 SSL/HTTPS 协议，存在安全隐患，功能添加较为缓慢。马克飞象是付费产品，79元/年。若 Markdown 编辑使用频率高，且需要在不同场合设备间切换，对数据安全性要求没有太高，推荐该产品。

##### Markdown 写作注意事项

此处主要收集记录一些博客撰写过程中，应该注意的事项，可能会不断扩展，错误主要表现为静态网页无法产生。

1. 行内代码引用时尽量使用一个反引号而不是三个反引号，否则容易引起格式混乱；行内代码中不引用百分号或其他转义符时容易引发 Hexo 生成静态网页的错误，如下所示

   ```
   # 使用一个反引号后转义失败 {%code format error!%} 引发如下错误
   INFO  Start processing
   FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
   Template render error: (unknown path)
     unexpected end of file
       at Object._prettifyError (/Users/CWJ/Documents/Blog/blog_chenwenjia1991/node_modules/nunjucks/src/lib.js:36:11)
   ...
   ```

2. 通过 HTML 方式实现文内跳转

   ```html
   # 1. 定义 ID
   <span id="jump">跳转去的地方</span>
   # 2. 使用 Markdown 语法
   [点击跳转](#jump)
   eg.. [跳转至博客的需求](#1)效果如下
   ```

   [跳转至博客的需求](#1)

3. Markdown 引用（quote）中添加代码异常，如下所示
<img src="https://blogimages-1252423296.cos.ap-beijing.myqcloud.com/blog_pictures/11%E6%9C%88/11-3-markdown-quote-error-view.png" width="80%"/>
会发现代码冲出现了引用符号，引用块中代码下方的引用块分离。Markdown渲染正常，在 Hexo 生成的页面中不一致，可通过如下方法解决(第二种方式更优雅方便)。
同时，若此处出现类初始化函数的下划线会被 Hexo 渲染器识别为 HTML 元素再传递，Math 公式中同样浮现该问题，建议将渲染器更换。详情 [Hexo-Theme-Next Issues 826](https://github.com/iissnan/hexo-theme-next/issues/826)。
{% codeblock lang:bash %}
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-kramed --save
{% endcodeblock %}
* 将代码块结尾的三个反引号与下一行文字之间的空行删除，用四个空格实现
* 使用 Hexo 官网给定的另一种[插入代码块的方式](https://hexo.io/zh-cn/docs/tag-plugins.html#%E6%A0%B7%E4%BE%8B-1)，代码示例
```
{% blockquote %}
上述代码在 Quote 中效果如下
{% codeblock lang:python %}
# test for code in quote
print("Hello World!")
{% endcodeblock %}
{% endblockquote %}
注：Hexo quote 中引用代码块，注意其 Markdown 解析器的不一致。
``` 
{% blockquote %}
上述代码在 Quote 中效果如下
{% codeblock lang:python %}
# test for code in quote
print("Hello World!")
{% endcodeblock %}
注：Hexo quote 中引用代码块，注意其解析的不一致。
{% endblockquote %}
渲染器更换后，其渲染器配置改为在 '_config.yml' 文件的 `kramed` 域中进行。默认开启的智能引号转换在中文环境下会将英文引号自动转换为中文的，若不期望该行为发生，配置修改如下。参考 [Hexo Issuer 1981](https://github.com/hexojs/hexo/issues/1981)。
{% codeblock lang:yaml %}
# Kramed config
kramed:
  smartypants: false
{% endcodeblock %}

### 后记 ###

花费了一周多时间终于完成了本篇博客的撰写。一直在努力避免将一篇博客写的太细太长，不知不觉还是写了很长的篇幅，后续可能考虑拆分博客以为了更好的阅读体验，比如将 Hexo 相关部分独立。

感谢乐于分享的朋友，本文的很多工作也是在参考他们的博客下完成的；感谢读至此的朋友，完成这篇博客的阅读也花费了您不少的时间与耐心。

愿你我共同前行~
