---
layout: post
title: "深入理解ActiveSupport::Concern"
date: 2014-04-11 15:51:10 +0800
comments: true
categories: [ruby, rails]
---

当model变得越来越胖的时候，很自然要给model减肥不是？要不然打开model一眼望去看不到头的方法会恼火死人！
不受人待见的胖model长得像下面这个样子：
{% codeblock lang:ruby %}
 class Dog
 #...此处略去平铺的200++个方法
 end
{% endcodeblock %}

很多NB大神祭出了自己给model减肥的方法，各有各的写法，大同小异，大概长得像下面这个样子：
{% codeblock lang:ruby %}
 module Courtship
 	def self.included(base)
 		base.extend ClassMethods
 	end

 	module ClassMethods
 		def	feel_upset
 			puts ""
 		end
 	end

 	
	def	other_side
		puts "where is my other side?"
	end

	def	say_love_you
	 	puts "i love you"
	end
 end

 class Dog
 	include Courtship
 end
{% endcodeblock %}

像上面这样就能把狗求爱的这个行为分离到一个模块中，这个模块中应该包含和求爱这个行为相关的方法。按照行为分离
出来的模块才有被reuse的可能性。比如猫猫也不是要求爱嘛。。。

传说中的核心成员们碰头一商量发现大家都这么干，千篇一律的，干脆搞到ActiveSupport里面去吧~，帮大家省两句代码吧。
于是乎代码就变成下面这个样子了：
{% codeblock lang:ruby %}
 require 'active_support/concern'
 
 module Courtship
 	def self.included(base)
 		base.extend ClassMethods
 	end

 	module ClassMethods
 		def	feel_upset
 			puts ""
 		end
 	end

 	
	def	other_side
		puts "where is my other side?"
	end

	def	say_love_you
	 	puts "i love you"
	end
 end
 
 class Dog
 	include Courtship
 end
{% endcodeblock %}

如果你真的相信CoreTeam为了体恤民情帮大家省两句代码，那就跟我一样太XX了！！~其实ActiveSupport::Concern主要帮我们解决了模块之间的依赖问题。




