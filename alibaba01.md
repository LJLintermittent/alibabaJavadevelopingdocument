# 《阿里巴巴Java开发手册》

## 第一章 编程规约

* 【强制】抽象类命名使用Abstract或Base开头，异常类命名使用Exception结尾，测试类命名以它要测试的类的名称开头，以Test结尾。

* 【强制】POJO类中的任何布尔类型的变量，都不要加is前缀，否则部分框架解析会引起序列化的错误。说明：定义为数据类型boolean isDeleted的属性，它的方法也是isDeleted()，框架在反向解析的时候，会误以为对应的属性名称是deleted，导致获取不到属性，进而抛出异常

* 【强制】不允许任何魔法值(未经预先定义的常量)直接出现在代码中

* 【强制】避免通过一个类的对象引用访问此类的静态变量或静态方法，造成编译器解析成本无谓增加，直接用类名访问即可

* 【强制】只有相同参数类型，相同业务含义，才可以使用Java的可变参数，避免使用Object，可变参数必须放置在参数列表的末尾，建议工程师尽量不使用可变参数

* 【强制】不能使用过时的类或方法

* 【强制】所有整型包装类对象之间值的比较，全部使用equals()。说明：对于Integer var = 某个数字，如果在-128~127范围内赋值，Integer对象是在IntegerCache.cache产生，会复用已有的对象。这个区间内的Integer可以直接用==判断，但是这个区间之外的数据都是在堆上产生的，并不会复用已有的对象，推荐使用equals()来判断。

  ~~~java
          Integer a = 9999;
          Integer b = 9999;
          /**
           *
           * Integer有个缓存池，存着-128~127,如果对象的范围是在这个范围之内会直接返回缓存中的值
           * 不在这个范围则在堆上新创建一个对象，那么既然新创建了对象，非基本数据类型使用==又是比较内存地址
           * 当然就不相等了
           * ==作用于八大基本数据类型，比较的是值
           * 由于==作用于非基本数据类型，比较的是内存地址
           */
          System.out.println(a == b);//false
          System.out.println(a == b.intValue());//true
          /**
           * Integer类重写了equals方法
           * 实现是使用intvalue()来得到Integer对象的value属性
           * 比较的时候是比较两个value值的
           */
          System.out.println(a.equals(b));//true
  ~~~

  

* 【强制】浮点数之间的等值判断，基本数据类型不能用==进行判断，包装数据类型不能用equals()方法进行判断。说明：浮点数采用尾数+阶码的编码方式，类似于科学计数法中的有效数字+指数的方式。二进制无法精确表示大部分十进制的小数。BigDecimal的等值比较应该使用compareTo()，不能使用equals()

  ~~~java
     /**
           * 二进制无法精确表示大部分十进制的小数
           * 对于这种情况，我们应该指定一个误差，
           * 若两个浮点数的差值的绝对值在这个范围之内，那么就认为它们是相等的
           * 例如:
           * 10的-6次方
           * int diff = 1e-6f;
           */
          float c = 1.0f - 0.9f;
          float d = 0.9f - 0.8f;
          System.out.println(c == d);//false
          Float e = Float.valueOf(c);
          Float f = Float.valueOf(d);
          System.out.println(e.equals(f));//false
  ~~~

  * 【强制】禁止使用构造方法BigDecimal(double)的方式把double的值转化为BigDecimal对象。说明：BigDecimal(double)存在精度损失的风险，在精确计算或值比较的场景中，可能会导致业务逻辑出现异常。正确方法：使用入参为String的BigDecimal方法，或使用BigDecimal的valueOf(),此方法内部使用了Double的toString，而Double的toString按double实际能表达的精度对尾数进行了截断。
  
  ~~~java
  BigDecimal v1  = new BigDecimal("0.1");
  BigDecimal v2 = BigDecimal.valueOf(0.1);
  ~~~
  
  * 【强制】所有的POJO类属性都必须使用包装数据类型
  
  * 【强制】RPC方法的返回值和参数必须使用包装数据类型
  
  * 【推荐】所有的局部变量都是用基本数据类型
  
    ~~~wiki
    说明：POJO类属性没有初值，是提醒使用者在需要使用的时候，必须自己显示地进行赋值，任何NPE问题或者入库检查，都由使用者保证
    数据库查询的结果可能是Null，因为自动拆箱，所以用基本数据类型接收会有NPE风险
    ~~~
  
  * 【强制】在定义DO/DTO/VO等POJO类时，不要设定任何属性默认值
  
  * 【强制】序列化类新增属性时，请不要修改serialVersionUID字段，避免反序列化失败，如果完全不兼容升级，那么为避免反序列化混乱，请修改serialVersionUID值
  
    ~~~wiki
    注意；serialVersionUID不一致会抛出序列化运行时异常
    ~~~
  
  * 【强制】构造方法里面禁止加入任何业务逻辑，如果有初始化逻辑，那么请放在init方法中
  
  * 【强制】POJO类必须写toString方法，如果继承了另一个POJO类，那么注意在前面加super.toString
  
    ~~~wiki
    说明：在方法执行抛出异常时，可以直接调用POJO的toString()方法打印其属性值，便于排查问题
    ~~~
  
  * 【强制】禁止在POJO类中同时存在对应属性Xxx的isXxx()和getXxx()方法
  
    ~~~wiki
    说明：框架在调用属性xxx的提取方法时，并不能确定哪种方法一定是被优先调用的
    ~~~
  
  * 【推荐】在getter/setter方法中，不要增加业务逻辑，否则会增加排查问题的难度(mall电商项目中在计算价格时在getter方法中加了计算逻辑，下次注意)
  
  * 【推荐】
  
    