## 背景

一般来说，要定义一个模块，在 Ruby 中为了更好的组织类方法和实例方法，参照以下这种写法

```ruby
module M  
  def self.included(base)  
    base.extend ClassMethod  
    base.send :include, InstanceMethod  
    base.class_eval do  
      # 调用base类的方法，一般用来声明式地修改类  
      my_attr_reader :name, :age  
    end  
  end  
  module ClassMethods  
    def class_method_1; end  
  end  
  module InstanceMethods  
    def instance_method_1; end  
  end  
end  
```
上面例子中 base 就是混入模块的类，class_eval 那一段代码中可以调用 base 类的类方法my_attr_reader。

但如果现在我有两个 module，M1 和 M2，M1 依赖于 M2，而且它们都需要在被混入类C时执行 my_attr_reader 方法，似乎可以写成下面这样

```ruby
module M2  
  def self.included(base)  
    base.class_eval do  
      my_attr_reader :age  
    end  
  end  
end  
module M1  
  def self.included(base)  
    base.class_eval do  
      my_attr_reader :name  
    end  
  end  
  include M2  
end  
class C  
  def self.my_attr_reader(*args); end  
  include M1  
end  
```
**ActiveSupport::Concern**就是用来解决这类问题的，以上写法可以改成

```ruby
require 'active_support'  
module M2  
  extend ActiveSupport::Concern  
  included do  
    my_attr_reader :age  
  end  
end  
module M1  
  extend ActiveSupport::Concern  
  include M2  
  included do  
    my_attr_reader :name  
  end  
end  
class C  
  def self.my_attr_reader(*args); end  
  include M1  
end  
```
## Class Method

```ruby
module Jed
  def self.shout!
    puts "Look out!"
  end
  def hehe!
    puts "I say!"
  end
end
class Huo
  include Jed
end
```
正确例子
```ruby
module Jed
  def self.included(base)
    base.extend(ClassMethods)
  end
  def self.shout!
    puts "Look out!"
  end
  def hehe!
    puts "I say!"
  end
end
```
## 源码分析

```ruby
module ActiveSupport
  module Concern
    class MultipleIncludedBlocks < StandardError #:nodoc:
      def initialize
        super "Cannot define multiple 'included' blocks for a Concern"
      end
    end
    def self.extended(base) #:nodoc:
      base.instance_variable_set(:@_dependencies, [])
    end
    def append_features(base)
      if base.instance_variable_defined?(:@_dependencies)
        base.instance_variable_get(:@_dependencies) << self
        return false
      else
        return false if base < self
        @_dependencies.each { |dep| base.send(:include, dep) }
        super
        base.extend const_get(:ClassMethods) if const_defined?(:ClassMethods)
        base.class_eval(&@_included_block) if instance_variable_defined?(:@_included_block)
      end
    end
    def included(base = nil, &block)
      if base.nil?
        raise MultipleIncludedBlocks if instance_variable_defined?(:@_included_block)
        @_included_block = block
      else
        super
      end
    end
    def class_methods(&class_methods_module_definition)
      mod = const_defined?(:ClassMethods) ?
        const_get(:ClassMethods) :
        const_set(:ClassMethods, Module.new)
      mod.module_eval(&class_methods_module_definition)
    end
  end
end
```
#### **self.extended**

它是一个回调方法，会在**ActiveSupport::Concern**被 extend进其他 module 或 class 时触发。参数 base 就是被 extend 的 module 或 class。   这个方法的作用是，为 base 类加入一个实例变量 @_dependencies。默认值是空数组。它用来保存这个模块依赖的其他模块的列表。

#### **append_features**

这也是一个回调方法。它是在一个 module 被 include 进入其他 module 或 class 时调用的。当一个 class（或module） include 另一个 module 时，class 会按照 include module 相反的顺序去调用每个 module 的 append_features 方法。这个方法的默认实现，是把 module 的变量，实例方法等东西 copy 到被混入的 class 中。如果我们重写一个 module 的 append_features，又什么都不做的话，那么这个 module 就像没有被混入一样。下面是一个例子：



```ruby
module M3  
  def self.append_features(base)  
    puts "call M3 append_features"  
    super  
  end  
  def m3_instance_method
    puts 'm3'
  end  
end  
module M4  
  def self.append_features(base)  
    puts "call M4 append_features"  
  end  
  def m4_instance_method
    puts 'm4'
  end  
end  
class C  
  include M3, M4  
end  
# 这时会打印  
# call M4 append_features  
# call M3 append_features  
c = C.new  
c.m3_instance_method  # 这个方法成功的混入到C中  
c.m4_instance_method  # 对象c没有这个方法
```
**ActiveSupport::Concern**中的**append_feature**并不是在 Concern 模块被 include 进其他模块时调用的（这个模块只会被extend，而且这个 append_features 并不是类方法），而是对于 extend 了**ActiveSupport::Concern**的模块而言（如 module M1 ），当它被混入类C时，会触发**append_features**方法。
这个方法的作用是，如果 base 类（或 module ）有 @_dependencies 列表时，将自己记入base 的 @_dependencies 中，然后直接 return（就是不混入 base ）。如果 base 类没有 @_dependencies 列表（这种情况可以肯定 base 就是最终要混入的 class），就循环自己的 @_dependencies 列表，依次把每个依赖的 module 混入 base。

#### **included**

很简单，如果有 block。就把 block 存进 @_included_block 变量。然后在**append_featuers**中传给 base.class_eval。没有block。就和普通的**included**回调方法一样。

## 总结

基本上，**ActiveSupport::Concern**的思路就是使用**append_features**回调，去修改 module 的被 include 时的默认行为，延迟 module 被实际 include 的时机。这个模块中使用了相当一部分元编程方法。

有些细节因为比较直观所以没讲，

* 用 instance_variable_defined? 判断 @_dependencies 变量定义了没有，
* 用 instance_variable_get 和 instance_variable_set 来获取和设置实例变量
* 第 18 行的 if base < self 。这个判断是说，当base是self的子类时，返回true，否则返回nil。我没有想到什么情况会有base是self的子类的，除非自定义一个类继承自module…… 
