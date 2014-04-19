---
layout: post
title: "服务器端使用headless、watir-webdriver、nokogiri抓取ajax数据"
date: 2014-04-05 21:26:48 +0800
comments: true
categories: ruby
keywords: "watir-webdriver, headless, nokogiri, ajax抓取, 服务器端部署watir-webdriver"
description: "如何抓取ajax返回数据, 服务器断部署watir-webdirver"
---

最近一个项目中需要去别的站点上扒下来些数据，但这个站的数据是ajax加载回来的。Nokogiri仅仅是个解析工具不能模拟浏览器的动作，所以要抓取ajax数据必须找个能模拟浏览器点击动作的工具，watir-webdriver就可以帮我们完成这个工作。

watir-webdriver是一款能模拟多种浏览器动作的自动化测试工具。例如填写浏览器表单并提交。

{% codeblock lang:ruby %}

require 'watir-webdriver'

b = Watir::Browser.new
b.goto 'bit.ly/watir-webdriver-demo'
b.text_field(:id => 'entry_0').set 'your name'
b.select_list(:id => 'entry_1').select 'Ruby'
b.select_list(:id => 'entry_1').selected? 'Ruby'
b.button(:name => 'submit').click
b.text.include? 'Thank you'

{% endcodeblock %}

上面这段代码就会自动的填好表单帮你点击Submit！

我的应用场景是抓取一个"Load More..."分页后的内容，这个分页当然是ajax的了,我是这么做的：

{% codeblock lang:ruby %}
require 'nokogiri'
require 'watir-webdriver'

browser = Watir::Browser.new #实例化一个browser对象（默认是Firefox）
browser.goto your_url #your_url是你要打开的页面
browser.a(:class => 'getmore').click # load more的css是'getmore',这样就触发了加载更多的动作
sleep(5) #网络条件不好的时候等等，等页面完全渲染完成以后
doc = Nokogiri::HTML(browser.html) #将加载回来的结果传给Nokogiri去解析
content = doc.xpath("your-xpath") #获取内容！yeah！
browser.close #别忘了关闭！

{% endcodeblock %}

本地测试了一下，It works! 正高兴的时候发现了还有一个问题没有解决，上面的代码打开我本地的Firefox浏览器，但是我这个脚本是要部署的服务器上的啊，服务器上又没有图形化的浏览器让我打开。。。这时候一顿找，终于找到了！headless！

headless其实是ruby把Xvfb库封装了一下，有了这东西，watir-webdriver就可以不用调用GUI的Firefox了。
果断

{% codeblock lang:ruby %}
 gem install headless
{% endcodeblock %}

使用headless后，代码也要相应的修改一下：

{% codeblock lang:bash %}
require 'nokogiri'
require 'watir-webdriver'
require 'headless'

headless = Headless.new #实例化headless
headless.start #启动

browser = Watir::Browser.new
browser.goto your_url
browser.a(:class => 'getmore').click
doc = Nokogiri::HTML(browser.html)
content = doc.xpath("your-xpath")
browser.close

headless.destroy #销毁headless对象
{% endcodeblock %}

这样的执行这段代码，就不会打开浏览器了。大功告成！
