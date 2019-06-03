---
title: Hexo-NexT 博客 note 与 tab 使用
date: 2019-06-03 15:24:02
categories:
 -DevelopTools
tags:
 - Hexo
 - NexT
 - 博客
---

在完成的博客网站上，随内容的丰富和完善，对展示内容的格式有进一步的需求和完善。本篇主要介绍 `Hexto NexT` 中的 `note` 与 `tab` 标签的使用，实现不同风格的提示块与选项卡功能。

提示块适用于不同重要程度的内容记录或展示；选项卡功能对于不同版本的代码展示有更好的展示效果。

详细说明如正文所示。

<!--more-->

### `note` ###
参考 [`Hexo NexT` 官方文档](https://theme-next.iissnan.com/tag-plugins.html)，该样式出现在 [Bootstrap 的官方文档](https://getbootstrap.com/)中。

在主题的配置文件中内容如下，对风格样式的调整在此处进行。
{% codeblock lang:yaml %}
 # Note tag (bs-callout).
 note:
   # Note tag style values:
   #  - simple    bs-callout old alert style. Default.
   #  - modern    bs-callout new (v2-v3) alert style.
   #  - flat      flat callout style with background, like on Mozilla or StackOverflow.
   #  - disabled  disable all CSS styles import of note tag.
   style: simple
   icons: false
   border_radius: 3
   # Offset lighter of background in % for modern and flat styles (modern: -12 | 12; flat: -18 | 6).
   # Offset also applied to label tag variables. This option can work with disabled note tag.
   light_bg_offset: 0
{% endcodeblock %}

使用方式如下
```html
{% note class_name %} Content (md partial supported) {% endnote %}
```

其中，`class_name` 可以是下标中的一个值：
> default
> primary
> success
> info
> warning
> danger

效果展示如下
{% note %} The content Without define Class Name {% endnote %}

{% note default %} The content Without define `default` {% endnote %}

{% note primary %} The content define `primary` {% endnote %}

{% note success %} The content define `success` {% endnote %}

{% note info %} The content define `info` {% endnote %}

{% note warning %} The content define `warning` {% endnote %}

{% note danger %} The content define `danger` {% endnote %}

### `tab` ###
此处参考 [`NexT 主题官方文档`](https://theme-next.org/docs/tag-plugins/tabs)。更多详细的用法可以参考此文档。

在主题的配置文件中内容如下，对风格样式的调整在此处进行。
{% codeblock lang:yaml %}
 # Tabs tag.
 tabs:
   enable: true
   transition:
     tabs: false
     labels: true
   border_radius: 0
{% endcodeblock %}

使用方式如下
```html
{% tabs Unique name, [index] %}
<!-- tab [Tab caption] [@icon] -->
Any content (support inline tags too).
<!-- endtab -->
{% endtabs %}
```

其中参数说明如下
> * `Unique name` 表示标签块标签（ tabs block tag）；
> * `index` 表示起始显示标签页，和 `Unique name` 用逗号隔开；
> * `Tab caption` 表示标签页名；
> * `icon` 表示 [FontAwesome](https://fontawesome.com/icons?d=gallery) 符号，以 `@` 开始，和 `Tab caption` 用空格隔开；
> * 每一对 `{% tabs %}` 和 `{% endtabs %}` 之间表示一个标签块；每一对 `<!-- tab -->` 和 `<!-- endtab -->` 之间表示一个标签页，每个标签块可以有多个标签页；
> * 标签块名需不一致，否则影响实现。

实例如下
```html

{% tabs 测试 %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}

```

实际效果如下所示

{% tabs 测试 %}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}


### 小结 ###
本文主要介绍了在博客撰写过程中使用到的 `Hexo NexT` 内置标签 `note` 与 `tab` 的使用，大大丰富了博客的样式，且使用方便简洁。