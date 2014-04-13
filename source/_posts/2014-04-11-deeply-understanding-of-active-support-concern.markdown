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
    def feel_upset
      puts ""
    end
  end

  def find_other_side
    puts "where is my other side?"
  end

  def say_love_you
    puts "i love you"
  end
end

class Dog
  include Courtship
end
{% endcodeblock %}

像上面这样就能把狗狗求爱的这个行为分离到一个模块中，这个模块中应该包含和求爱这个行为相关的方法。按照行为分离出来的模块才有被reuse的可能性。比如猫猫也不是要求爱嘛。。。

传说中的核心成员们碰头一商量发现大家都这么干，千篇一律的，干脆搞到ActiveSupport里面去吧~，帮大家省两句代码吧。于是乎代码就变成下面这个样子了：
{% codeblock lang:ruby %}
require 'active_support/concern'
 
module Courtship
  extend ActiveSupport::Concern
    module ClassMethods
	   #...
    end
  #...
end
 
class Dog
  include Courtship
end
{% endcodeblock %}

如果你真的相信CoreTeam为了体恤民情帮大家省两句代码，那就跟我一样太XX了！！~其实ActiveSupport::Concern主要帮我们解决了模块之间的依赖问题。模块之间的依赖其实就是假如模块B中调用了模块A中得方法那么我们就说模块B依赖模块A，回到狗狗的例子，比如狗狗求爱的模块依赖一个名叫洗澡的模块，求爱之前总是要整理整理干净嘛。。。像下面这个样子
{% codeblock lang:ruby %}
module Bath
  def self.included(base)
    base.class_eval do
      def self.take_a_hot_bath
        puts "wow~"
      end
    end
  end
end

module CourtShip
  def self.included(base)
    base.extend ClassMethods
    base.take_a_hot_bath #洗个热水澡，这里调用了Bath模块中的方法
  end
  module ClassMethods
   #...
  end
  #...
end

class Dog
  include Bath   #引入Bath模块，这个是被逼的，因为Courtship中使用了Bath的方法
  include Courtship
end
{% endcodeblock %}

上面这个例子里面Dog类本不应该去关心CourtShip和Bath之间的依赖关系对吧？要是引用N多模块难道我们还要搞清楚他们之间复杂的依赖关系吗？所以很自然想到为什么不在CourtShip里面把Bath模块引入进来，然后在Dog类中只引入CourtShip模块呢？把依赖交给模块自身去处理这点思路肯定是对的，好我们这么做试试看
{% codeblock lang:ruby %}
module Bath
#...
end

module CourtShip
 include Bath
  
  def self.included(base)
    base.take_a_hot_bath	  
  end
end

class Dog
 include CourtShip #undefined method `take_a_hot_bath' for Dog:Class 
end
{% endcodeblock %}

想法对了但上面这段代码是不能工作的。Why？原因是当Bath模块被include的时候base是CourtShip而不是Dog,这样的话，self.take_a_hot_bath方法就变成了CourtShip的singleton_method,Dog类无论是include还是extend都访问不到这个方法，所以就会报找不到方法"take_a_hot_bath"这个错误

使用ActiveSupport::Corncern就帮助我们解决了这个问题
{% codeblock lang:ruby %}
require 'active_support/concern'
module Bath
  extend ActiveSupport::Concern
  included do 
    def self.take_a_hot_bath
      puts "wow~"
    end
  end
end

module CourtShip
  extend ActiveSupport::Concern
  include Bath
  included do
    self.take_a_hot_bath
  end
end

class Dog
  include CourtShip
end
{% endcodeblock %}
