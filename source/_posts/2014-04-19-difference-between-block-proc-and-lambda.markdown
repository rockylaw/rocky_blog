---
layout: post
title: "block、proc和lambda的区别"
date: 2014-04-19 18:29:13 +0800
comments: true
categories: ruby
keywords: "ruby, block, proc, lambda, lambda和proc的区别"
description: "ruby语言中block, Proc和Lambda的区别"
---

这几个概念总是被人问起，搞来搞去的，搞晕了肿么办？谨以此文防止搞晕！

首先要搞清出proc和lambda之间的区别，得给block正名
block仅仅只是一个语法概念，而proc和lambda都是Proc类的对象,block离开了具体的上下文是不能执行的
{% codeblock lang:ruby %}
{ puts "i'm a block" } #=> 这是一个块block，单独执行会报SyntaxError
2.times { puts "i'm a block" } #=> 给block一个执行环境（2.times）后，就能够被执行
{% endcodeblock %}
由此可见block并不是一个对象，而指的是ruby中得一个语法现象，由一对花括号{}括起来或是有do..end关键字包裹起来的内容都是一个块（block），块离开了上下文执行环境是不能单独执行的。

搞清了block这个概念之后，先来看看proc和lambda
{% codeblock lang:ruby %}
proc = Proc.new{ puts "i'm a proc" }
proc.class #=> Proc

lamb = lambda{ puts "i'm a lambda" }
lamb.class #=> Proc
{% endcodeblock %}
这样我们就知道了proc对象和lambda对象实际上都是Proc类的对象。那么他们具体有什么不同呢？

##lambda会检查参数的个数，而proc不会##
{% codeblock lang:ruby %}
lamb = lambda{|x| puts x }
lamb.call("first") #=> first
lamb.call("first","second") #=> ArgumentError: wrong number of arguments (2 for 1)

proc = Proc.new{|x| puts x }
proc.call("first") #=> first
proc.call("first", "second") #=> first
proc.call("first", "second", "third") #=> first

proc = Proc.new{|x,y| puts x, y }
proc.call("first", "second", "third") #=> first second

{% endcodeblock %}
lambda对象会去检查传入参数的个数，而proc对象会默认忽略掉多余传入的参数

##lambda和proc看待return关键字是不同的##
{% codeblock lang:ruby %}
def lambda_treat_return
  lamb = lambda{ return }
  lamb.call
  puts "passed"
end
lambda_treat_return #=> passed #return之后方法的执行并没有中断，继续执行

def proc_treat_return
  proc = Proc.new{ return }
  proc.call
  puts "passed" 
end
proc_treat_return #=> 没有输出，方法的执行中断了
{% endcodeblock %}

lambda中return之后方法体会继续向下执行，而proc中得return会直接退出方法

另外可以通过'&'符号把一个proc对象转换成block
{% codeblock lang:ruby %}
  proc = Proc.new{ puts "it's me"}
  2.times &proc #=> it's me it's me
{% endcodeblock %}
