## class文件创建及分析

1. 创建项目结构：

   ```shell
   #项目名称testclass
   mkdir testClass
   #进入相应目录
   cd testClass
   #源码目录
   mkdir src
   #输出目录
   mkdir out
   #进入源码目录
   cd src
   #创建包
   mkdir -p com/hilary/classbyte
   #进入到classbyte目录
   cd com/hilary/classbyte
   #创建文件
   ```

   ![image-20220222142309153](/Users/ext.chenkai9/Library/Application Support/typora-user-images/image-20220222142309153.png)

​		

```
└── src
    └── com
        └── hilary
            └── classbyte
                ├── Simple.class
                ├── Simple.java
                ├── SimpleField.java
                └── SimpleMethod.java
```









1. 创建java文件

   1. 最简单的一个Java类，先分析最基本的结构

      Simple.java 

      ```java
      package com.hilary.classbyte;
      
      public class Simple {
      }
      ```

   2. 只有变量的类

      ```java
      package com.hilary.classbyte;
      
      public class SimpleField {
          private int intA = 2;
          private int intB;
          protected String strC = null;
          public String strD = "aa";
          public volatile long  mLong = 1025L;
          public Simple mSimple;
      
          public static boolean mBoolean;
          public final static boolean mBooleanFinal = true;
      
      }
      
      ```

   3. 只有方法的类

      ```java
      package com.hilary.classbyte;
      
      public class SimpleMethod {
      
          public void test() { }
      
          public void test(String a){}
      
          public void test2(int b, String c){}
      
          public String getName() {
              return "hilary";
          }
      
          public static void sayName() {
              System.out.println("###Name is hilary###");
          }
      
          public int getAge() {
              int baseAge = 18;
              int offsetAge = 2;
              return baseAge + offsetAge;
          }
      
      }
      
      ```

   4. 编译代码

      ```shell
      #进入项目根目录
      javac -d ./out src/com/hilary/classbyte/*.java
      
      具体参数可参考javac -help 查看
      
      用法: javac <options> <source files>
      其中, 可能的选项包括:
        @<filename>                  从文件读取选项和文件名
        -Akey[=value]                传递给注释处理程序的选项
        --add-modules <模块>(,<模块>)*
              除了初始模块之外要解析的根模块; 如果 <module>
                      为 ALL-MODULE-PATH, 则为模块路径中的所有模块。
        --boot-class-path <path>, -bootclasspath <path>
              覆盖引导类文件的位置
        --class-path <path>, -classpath <path>, -cp <path>
              指定查找用户类文件和注释处理程序的位置
        -d <directory>               指定放置生成的类文件的位置
        -deprecation                 输出使用已过时的 API 的源位置
        --enable-preview             启用预览语言功能。要与 -source 或 --release 一起使用。
        -encoding <encoding>         指定源文件使用的字符编码
        -endorseddirs <dirs>         覆盖签名的标准路径的位置
        -extdirs <dirs>              覆盖所安装扩展的位置
        -g                           生成所有调试信息
        -g:{lines,vars,source}       只生成某些调试信息
        -g:none                      不生成任何调试信息
        -h <directory>               指定放置生成的本机标头文件的位置
        --help, -help, -?            输出此帮助消息
        --help-extra, -X             输出额外选项的帮助
        -implicit:{none,class}       指定是否为隐式引用文件生成类文件
        -J<flag>                     直接将 <标记> 传递给运行时系统
        --limit-modules <模块>(,<模块>)*
              限制可观察模块的领域
        --module <模块>(,<模块>)*, -m <模块>(,<模块>)*
              只编译指定的模块，请检查时间戳
        --module-path <path>, -p <path>
              指定查找应用程序模块的位置
        --module-source-path <module-source-path>
              指定查找多个模块的输入源文件的位置
        --module-version <版本>        指定正在编译的模块版本
        -nowarn                      不生成任何警告
        -parameters                  生成元数据以用于方法参数的反射
        -proc:{none,only}            控制是否执行注释处理和/或编译。
        -processor <class1>[,<class2>,<class3>...]
              要运行的注释处理程序的名称; 绕过默认的搜索进程
        --processor-module-path <path>
              指定查找注释处理程序的模块路径
        --processor-path <path>, -processorpath <path>
              指定查找注释处理程序的位置
        -profile <profile>           请确保使用的 API 在指定的配置文件中可用
        --release <release>
              为指定的 Java SE 发行版编译。支持的发行版：7, 8, 9, 10, 11, 12, 13, 14, 15, 16
        -s <directory>               指定放置生成的源文件的位置
        --source <release>, -source <release>
              提供与指定的 Java SE 发行版的源兼容性。支持的发行版：7, 8, 9, 10, 11, 12, 13, 14, 15, 16
        --source-path <path>, -sourcepath <path>
              指定查找输入源文件的位置
        --system <jdk>|none          覆盖系统模块位置
        --target <release>, -target <release>
              生成适合指定的 Java SE 发行版的类文件。支持的发行版：7, 8, 9, 10, 11, 12, 13, 14, 15, 16
        --upgrade-module-path <path>
              覆盖可升级模块位置
        -verbose                     输出有关编译器正在执行的操作的消息
        --version, -version          版本信息
        -Werror                      出现警告时终止编译
      ```

   

   ## 分析Class文件

   ​		Class文件是一组以8位为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在文件之中，中间没有添加任何分隔符。当遇到需要占用8字节以上空间的数据项时，则会按照高位在前的方式分割成若干个8字节进行储蓄。

   ​		Class文件以伪数据结构存储数据，这种数据结构只有两种数据类型：【无符号数】和【表】。后面解析都要以这两种数据类型为基础；

   		* 无符号数属于基本的数据类型，以u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节和8个字节的无符号数，无符号数可以描述数字、索引引用、数量值或UTF-8字符串
   		* 表是占多个无符号数或者其他才作为数据项构成的复合数据类型。

   先以最简单的格式分析Class内容，内容顺序：见【Class文件格式表一】

   ```shell
   javac -g:none -d ./out src/com/hilary/classbyte/*.java	//none不输出任何辅助信息
   ```

   得出结果如下：

   Simple.class

   ```
   cafe babe 0000 003c 000a 0a00 0200 0307
   0004 0c00 0500 0601 0010 6a61 7661 2f6c
   616e 672f 4f62 6a65 6374 0100 063c 696e
   6974 3e01 0003 2829 5607 0008 0100 1b63
   6f6d 2f68 696c 6172 792f 636c 6173 7362
   7974 652f 5369 6d70 6c65 0100 0443 6f64
   6500 2100 0700 0200 0000 0000 0100 0100
   0500 0600 0100 0900 0000 1100 0100 0100
   0000 052a b700 01b1 0000 0000 0000 
   ```

   可以参考反编译结果，翻译以上内容，

   ```shell
   javap -v out/com/hilary/classbyte/Simple.class
   ```

   ```
   Classfile /Users/ext.chenkai9/dev/studyNotes/java_step_foot/java/java字节码/testClass/out/com/hilary/classbyte/Simple.class
     Last modified 2022年3月4日; size 142 bytes
     SHA-256 checksum 840eb09c8b6692e9111718bbc0782ce33689cf5d5e4f7751b81b490f17de54d1
   public class com.hilary.classbyte.Simple
     minor version: 0
     major version: 60
     flags: (0x0021) ACC_PUBLIC, ACC_SUPER
     this_class: #7                          // com/hilary/classbyte/Simple
     super_class: #2                         // java/lang/Object
     interfaces: 0, fields: 0, methods: 1, attributes: 0
   Constant pool:
      #1 = Methodref          #2.#3          // java/lang/Object."<init>":()V
      #2 = Class              #4             // java/lang/Object
      #3 = NameAndType        #5:#6          // "<init>":()V
      #4 = Utf8               java/lang/Object
      #5 = Utf8               <init>
      #6 = Utf8               ()V
      #7 = Class              #8             // com/hilary/classbyte/Simple
      #8 = Utf8               com/hilary/classbyte/Simple
      #9 = Utf8               Code
   {
     public com.hilary.classbyte.Simple();
       descriptor: ()V
       flags: (0x0001) ACC_PUBLIC
       Code:
         stack=1, locals=1, args_size=1
            0: aload_0
            1: invokespecial #1                  // Method java/lang/Object."<init>":()V
            4: return
   }
   
   ```

   现在开始

   

   cafe babe

   ```
   magic	
   每个Class文件的头4个字节被称为魔数（Magic Number），它的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件。GIF或者JPEG等文件在文件头中都存有魔数。使用魔数而不是扩展名来进行识别主要是基于安全考虑，因为文件扩展名可以随意改动
   ```

   0000

   ```
   minor_version		次版本号
   第5和第6个字节是次版本号，JDK 12之前次版本号一直没用，全部为零，JDK 12及以后版本为了标识复杂的新功能特性，需要以“公测”	的形式放出，把次版本号标识为65535，以便Java虚拟机在加载类文件时能够区分出来。
   ```

   003c

   ```
   major_version		主版本号：16
   第7和第8字节为主版本号：60，版本号是根据编辑jdk版本生成的，见版本号列表二
   ```

   000a

   常量池

   constant_pool_count = 9
   常量池有17种数据类型，见结构总表三
   常量池第一项为u2类型数据，代表常量池容量计数值，第0项常量为空，如果后面某些指向常量池的索引值的数据在特殊情况下表达“不引用任何一个常量池项目”的含义时，可以把索引值设置为0来表示，所以常量数为常量池数减1。

   ##############常量池表######################################

   > ```
   > tag含义见：常量池中的17种数据类型的结构表
   > 
   > #1	0a							tag				=		10		//CONSTANT_Methodref_info
   > 		00 02						index			=		#2		//
   > 		00 03						index			=		#3		//		
   > #2	07							tag				=		7			//CONSTANT_Class_info
   > 		0004						index			=		#4
   > #3	0c							tag				=		12		//CONSTANT_NameAndType_info
   > 		00 05						index			=		#5		
   > 		00 06						index			=		#6
   > #4	01							tag				=		1			//CONSTANT_Utf8_info
   > 		00 10						length		=		16
   > 		java/lang/Object
   > 		6a 61 76 61 2f 6c 61 6e 67 2f 4f 62 6a 65 63 74
   > #5	01							tag				=		1			//CONSTANT_Utf8_info
   > 		00 06						length		=		6	
   > 		<init>
   > 		3c 69 6e 69 74 3e
   > #6	01							tag				=		1			//CONSTANT_Utf8_info
   > 		0003						length    = 	3
   > 		()V
   > 		28 29 56
   > #7	07							tag				=		7			//CONSTANT_Class_info
   > 		0008						index			=		8		
   > #8	01							tag				=		1			//CONSTANT_Utf8_info
   > 		00 1b						length    = 	27
   > 		com/hilary/classbyte/Simple
   > 		63 6f 6d 2f 68 69 6c 61 72 79 2f 63 6c 61 73 73 
   > 		62 79 74 65 2f 53 69 6d 70 6c 65
   > #9	01							tag				=		1		//CONSTANT_Utf8_info
   > 		00 04						length		=		4		
   > 		Code
   > 		43 6f 64 65
   > ```

   ################access_flags###见访问标志表#######################	

   0021	

   ```
   0x0020 | 0x0001 = 0x0021	(ACC_PUBLIC | ACC_SUPER)
   ```

   ##################this_class####当前类全限定名##############################

   0007			

   ```
   常量池#7 -> #8 -> com/hilary/classbyte/Simple
   ```

   ##################supper_class####父类全限定名#############################

   00 02			

   ```
   常量池#2 -> #8 -> java/lang/Object
   ```

   #####################interfaces_count#####################################
   0000

   ###################interfaces（接口表）###################################

   ```
   没有继承或实现接口时，此项不会在二进制文件中体现
   ```

   ####################fields_count#########################################
   0000

   ######################fields（字段表）####################################

   ```
   此类没有中没有字段时，此项不会在二进制文件中体现
   ```

   ####################methods_count#######################################
   0001

   ```
   有一个方法，每个类至少会有一个默认的构造函数
   ```

   #####################methods（方法表）####################################

   > 方法 1
   >
   > 00 01							access_flags:									//ACC_PUBLIC
   > 00 05							name_index = #5							//<init>
   > 00 06							desciptor_index = # 6					//()V
   > 00 01							attributes_count = #1					//方法有一个属性：Code
   >
   > > attribute_info 		Code属性表
   > >
   > > 0009						attribute_name_index = #9      	//Code
   > >
   > > 00000011				attribute_length = 17					//长度为17
   > >
   > > 0001						max_stack = 1								//最大栈深度为1
   > >
   > > 0001						max_locals=1								//局部变量所需要最大存储空间
   > >
   > > 0000 0005				code_length=5							//指令长度为5个字节
   > >
   > > > code指令
   > > >
   > > > 2a		aload_0
   > > > b7 		invokespecial
   > > > 0001 	#1   //Method java/lang/Object."<init>":()V
   > > > b1		return
   > >
   > > 0000 						exception_table_length = 0		//异常表长度为0，则异常表不显示：exception_table							
   > >
   > > 0000						attributes_count = 0
   > >
   > > 0000						attributes = 0								//属性表空指向常量池0

   

   
