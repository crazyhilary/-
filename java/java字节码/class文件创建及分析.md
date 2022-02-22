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

   Simple.class

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
   0007 0002 0000 0000 0001 0001 0005 0006
   0001 0009 0000 001d 0001 0001 0000 0005
   2ab7 0001 b100 0000 0100 0a00 0000 0600
   0100 0000 0300 0100 0b00 0000 0200 0c
   ```

   SimpleField.class

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

   SimpleMethod.class

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

   

