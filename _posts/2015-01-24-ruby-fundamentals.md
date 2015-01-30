---
layout: post
title: "Ruby Fundamentals"
description: "Ruby Fundamentals"
category: Ruby 
tags: [Ruby]
---

{% include JB/setup %}

#ruby的交互
通过IRB 或者安装 pry gem启动，ctrl-D退出，ctrl-C取消当前操作

#数据类型
##Numbers
四则运算 ＋ － ＊ ／ ％
浮点运算 请加上 .0

##String
+ 连接字符串  ＊ 3  把字符串连接3次
“” 内嵌入#{} ruby 表达式和特殊转义字符
length 长度  empty？是否为空

##Symbol
以：开头的标识符，同样Symbol的内存指向相同空间，内存更有效

##Array
[] 从0开头
＋ ［ 4，5，6］ 将返回一个连接两个数组的新数组
<<  4  将返回一个截取前4个的新数组

##Hash
{ symbol: dd}
键值对  keys 方法返回键数组， values 返回值数组

##Boolean
true false

#实数
必须是大写字母开头

#变量
小写，以下划线相连

#控制流
##条件

		if 
		end
		
		if 
		elseif 
		else
		end
		
##叠代

		for

对数组和hash对象，还有each方法，通过闭包叠代
	
		list.each do |number|
			puts number
		end
		
		hash.each do |key,value|
			puts "the #{key} is #{value}"
		end
		
#方法
ruby的方法总是有返回值，实在没有其他值就是nil

#class
##有一个特别的方法 initialize，用于初始化
##实例变量以@开头
不能在类外存取，必须有getter 和 setter方法，

		attr_accessor :symbol
		attr_reader :symbol  只读
		
##类方法 
##类继承
< 

#module
模块，在模块中定义方法 用于多重继承的

		include 模块名 把模块的方法作为该类的实例方法
		prepend 模块名 把模块的方法覆盖该类已有的实例方法
		extend 模块名 把模块的方法作为该类的类方法
		
模块也可以做我命名空间::

#ruby的对象模式

##祖先
如何去找到一个被调用方法的准确位置

		Person.ancestors  返回该类的祖先数组包括module
		[Person, Distractable, Comparable, Object, Kernel, BasicObject]
		BasicObject，Object是类 ，Distractable, Comparable, Kernel是module

BasicObject 是ruby所有类的父类

Object是ruby的默认根对象，其包含了Kernel module

Kernel module定义了ruby的方法 puts exit

如果没有找到方法抛出NoMethodError

##方法

		Person.methods 列出类方法数组
		Person.instance_methods  列出实例方法数组
		
##类.class
任何一个类的class就是Class

##反射
决定一个对象的类型通过class方法

知道一个对象的类方法和实例方法methods instance_methods

		is_a? 类  判断是否是某类对象
		instance_of? 类 判断某对象是否是该类的直接对象
		
##duck typing
是否接受某方法的调用，不管类名和继承
		
		respond_to? :method 如果是false抛出NoMethodError例外
		
##元编程
###define_method
动态定义了一个方法，可以在define_method被调用后调用定义的方法

		define_method "can_#{f}!" do
		                   @features[f] = true
		 end
		 
###class_eval

		class_eval "
			def #{attr}
				@#{attr}
			end
		"
		
###method_missing	
如果一个对象没有找到方法，该类会调用method_missing直到BasicObject，没有抛出NoMethodError例外
			
			def method_missing(name, *args, &block)
				word = name
			    puts "#{word}, #{word}, #{word}"
			end
			就会拦截全部其他不知名方法

在Rails中动态find_by_列名
			

