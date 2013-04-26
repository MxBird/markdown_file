# 浅谈Ruby on Rails中的include和extend

	本文将介绍浅谈Ruby on Rails中的include和extend。
	include主要用来将一个模块插入（mix）到一个类或者其它模块。
	extend 用来在一个对象（object，或者说是instance）中引入一个模块，这个类从而也具备了这个模块的方法。

从模块引入方法、变量，使得编程变得简单，扩展性愈强，比以往的类的继承更灵活。这样的引入，仿佛将一个方法块，复制了一份放到了你所引用的类或者模块里面。你完全可以将多个互不相干的类中相同的方法拿出来写到一个模块中，这样可以使得代码精简，符合Ruby的设计初衷，而且，使得你的程序更有条理。

## Ruby on Rails常见用法
通常引用模块有以下3种情况：

1.在类定义中引入模块，使模块中的方法成为类的实例方法

这种情况是最常见的

直接 include 即可

2.在类定义中引入模块，使模块中的方法成为类的类方法

这种情况也是比较常见的

直接 extend 即可

3.在类定义中引入模块，既希望引入实例方法，也希望引入类方法

这个时候需要使用 include,

但是在模块中对类方法的定义有不同，定义出现在self.included 块中def self.included(c) ... end 中

## Ruby on Rails实例讲解

	#基类
	module Base
	  #显示
	  def show  
	    puts "You came here!"  
	  end  
	end  
	class Car  
	  extend Base   #扩展了类方法，我们可以通过Car.show调用  
	end  
	class Bus  
	  include Base  #扩展了实例方法，可以通过Bus.new.show调用  
	end

但是我们经常有这样的需要，希望基类足够强大，既可以扩展实例方法，也可以扩展类方法，Ruby on Rails同样提供了解决方案。

	Code  
	#基类  
	module Base  
	  def show  
	    puts "You came here!"  
	  end  
	 
	  #扩展类方法  
	  def self.included(base)  
	    def base.call  
	      puts "I'm strong!"  
	    end  
	    base.extend(ClassMethods)  
	  end  
	 
	  #类方法  
	  module ClassMethods  
	    def hello  
	      puts "Hello baby!"  
	    end  
	  end  
	     
	end  
	 
	class Bus  
	  include Base  
	end

此时Bus已经具备了实例方法show,类方法：call 、hello,访问方式

	Bus.new.show  
	Bus.call  
	Bus.hello  

肯定也有人提出此类疑问，使用extend能够实现此功能不？

答案是：暂未找到，如您找到请明示，多谢！

我也曾经做过以下实验，结果没有成功，在此也张贴出来，希望能给您带来一些启示。

	Code  
	#基类  
	module Base  
	  def show  
	    puts "You came here!"  
	  end  
	    
	    #扩展实例方法  
	  def self.extended(base)  
	    base.extend(InstanceMethods)  
	  end  
	 
	  module InstanceMethods  
	    def love  
	      puts 'We are instances,loving each other!'  
	    end  
	  end  
	 
	end  
	 
	class Car  
	  extend Base  
	end 

但是这样，实例方法扩展失败，依然扩展了类方法
	
	Car.show  
	Car.love             #类方法  
	Car.new.love      #undefined method 'love' 
