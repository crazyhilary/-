接下来我们分析文件中的辅助信息

```shell
执行：javac -g:lines
```

二进制文件

```
cafe babe 0000 003c 000b 0a00 0200 0307
0004 0c00 0500 0601 0010 6a61 7661 2f6c
616e 672f 4f62 6a65 6374 0100 063c 696e
6974 3e01 0003 2829 5607 0008 0100 1b63
6f6d 2f68 696c 6172 792f 636c 6173 7362
7974 652f 5369 6d70 6c65 0100 0443 6f64
6501 000f 4c69 6e65 4e75 6d62 6572 5461
626c 6500 2100 0700 0200 0000 0000 0100
0100 0500 0600 0100 0900 0000 1d00 0100
0100 0000 052a b700 01b1 0000 0001 000a
0000 0006 0001 0000 0003 0000 
```

反编译文件

```
Classfile /Users/ext.chenkai9/dev/studyNotes/java_step_foot/java/java字节码/testClass/src/com/hilary/classbyte/Simple.class
  Last modified 2022年3月5日; size 172 bytes
  SHA-256 checksum 3cbf35e7fbfc82dbf4aa890afd2a3f0b3fde6d2582fa49f4182b30237346f913
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
  #10 = Utf8               LineNumberTable
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
```

解析

```
cafe babe //魔数
0000 			//次版本号
003c 			//主版本号 60
000b 			//常量池11，比没有附加信息的常量池多了一个常量
##########常量池##################
#1	0a				//	tag = CONSTANT_Methodref_info
		0002			//	index = #2
		0003			//	index = #3
#2	07				//	tag = CONSTANT_Class_info
		0004			//	index = #4
#3	0c				//	tag = CONSTANT_NameAndType_info
		0005			//	index = #5
		0006			//	index = #6
#4	01 				//	tag = CONSTANT_Utf8_info
		0010			// length = 16 
		6a 61 76 61 2f 6c 61 6e 67 2f 4f 62 6a 65 63 74		java/lang/Object
#5	01				// tag = CONSTANT_Utf8_info
		0006			// length = 6
		3c 69 6e 69 74 3e			<init>
#6	01				// tag = CONSTANT_Utf8_info
		0003 			length = 3
		28 29 56 	// ()V
#7 	07 
		0008			//length = 8 		
#8	01				// tag = CONSTANT_Utf8_info
		001b			length = 27
		63 6f 6d 2f 68 69 6c 61 72 79 2f 63 6c 61 73 73 62
		79 74 65 2f 53 69 6d 70 6c 65		// com/hilary/classbyte/Simple
#9 	01				// tag = CONSTANT_Utf8_info
		0004			length = 4
		43 6f 64 65			// Code		
#10	01 				// tag = CONSTANT_Utf8_info
		000f 			length = 15
		4c 69 6e 65 4e 75 6d 62 65 72 54 61 62 6c 65 		// LineNumberTable
##########access_flags############		
0021			// 0x0020 | 0x0001 = 0x0021	(ACC_PUBLIC | ACC_SUPER)
##########this_class##############
0007			// com/hilary/classbyte/Simple
##########supper_class############
0002			// java/lang/Object
##########interfaces_count########
0000
##########fields_count############
0000
##########methods_count###########
0001			//一个构造方法
##########method_info#############
0001			// access_flags = ACC_PUBLIC
0005			//name_index = #5	= <init>
0006			//desciptor_index = #6 = ()V
0001			//attributes_count = 1，方法有一个属性
####attributes_info###
0009			//attribute_name_index = 9 = Code
0000 001d	// attribute_length = 29
###Code属性#
0001			//max_stack = 1
0001			//max_locals = 1
0000 0005	//code_length = 5
// code：
  2a		aload_0
  b7 		invokespecial
  0001 	#1   //Method java/lang/Object."<init>":()V
  b1		return

0000 		// exception_table_length = 0
0001 		// attributes_count = 1
#####LineNumberTable
LineNumberTable属性用于描述Java源码行号与字节码行号之间的对应关系。它并不是运行时必需的属性，默认会生成到Class文件中，可以在Javac中使用-g:none或-g:lines选项来取消或显示，对程序产生的影响主要是抛出异常时，堆栈中将不会显示出错的行号，并且在调试程序的时候，也无法按照源码行来设置断点
  000a		// attribute_name_index = 10 = LineNumberTable
  0000 0006 // attribute_length = 6
  0001 		// line_number_table_length = 1
  ##line_number_info
  0000 		// start_pc = 0
  0003 		// line_number = 3
  ##Local_variable_info
  0000 		// attribute_name_inidex = 0
```

Javac -g:lines与javac -g:lines之间的区别只有两个地方有区别：

1. 其中常量池中第10项是多出来的
2. LineNumberTable属性为新增的属性表
