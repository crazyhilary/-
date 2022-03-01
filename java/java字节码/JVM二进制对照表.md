

##### Class文件格式表

| 类型           | 名称                          | 数量                  |
| -------------- | ----------------------------- | --------------------- |
| u4             | magic（魔数）                 | 1                     |
| u2             | minor_version（次版本号）     | 1                     |
| u2             | major_version（主版本号）     | 1                     |
| u2             | constant_pool_count（常量数） | 1                     |
| cp_info        | constant_pool（常量池表）     | constant_pool_count-1 |
| u2             | access_flags（访问标识）      | 1                     |
| u2             | this_class（当前类全限量名）  | 1                     |
| u2             | supper_class（父类全限量名）  | 1                     |
| u2             | interfaces_count（接口数量）  | 1                     |
| u2             | interfaces（接口表）          | interfaces_count      |
| u2             | fields_count（字段数量）      | 1                     |
| field_info     | fields                        | fields_count          |
| u2             | methods_count                 | 1                     |
| method_info    | methods                       | methods_count         |
| u2             | attributes_count              | 1                     |
| attribute_info | attributes                    | attributes_count      |



1. **常量池中的17种数据类型的结构总表**

<table>
  <tr>
  	<th>常量</th>
    <th>项目</th>
    <th>类型</th>
    <th>描述</th>
  </tr>
  <tr>
  	<td rowspan="3">CONSTANT_Utf8_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为1</td>
  </tr>
  <tr>
  	<td>length</td>
    <td>u2</td>
    <td>UTF-8编码的字符串占用了字节数</td>
  </tr>
  <tr>
  	<td>bytes</td>
    <td>u1</td>
    <td>长度为length的UTF-8编码的字符串</td>
  </tr>
  <tr>
  	<td rowspan="2">CONSTANT_Integer_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为3</td>
  </tr>
  <tr>
  	<td>bytes</td>
    <td>u4</td>
    <td>按照高位在前存储的int值</td>
  </tr>
  <tr>
  	<td rowspan="2">CONSTANT_Float_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为4</td>
  </tr>
  <tr>
  	<td>bytes</td>
    <td>u4</td>
    <td>按照高位在前存储的float值</td>
  </tr>
  <tr>
  	<td rowspan="2">CONSTANT_Long_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为5</td>
  </tr>
  <tr>
  	<td>bytes</td>
    <td>u8</td>
    <td>按照高位在前存储的long值</td>
  </tr>
  <tr>
  	<td rowspan="2">CONSTANT_Double_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为6</td>
  </tr>
  <tr>
  	<td>bytes</td>
    <td>u8</td>
    <td>按照高位在前存储的double值</td>
  </tr>
  <tr>
  	<td rowspan="2">CONSTANT_Class_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为7</td>
  </tr>
  <tr>
  	<td>index</td>
    <td>u2</td>
    <td>指向全限定名常量项的索引</td>
  </tr>
  <tr>
  	<td rowspan="2">CONSTANT_String_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为8</td>
  </tr>
  <tr>
  	<td>index</td>
    <td>u2</td>
    <td>指向字符串字面量的索引</td>
  </tr>
  <tr>
  	<td rowspan="3">CONSTANT_Fieldref_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为9</td>
  </tr>
  <tr>
  	<td>index</td>
    <td>u2</td>
    <td>指向声明字段的类或者接口描述符CONSTANT_Class_info的索引项</td>
  </tr>
    <tr>
  	<td>index</td>
    <td>u2</td>
    <td>指向字段描述符CONSTANT_NameAndType的索引项</td>
  </tr>
  <tr>
  	<td rowspan="3">CONSTANT_Methodref_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为10</td>
  </tr>
  <tr>
  	<td>index</td>
    <td>u2</td>
    <td>指向声明方法的类描述符CONSTANT_Class_info的索引项</td>
  </tr>
    <tr>
  	<td>index</td>
    <td>u2</td>
    <td>指向名称及类型描述符CONSTANT_NameAndType的索引项</td>
  </tr>
  <tr>
  	<td rowspan="3">CONSTANT_InterfaceMethodref_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为11</td>
  </tr>
  <tr>
  	<td>index</td>
    <td>u2</td>
    <td>指向声明方法的接口描述符CONSTANT_Class_info的索引项</td>
  </tr>
    <tr>
  	<td>index</td>
    <td>u2</td>
    <td>指向名称及类型描述符CONSTANT_NameAndType的索引项</td>
  </tr>
  <tr>
  	<td rowspan="3">CONSTANT_NameAndType_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为12</td>
  </tr>
  <tr>
  	<td>index</td>
    <td>u2</td>
    <td>指向该字段或方法名称常量项的索引</td>
  </tr>
    <tr>
  	<td>index</td>
    <td>u2</td>
    <td>指向该字段或方法描述符常量项的索引</td>
  </tr>
  <tr>
  	<td rowspan="3">CONSTANT_MethodHandle_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为15</td>
  </tr>
  <tr>
  	<td>reference_kind</td>
    <td>u1</td>
    <td>值必须在1至9之间（包括1和9），它决定了方法句柄的类型。方法句柄类型的值表示方法句柄的字节码行为</td>
  </tr>
    <tr>
  	<td>reference_index</td>
    <td>u2</td>
    <td>值必须是对常量池的有效索引</td>
  </tr>
  <tr>
  	<td rowspan="2">CONSTANT_MethodType_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为16</td>
  </tr>
  <tr>
  	<td>descriptor_index</td>
    <td>u2</td>
    <td>值必须是对常量池的有效索引，常量池在该索引处的项必须是CONSTANT_Utf8_info结构，表示方法的描述符</td>
  </tr>
  <tr>
  	<td rowspan="3">CONSTANT_Dynamic_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为17</td>
  </tr>
  <tr>
  	<td>bootstrap_method_att_index</td>
    <td>u2</td>
    <td>值必须是对当前Class文件中引导方法表的bootstrap_methods[]数组的有效索引</td>
  </tr>
    <tr>
  	<td>name_and_type_index</td>
    <td>u2</td>
    <td>值必须是对当前常量池的有效索引，常量池在该索引处的项必须是CONSTANT_NameAndType_info结构，表示方法名和方法描述符</td>
  </tr>
  <tr>
  	<td rowspan="3">CONSTANT_InvokeDynamic_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为18</td>
  </tr>
  <tr>
  	<td>bootstrap_method_att_index</td>
    <td>u2</td>
    <td>值必须是对当前Class文件中引导方法表的bootstrap_methods[]数组的有效索引</td>
  </tr>
    <tr>
  	<td>name_and_type_index</td>
    <td>u2</td>
    <td>值必须是对当前常量池的有效索引，常量池在该索引处的项必须是CONSTANT_NameAndType_info结构，表示方法名和方法描述符</td>
  </tr>
  <tr>
  	<td rowspan="2">CONSTANT_Module_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为19</td>
  </tr>
  <tr>
  	<td>name_index</td>
    <td>u2</td>
    <td>值必须是对当前常量池的有效索引，常量池在该索引处的项必须是CONSTANT_Utf8_info结构，表示模块名称</td>
  </tr>
    <tr>
  	<td rowspan="2">CONSTANT_Package_info</td>
    <td>tag</td>
    <td>u1</td>
    <td>值为20</td>
  </tr>
  <tr>
  	<td>name_index</td>
    <td>u2</td>
    <td>值必须是对当前常量池的有效索引，常量池在该索引处的项必须是CONSTANT_Utf8_info结构，表示包名称</td>
  </tr>



2. **访问标志表**

| 标志名称       | 标志值 | 含义                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001 | 是否为public类型                                             |
| ACC_FINAL      | 0x0010 | 是否被声明为final，只有类可设置                              |
| ACC_SUPER      | 0x0020 | 是否允许使用invokespecial字节码指令的新语义，invokespecial指令的语义在JDK1.0.2发生过改变，为了区别这条指令使用哪种语义，JDK1.0.2之后编译出来的类的这个标志都必须为真 |
| ACC_INTERFACE  | 0x0200 | 标识这是一个接口                                             |
| ACC_ABSTRACT   | 0x0400 | 是否为abstract类型，对于接口或者抽象类来说，此标志值为真，其它类型值为假 |
| ACC_SYNTHETIC  | 0x1000 | 标识这个类并非由用户代码产生的                               |
| ACC_ANNOTATION | 0x2000 | 标识这是一个注解                                             |
| ACC_ENUM       | 0x4000 | 标识这是一个枚举                                             |
| ACC_MODULE     | 0x8000 | 标识这是一个模块                                             |

3. **二进制对照表**

| ASCII值 | 控制字符 | ASCII值 | 控制字符 | ASCII值 | 控制字符 | ASCII值 | 控制字符 |
| :------ | :------- | :------ | :------- | :------ | :------- | :------ | :------- |
| 0       | NUT      | 32      | (space)  | 64      | @        | 96      | 、       |
| 1       | SOH      | 33      | !        | 65      | A        | 97      | a        |
| 2       | STX      | 34      | "        | 66      | B        | 98      | b        |
| 3       | ETX      | 35      | #        | 67      | C        | 99      | c        |
| 4       | EOT      | 36      | $        | 68      | D        | 100     | d        |
| 5       | ENQ      | 37      | %        | 69      | E        | 101     | e        |
| 6       | ACK      | 38      | &        | 70      | F        | 102     | f        |
| 7       | BEL      | 39      | ,        | 71      | G        | 103     | g        |
| 8       | BS       | 40      | (        | 72      | H        | 104     | h        |
| 9       | HT       | 41      | )        | 73      | I        | 105     | i        |
| 10      | LF       | 42      | *        | 74      | J        | 106     | j        |
| 11      | VT       | 43      | +        | 75      | K        | 107     | k        |
| 12      | FF       | 44      | ,        | 76      | L        | 108     | l        |
| 13      | CR       | 45      | -        | 77      | M        | 109     | m        |
| 14      | SO       | 46      | .        | 78      | N        | 110     | n        |
| 15      | SI       | 47      | /        | 79      | O        | 111     | o        |
| 16      | DLE      | 48      | 0        | 80      | P        | 112     | p        |
| 17      | DCI      | 49      | 1        | 81      | Q        | 113     | q        |
| 18      | DC2      | 50      | 2        | 82      | R        | 114     | r        |
| 19      | DC3      | 51      | 3        | 83      | S        | 115     | s        |
| 20      | DC4      | 52      | 4        | 84      | T        | 116     | t        |
| 21      | NAK      | 53      | 5        | 85      | U        | 117     | u        |
| 22      | SYN      | 54      | 6        | 86      | V        | 118     | v        |
| 23      | TB       | 55      | 7        | 87      | W        | 119     | w        |
| 24      | CAN      | 56      | 8        | 88      | X        | 120     | x        |
| 25      | EM       | 57      | 9        | 89      | Y        | 121     | y        |
| 26      | SUB      | 58      | :        | 90      | Z        | 122     | z        |
| 27      | ESC      | 59      | ;        | 91      | [        | 123     | {        |
| 28      | FS       | 60      | <        | 92      | /        | 124     | \|       |
| 29      | GS       | 61      | =        | 93      | ]        | 125     | }        |
| 30      | RS       | 62      | >        | 94      | ^        | 126     | `        |
| 31      | US       | 63      | ?        | 95      | _        | 127     | DEL      |



4. **特殊字符解释**

| NUL空        | VT 垂直制表   | SYN 空转同步  |
| ------------ | ------------- | ------------- |
| STX 正文开始 | CR 回车       | CAN 作废      |
| ETX 正文结束 | SO 移位输出   | EM 纸尽       |
| EOY 传输结束 | SI 移位输入   | SUB 换置      |
| ENQ 询问字符 | DLE 空格      | ESC 换码      |
| ACK 承认     | DC1 设备控制1 | FS 文字分隔符 |
| BEL 报警     | DC2 设备控制2 | GS 组分隔符   |
| BS 退一格    | DC3 设备控制3 | RS 记录分隔符 |
| HT 横向列表  | DC4 设备控制4 | US 单元分隔符 |
| LF 换行      | NAK 否定      | DEL 删除      |



**字段表结构**

| 类型           | 名称             | 数量             |
| -------------- | ---------------- | ---------------- |
| u2             | access_flags     | 1                |
| u2             | name_index       | 1                |
| u2             | descriptor_index | 1                |
| u2             | attributes_count | 1                |
| attribute_info | attributes       | attributes_count |

**方法表结构**

| 类型           | 名称             | 数量             |
| -------------- | ---------------- | ---------------- |
| u2             | access_flags     | 1                |
| u2             | name_index       | 1                |
| u2             | desciptor_index  | 1                |
| u2             | attributes_count | 1                |
| attribute_info | attributes       | attributes_count |

**方法访问标志**

| 标志名称         | 标志值 | 含义                             |
| ---------------- | ------ | -------------------------------- |
| ACC_PUBLIC       | 0x0001 | 方法是否为public                 |
| ACC_PRIVATE      | 0x0002 | 方法是否为private                |
| ACC_PROTECTED    | 0x0004 | 方法是否为protected              |
| ACC_STATIC       | 0x0008 | 方法是否为static                 |
| ACC_FINAL        | 0x0010 | 方法是否为final                  |
| ACC_SYNCHRONIZED | 0x0020 | 方法是否为synchronized           |
| ACC_BRIDGE       | 0x0040 | 方法是不是由编译器产生的桥接方法 |
| ACC_VARARGS      | 0x0080 | 方法是否接受不定参数             |
| ACC_NATIVE       | 0x0100 | 方法是否 为native                |
| ACC_ABSTRACT     | 0x0400 | 方法是否为abstract               |
| ACC_STRICT       | 0x0800 | 方法是否为strictfp               |
| ACC_SYNTHETIC    | 0x1000 | 方法是否由编译器自动产生         |



**虚拟机规范预定义的属性**

| 属性名称                             | 使用位置                     | 含义                                                         |
| ------------------------------------ | ---------------------------- | ------------------------------------------------------------ |
| Code                                 | 方法表                       | Java代码编译成的字节码指令                                   |
| ConstantValue                        | 字段表                       | 由final 关键字定义的常量值                                   |
| Deprecated                           | 类、方法表、字段表           | 被声明为deprecated的方法和字段                               |
| Exceptions                           | 方法表                       | 方法抛出的异常列表                                           |
| EnclosingMethod                      | 类文件                       | 仅当一个类为局部类或者匿名类时才能垫拥有这个属性，这个属性用于标示这个类所在的外围方法 |
| InnerClasses                         | 类文件                       | 内部类列表                                                   |
| LineNumberTable                      | Code属性                     | Java源码的行号与字节码指令的对应关系                         |
| LocalVariableTable                   | Code属性                     | 方法的局部变量描述                                           |
| StackMapTable                        | Code属性                     | JDK 6中新增的属性，供新的类型检查验证器（Type Checker）检查和处理目标方法的局部变量和操作数所需要的类型是否匹配 |
| Signature                            | 类、方法表、字段表           | JDK5 中新增的属性，用于支持泛型情况下的方法签名。在Java语言中，任何类、接口、初始化方法或成员的泛型签名如果包含了类型变量（Type Variables）或参数化类型（Parameterized Type)，则Signature属性会为它记录泛型签名信息。由于Java的泛型采用擦除法实现，为了避免类型信息被擦除后导致的签名混乱，需要这个属性记录泛型中的相关信息 |
| SourceFile                           | 类文件                       | 记录源文件名称                                               |
| SourceDebugExtensiion                | 类文件                       | JDK5 中新增的属性，用于存储额外的调试信息。譬如在进行JSP文件调试时，无法通过Java堆栈来定位到JSP文件的型号，JSR 45提案为这些非Java语言编写，却需要编译成字节码并运行在Java虚拟机中的程序提供了一个进行调试的标准机制，使用该属性就可以用于存储这个标准所新加入的调试信息 |
| Synthetic                            | 类、方法表、字段表           | 标识方法或字段为编译器自动生成的                             |
| LocalVariableTypeTable               | 类                           | JDK 5中新增的属性，它使用特征签名代替描述符，是为了引入泛型语法之后能描述泛型参数化类型面添加 |
| RuntimeVisibleAnnotations            | 类、方法表、字段表           | JDK 5中新增的属性，为动态注解提供支持。该属性用于指明哪些注解是运行时（实际上运行时就是进行反射调用）可见的 |
| RuntimeInvisibleAnnotations          | 类、方法表、字段表           | JDK 5中新增的属性，与RuntimeVisibleAnnotations属性作用刚好相反，用于指明哪些注解是运行时不可见的 |
| RuntimeVisibleParameterAnnotations   | 方法表                       | JDK 5中新增的属性，作用与RuntimeVisibleAnnotations属性类似，只不过作用对象为方法参数 |
| RuntimeInvisibleParameterAnnotations | 方法表                       | JDK 5中新增的属性，作用与RuntimeInvisibleAnnotations属性类似，只不过作用对象为方法参数 |
| AnnotationDefault                    | 方法表                       | JDK 5中新增的属性，用于记录注解类元素的默认值                |
| BootstrapMethods                     | 类文件                       | JDK 7中新增的属性，用于保存invokedynamic指令引用的引导方法限定符 |
| RuntimeVisibleTypeAnnotatinons       | 类、方法表、字段表，Code属性 | JDK 8中新增的属性，为实现JSR 308中新增的类型注解提供的支持，用于指明哪些类注解是运行时（实际上运行时就是进行反射调用）可见的 |
| RuntimeInvisibleTypeAnnotions        | 类、方法表、字段表，Code属性 | JDK 8中新增的属性，为实现JSR 308中新增的类型注解提供的支持，与RuntimeVisibleTypeAnnotations属性作用刚好相反，与RuntimeVisibleTypeAnnotations属性作用刚好相反，用于指明哪些注解是运行时不可见的 |
| MethodParameters                     | 方法表                       | JDK 8中新增的属性，用于支持（编译时加上-parameters参数）将方法名称编译进Class文件中，并可运行时获取。此前要获取方法名称（典型的如IDE的代码提示）只能通过JavaDoc中得到 |
| Module                               | 类                           | JDK 9中新增的属性，用于记录一个Module的名称以及相关信息（requires、exports、opens、uses、provides） |
| ModulePackages                       | 类                           | JDK 9中新增的属性，用于记录一个模块中所有被exports或者open的包 |
| ModuleMainClass                      | 类                           | JDK 9中新增的属性，用于指定一个模块的主类                    |
| NestHost                             | 类                           | JDK 11中新增的属性，用于支持嵌套类（Java 中的内部类）的反射和访问控制的API，一个内部类通过该属性得知自己的宿主类 |
| NestMembers                          | 类                           | JDK 11中新增的属性，用于支持嵌套类（Java 中的内部类）的反射和访问控制的API，一个宿主类通过该属性得知自己有哪些内部类 |

 **Code属性表的结构**

| 类型           | 名称                   | 数量                   |
| -------------- | ---------------------- | ---------------------- |
| u2             | attribute_name_index   | 1                      |
| u4             | attribute_length       | 1                      |
| u2             | max_stack              | 1                      |
| u2             | max_locals             | 1                      |
| u4             | code_length            | 1                      |
| u1             | code                   | code_length            |
| u2             | exception_table_length | 1                      |
| exception_info | exception_table        | exception_table_length |
| u2             | attributes_count       | 1                      |
| attribute_info | attributes             | attributes_count       |

**属性表结构**

| 类型 | 名称                 | 数量             |
| ---- | -------------------- | ---------------- |
| u2   | attribute_name_index | 1                |
| u4   | attribute_length     | 1                |
| u1   | info                 | attribute_length |



**Exceptions属性结构**

| 类型 | 名称                  | 数量                 |
| ---- | --------------------- | -------------------- |
| u2   | attribute_name_index  | 1                    |
| u4   | attribute_length      | 1                    |
| u2   | number_of_exceptions  | 1                    |
| u2   | exception_index_table | number_of_exceptions |

**LineNumberTable属性结构**

| 类型              | 名称                     | 数量                     |
| ----------------- | ------------------------ | ------------------------ |
| u2                | attribute_name_index     | 1                        |
| u4                | attribute_length         | 1                        |
| u2                | line_number_table_length | 1                        |
| lline_number_info | line_number_table        | line_number_table_length |

**line_number_info**

| 类型 | 名称        | 数量 |
| ---- | ----------- | ---- |
| u2   | start_pc    | 1    |
| u2   | line_number | 1    |

**LocalVariableTable属性结构**

| 类型                | 名称                        | 数量                        |
| ------------------- | --------------------------- | --------------------------- |
| u2                  | attribute_name_inidex       | 1                           |
| u4                  | attribute_length            | 1                           |
| u2                  | local_variable_table_length | 1                           |
| local_variable_info | local_variable_table        | local_variable_table_length |



**Local_variable_info项目结构**

| 类型 | 名称             | 数量 |
| ---- | ---------------- | ---- |
| u2   | start_pc         | 1    |
| u2   | length           | 1    |
| u2   | name_index       | 1    |
| u2   | descriptor_index | 1    |
| u2   | index            | 1    |

**LocalVariableTypeTable属性结构**

| 类型                | 名称                        | 数量                        |
| ------------------- | --------------------------- | --------------------------- |
| u2                  | attribute_name_inidex       | 1                           |
| u4                  | attribute_length            | 1                           |
| u2                  | local_variable_table_length | 1                           |
| local_variable_info | local_variable_table        | local_variable_table_length |

**Local_variable_type_info项目结构**

| 类型 | 名称       | 数量 |
| ---- | ---------- | ---- |
| u2   | start_pc   | 1    |
| u2   | length     | 1    |
| u2   | name_index | 1    |
| u2   | Signature  | 1    |
| u2   | index      | 1    |

**SourceFile属性结构**

| 类型 | 名称                 | 数量 |
| ---- | -------------------- | ---- |
| u2   | attribute_name_index | 1    |
| u2   | attribute_length     | 1    |
| u2   | sourcefile_index     | 1    |

**SourceDebugExtension属性结构**

| 类型 | 名称                              | 数量 |
| ---- | --------------------------------- | ---- |
| u2   | attribute_name_index              | 1    |
| u4   | attribute_length                  | 1    |
| u1   | debug_extension[attribute_length] | 1    |

**ConstantValue属性结构**

| 类型 | 名称                 | 数量 |
| ---- | -------------------- | ---- |
| u2   | attribute_name_index | 1    |
| u4   | attribute_length     | 1    |
| u2   | constantvalue_index  | 1    |

**InnerClasses属性结构**

| 类型             | 名称                 | 数量              |
| ---------------- | -------------------- | ----------------- |
| u2               | attribute_name_index | 1                 |
| u4               | attribute_length     | 1                 |
| u2               | number_of_classes    | 1                 |
| Inner_class_info | Inner_class          | number_of_classes |

**inner_classes_info表的结构**

| 类型 | 名称                     | 数量 |
| ---- | ------------------------ | ---- |
| u2   | inner_class_info_index   | 1    |
| u2   | outer_class_info_index   | 1    |
| u2   | inner_name_index         | 1    |
| u2   | inner_class_access_flags | 1    |

**inner_class_access_flags标志**

| 标志名称       | 标志值 |                                |
| -------------- | ------ | ------------------------------ |
| ACC_PUBLIC     | 0x0001 | 内部类是否为public             |
| ACC_PRIVATE    | 0x0002 | 内部类是否为private            |
| ACC_PROTECTED  | 0x0004 | 内部类是否为protected          |
| ACC_STATIC     | 0x0008 | 内部类是否为static             |
| ACC_FINAL      | 0x0010 | 内部类是否为final              |
| ACC_INTERFACE  | 0x0020 | 内部类是否为接口               |
| ACC_ABSTRACT   | 0x0400 | 内部类是否为abstract           |
| ACC_SYNTHETIC  | 0x1000 | 内部类是否并非由用户代码产生的 |
| ACC_ANNOTATION | 0x2000 | 内部类是不是一个注解           |
| ACC_ENUM       | 0x4000 | 内部类是不是一个枚举           |

**Deprecated及Synthetic属性结构**

| 类型 | 名称                 | 数量 |
| ---- | -------------------- | ---- |
| u2   | attribute_name_index | 1    |
| u4   | attribute_length     | 1    |

**StackMapTable属性结构**

| 类型            | 名称                    | 数量              |
| --------------- | ----------------------- | ----------------- |
| u2              | attribute_name_index    | 1                 |
| u4              | attribute_length        | 1                 |
| u2              | number_of_entries       | 1                 |
| stack_map_frame | stack_map_frame_entries | number_of_entries |

**Signature属性结构**

| 类型 | 名称                 | 数量 |
| ---- | -------------------- | ---- |
| u2   | attribute_name_index | 1    |
| u4   | attribute_length     | 1    |
| u2   | signature_index      | 1    |

**BootstrapMethods属性结构**

| 类型             | 名称                  | 数量                  |
| ---------------- | --------------------- | --------------------- |
| u2               | attribute_name_index  | 1                     |
| u4               | attribute_length      | 1                     |
| u2               | num_bootstrap_methods | 1                     |
| bootstrap_method | bootstrap_methods     | num_bootstrap_methods |

**bootstrap_method属性结构**

| 类型 | 名称                    | 数量                    |
| ---- | ----------------------- | ----------------------- |
| u2   | bootstrap_method_ref    | 1                       |
| u2   | num_bootstrap_arguments | 1                       |
| u2   | bootstrap_arguments     | num_bootstrap_arguments |

**Methodparameters属性结构**

| 类型      | 名称                 | 数量            |
| --------- | -------------------- | --------------- |
| u2        | attribute_name_index | 1               |
| u4        | attribute_length     | 1               |
| u1        | parameter_count      | 1               |
| parameter | parameters           | parameter_count |

**parameter属性结构**

| 类型 | 名称         | 数量 |
| ---- | ------------ | ---- |
| u2   | name_index   | 1    |
| u2   | access_flags | 1    |

**Module属性结构**

| 类型    | 名称                 | 数量           |
| ------- | -------------------- | -------------- |
| u2      | attribute_name_index | 1              |
| u4      | attribute_length     | 1              |
| u2      | module_name_index    | 1              |
| u2      | module_flags         | 1              |
| u2      | module_version_index | 1              |
| u2      | requires_count       | 1              |
| require | requires             | requires_count |
| u2      | exports_count        | 1              |
| export  | exports              | exports_count  |
| u2      | opens_count          | 1              |
| open    | opens                | opens_count    |
| u2      | uses_count           | 1              |
| use     | uses_index           | uses_count     |
| u2      | provides_count       | 1              |
| private | provides             | provides_count |

**exports属性结构**

| 类型   | 名称             | 数量             |
| ------ | ---------------- | ---------------- |
| u2     | exports_index    | 1                |
| u2     | exports_flags    | 1                |
| u2     | exports_to_count | 1                |
| export | exports_to_index | exports_to_count |

**ModulePackages属性结构**

| 类型 | 名称                 | 数量          |
| ---- | -------------------- | ------------- |
| u2   | attribute_name_index | 1             |
| u4   | attribute_length     | 1             |
| u2   | pckage_count         | 1             |
| u2   | pcakge_index         | package_count |

**ModulePackages属性结构**

| 类型 | 名称                 | 数量          |
| ---- | -------------------- | ------------- |
| u2   | attribute_name_index | 1             |
| u4   | attribute_length     | 1             |
| u2   | package_count        | 1             |
| u2   | package_index        | package_count |

**ModuleMainClass属性结构**

| 类型 | 名称                 | 数量 |
| ---- | -------------------- | ---- |
| u2   | attribute_name_index | 1    |
| u4   | attribute_length     | 1    |
| u2   | main_class_index     | 1    |

**RuntimeVisibleAnnotations属性结构**

| 类型       | 名称                 | 数量            |
| ---------- | -------------------- | --------------- |
| u2         | attribute_name_index | 1               |
| u4         | attribute_length     | 1               |
| u2         | num_annotations      | 1               |
| annotation | annotations          | num_annotations |

**annotation属性结构**

| 类型               | 名称                    | 数量                    |
| ------------------ | ----------------------- | ----------------------- |
| u2                 | type_index              | 1                       |
| u2                 | num_element_value_pairs | 1                       |
| element_value_pair | element_value_pairs     | num_element_value_pairs |

