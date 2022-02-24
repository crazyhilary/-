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

2. 创建java文件

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
      
          public void test2(int b){}
      
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
      ```

   

   ## 分析Class文件

   ​		Class文件是一组以8位为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在文件之中，中间没有添加任何分隔符。当遇到需要占用8字节以上空间的数据项时，则会按照高位在前的方式分割成若干个8字节进行储蓄。

   ​		Class文件以伪数据结构存储数据，这种数据结构只有两种数据类型：【无符号数】和【表】。后面解析都要以这两种数据类型为基础；
   
   		* 无符号数属于基本的数据类型，以u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节和8个字节的无符号数，无符号数可以描述数字、索引引用、数量值或UTF-8字符串
   		* 表是帖多个无符号数或者其他才作为数据项构成的复合数据类型。
   
   
   
   #### 内容顺序：见【Class文件格式表】
   
   #### 1. Simple.class
   
   ```
   cafe babe 0000 003c 000d 0a00 0200 0307
   0004 0c00 0500 0601 0010 6a61 7661 2f6c
   616e 672f 4f62 6a65 6374 0100 063c 696e
   6974 3e01 0003 2829 5607 0008 0100 1b63
   6f6d 2f68 696c 6172 792f 636c 6173 7362
   7974 652f 5369 6d70 6c65 0100 0443 6f64 
   6501 000f 4c69 6e65 4e75 6d62 6572 5461	
   626c 6501 000a 536f 7572 6365 4669 6c65
   0100 0b53 696d 706c 652e 6a61 7661 0021
   0007 0002 0000 0000 
   0001 0001 0005 0006
   0001 0009 0000 001d 0001 0001 0000 0005
   2ab7 0001 b100 0000 0100 0a00 0000 0600
   0100 0000 0300 0100 0b00 0000 0200 0c
   ```
   
   文件解析
   
   ```
   ca fe ba be				Java魔数
   00 00						次版本号
   00 3c						主版本号：60->JDK16
   00 0d						常量数量：13
   #############常量池表 cp_info########参考：常量池中的17种数据类型的结构总表
   #1	0a							tag				=		10		//CONSTANT_Methodref_info
   		00 02						index			=		#2		//
   		00 03						index			=		#3		//		
   #2	07							tag				=		7			//CONSTANT_Class_info
   		0004						index			=		#4
   #3	0c							tag				=		12		//CONSTANT_NameAndType_info
   		00 05						index			=		#5		
   		00 06						index			=		#6
   #4	01							tag				=		1			//CONSTANT_Utf8_info
   		00 10						length		=		16
   		java/lang/Object
   		6a 61 76 61 2f 6c 61 6e 67 2f 4f 62 6a 65 63 74
   #5	01							tag				=		1			//CONSTANT_Utf8_info
   		00 06						length		=		6	
   		<init>
   		3c 69 6e 69 74 3e
   #6	01							tag				=		1			//CONSTANT_Utf8_info
   		0003						length    = 	3
   		()V
   		28 29 56
   #7	07							tag				=		7			//CONSTANT_Class_info
   		0008						index			=		8		
   #8	01							tag				=		1			//CONSTANT_Utf8_info
   		00 1b						length    = 	27
   		com/hilary/classbyte/Simple
   		63 6f 6d 2f 68 69 6c 61 72 79 2f 63 6c 61 73 73 
   		62 79 74 65 2f 53 69 6d 70 6c 65
   #9	01							tag				=		1		//CONSTANT_Utf8_info
   		00 04						length		=		4		
   		Code
   		43 6f 64 65
   #10	01							tag				=		1		//CONSTANT_Utf8_info
   		00 0f						length		=		15
   		LineNumberTable
   		4c 69 6e 65 4e 75 6d 62 65 72 54 61 62 6c 65
   #11	01							tag				=		1		//CONSTANT_Utf8_info
   		000a						length		=		10
   		SourceFile
   		53 6f 75 72 63 65 46 69 6c 65
   #12	01							tag				=		1		//CONSTANT_Utf8_info
   		00 0b						length		=		11	
   		Simple.java
   		53 69 6d 70 6c 65 2e 6a 61 76 61
   ################access_flags###见访问标志表#######################		
   0021	0x0020 | 0x0001 = 0x0021	(ACC_PUBLIC | ACC_SUPER)
   ##################this_class##################################
   00 07			常量池#7 -> #8 -> com/hilary/classbyte/Simple
   ##################supper_class##################################
   00 02			常量池#2 -> #8 -> java/lang/Object
   #####################interfaces_count#####################
   00 00
   ####################interfaces###########################
   00 00
   ####################fields_count#########################
   00 01
   ####################fields###见字段###########################
   00 01
   
   
   ```
   
   反编译
   
   ```
   Last modified 2022年2月22日; size 207 bytes
     SHA-256 checksum ab01e73c5f45994584869f834d7303534984b36a16b3846ad6cf988812fb6fdc
     Compiled from "Simple.java"
   public class com.hilary.classbyte.Simple
     minor version: 0
     major version: 60
     flags: (0x0021) ACC_PUBLIC, ACC_SUPER
     this_class: #7                          // com/hilary/classbyte/Simple
     super_class: #2                         // java/lang/Object
     interfaces: 0, fields: 0, methods: 1, attributes: 1
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
     #10 = Utf8               LineNumberTable
     #11 = Utf8               SourceFile
     #12 = Utf8               Simple.java
   {
     public com.hilary.classbyte.Simple();
       descriptor: ()V
       flags: (0x0001) ACC_PUBLIC
       Code:
         stack=1, locals=1, args_size=1
            0: aload_0
            1: invokespecial #1                  // Method java/lang/Object."<init>":()V
            4: return
         LineNumberTable:
           line 3: 0
   }
   SourceFile: "Simple.java"
   ```
   
   
   
   
   
   
   
   #### 2. SimpleField.class
   
   ```
   cafe babe 0000 003c 0028 0a00 0200 0307
   0004 0c00 0500 0601 0010 6a61 7661 2f6c
   616e 672f 4f62 6a65 6374 0100 063c 696e
   6974 3e01 0003 2829 5609 0008 0009 0700
   0a0c 000b 000c 0100 2063 6f6d 2f68 696c
   6172 792f 636c 6173 7362 7974 652f 5369
   6d70 6c65 4669 656c 6401 0004 696e 7441
   0100 0149 0900 0800 0e0c 000f 0010 0100
   0473 7472 4301 0012 4c6a 6176 612f 6c61
   6e67 2f53 7472 696e 673b 0800 1201 0002
   6161 0900 0800 140c 0015 0010 0100 0473
   7472 4405 0000 0000 0000 0401 0900 0800
   190c 001a 001b 0100 056d 4c6f 6e67 0100
   014a 0100 0469 6e74 4201 0007 6d53 696d
   706c 6501 001d 4c63 6f6d 2f68 696c 6172
   792f 636c 6173 7362 7974 652f 5369 6d70
   6c65 3b01 0008 6d42 6f6f 6c65 616e 0100
   015a 0100 0d6d 426f 6f6c 6561 6e46 696e
   616c 0100 0d43 6f6e 7374 616e 7456 616c
   7565 0300 0000 0101 0004 436f 6465 0100
   0f4c 696e 654e 756d 6265 7254 6162 6c65
   0100 0a53 6f75 7263 6546 696c 6501 0010
   5369 6d70 6c65 4669 656c 642e 6a61 7661
   0021 0008 0002 0000 0008 0002 000b 000c
   0000 0002 001c 000c 0000 0004 000f 0010
   0000 0001 0015 0010 0000 0041 001a 001b
   0000 0001 001d 001e 0000 0009 001f 0020
   0000 0019 0021 0020 0001 0022 0000 0002
   0023 0001 0001 0005 0006 0001 0024 0000
   0044 0003 0001 0000 001c 2ab7 0001 2a05
   b500 072a 01b5 000d 2a12 11b5 0013 2a14
   0016 b500 18b1 0000 0001 0025 0000 0016
   0005 0000 0003 0004 0004 0009 0006 000e
   0007 0014 0008 0001 0026 0000 0002 0027
   
   ```
   
   #### 3. SimpleMethod.class
   
   ```
   cafe babe 0000 003c 0011 0a00 0200 0307
   0004 0c00 0500 0601 0010 6a61 7661 2f6c
   616e 672f 4f62 6a65 6374 0100 063c 696e
   6974 3e01 0003 2829 5607 0008 0100 2163
   6f6d 2f68 696c 6172 792f 636c 6173 7362
   7974 652f 5369 6d70 6c65 4d65 7468 6f64
   0100 0443 6f64 6501 000f 4c69 6e65 4e75
   6d62 6572 5461 626c 6501 0004 7465 7374
   0100 1528 4c6a 6176 612f 6c61 6e67 2f53
   7472 696e 673b 2956 0100 0574 6573 7432
   0100 0428 4929 5601 000a 536f 7572 6365
   4669 6c65 0100 1153 696d 706c 654d 6574
   686f 642e 6a61 7661 0021 0007 0002 0000
   0000 0004 0001 0005 0006 0001 0009 0000
   001d 0001 0001 0000 0005 2ab7 0001 b100
   0000 0100 0a00 0000 0600 0100 0000 0300
   0100 0b00 0600 0100 0900 0000 1900 0000
   0100 0000 01b1 0000 0001 000a 0000 0006
   0001 0000 0005 0001 000b 000c 0001 0009
   0000 0019 0000 0002 0000 0001 b100 0000
   0100 0a00 0000 0600 0100 0000 0700 0100
   0d00 0e00 0100 0900 0000 1900 0000 0200
   0000 01b1 0000 0001 000a 0000 0006 0001
   0000 0009 0001 000f 0000 0002 0010 
   ```
   
   

