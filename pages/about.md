---
layout: page
title: About
description: 好好学习，天天向上
keywords: 程立涛, chenglitao521
comments: true
menu: 关于
permalink: /about/
---

当作为一名程序员的你看到这篇文章时，我想你已经猜到了我也是一名程序员。

毕业后一直从事Java开发，至今已有三年时间。

热爱这个行业，热爱coding,也从来没有想过以后会转行干点别的，因为觉得自己除了写bug好像其他手艺都不能养活自己。

So...还是好好写bug吧。哈哈！

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
