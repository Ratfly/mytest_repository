## 一、正则表达式－－－煜泽 ##
---------------------------
### 会议内容
```
1、正则表达式简介
2、使用正则表达式
```
----------------------------
### 技术分享
----------
* 正则表达式是用来做什么的？
```
eg: 你正在搜索一个文件，想把文件里所有的单词car(不区分字母大小写)都找出来，但是不想把包含着字符串car的其他单词（carry）也找出来
eg: 正在编辑一段源代码并且要把所有的size都替换为isize,但是仅限于单词size本身而不涉及那些包含着字符串size的其他单词。
```
* 正则表达式是什么？
```
    正则表达式是一些用来匹配和处理文本的字符串；
	正则表达式使用正则表达式语言创建的；
	与其他程序设计语言一样，正则表达式语言也有需要使用者去学习的特殊的语法和指令；
    正则表达式语言并不是一种完备的程序设计语言，它甚至算不上是一种能够直接安装并运行的程序。
    正则表达式语言是内置与其他语言或软件产品里的“迷你”语言；
    eg:    \b[Cc][Aa][Rr]\b     用于搜索中文本中car的正则表达式
```
----------

##### 1、匹配单个字符 #####
	
    使用说明：
      1 正则表达式可以包含纯文本（甚至可以只包含纯文本）
      2 正则表达式是区分大小写的（如果不想区分，需要在所使用的语言或工具里进行设置）；
      3 .字符可以匹配任何单个字符、字母、数字甚至是.字符本身；
      4 \.对.进行了转义，用于表示.本身。（\字符是一个元字符，表示这个字符有特殊含义，而不是字符本身的含义）

例子：

    文本：
      Hello, my name is Ben. Please visit my website at 
    正则表达式： Ben
    结果：
      Hello, my name is Ben. Please visit my website at 

    文本：sales.xls
          sales1.xls
          sales2.xls
    正则表达式：sales.
    结果：sales.xls(匹配)
          sales1.xls(匹配)
          sale2.xls
----------------------------------------------
    文本：
      na1.xls
      na2.xls
      sa1.xls  
      apac1.xls 
    正则表达式： .a.\.xls

    结果：
      na1.xls(匹配)
      na2.xls(匹配)
      sa1.xls(匹配)
      apac1.xls
      
##### 2、匹配一组字符 #####

    使用说明： 
      1 元字符[和]用来定义一个字符集合（其含义是必须匹配该集合里的字符之一）
      2 定义一个字符集合的方法：	  
         a. 把所有的字符都列举出来；
         b. 利用元字符-以字符区间的方式给出。
      3 字符集合可以用元字符^来求非；（除了该字符集合里的字符，其他字符都可以被匹配）
  
例子：

    文本：
      sam.xls
      na1.xls
      na2.xls
      sa1.xls
      ca1.xls
    正则表达式： [ns]a[0123456789]\.xls
    结果：
      sam.xls
      na1.xls(匹配)
      na2.xls(匹配)
      sa1.xls
      ca1.xls
------
    正则表达式： [ns]a[0-9]\.xls
    结果：   sam.xls
             na1.xls
             na2.xls
             sa1.xls
             ca1.xls
    [0-9]与[0123456789]完全等价；
    其他合法的字符区间：
    A-Z, 匹配从A到Z的所有大写字母；
    a-z, 匹配从a到z的所有小写字母；
    A-F, 匹配从A到F的所以大写字母；
    A-z, 匹配从ASCII字符从A到z之间的所有字符。

-----
    文本：
      sam.xls
      na1.xls
      na2.xls
      sa1.xls
      ca1.xls
    正则表达式： [ns]a[^0-9]\.xls
    结果：
      sam.xls(匹配)
      na1.xls
      na2.xls
      sa1.xls
      ca1.xls
      
##### 3、使用元字符 #####
    使用说明：
      1 元字符是一些在正则表达式里有着特殊含义的字符，（如：.,[,],等），他们无法表达自身，因此需要用\对他们进行转义。
      2 空白元字符：
      [\f] 换页符     [\n] 换行符
      [\r] 回车符     [\t] 制表符（Tab键）
      [\v] 垂直制表符
      3 匹配特定字符类别
      数字元字符：     \d 任何一个数字字符
                       \D 任何一个非数字字符
      字母数字元字符： \w 任何一个字母数字（大小写均可）或下划线
                       \W 任何一个非字母数字或非下划线字符
      空白字符元字符： \s 任何一个空白字符
                       \S 任何一个非空白字符
      4 使用这些简短的元字符可以用来简化正则表达式；

##### 4、重复匹配 #####
    使用说明：
      1 + 元字符匹配字符或字符集合的一次或多次重复出现；
      2 * 元字符匹配字符或字符集合的零次或多次重复出现；
      3 ？元字符匹配字符或者字符集合的零次或一次出现；

例子：
```
文本：
   Send personal email to ben@forta.com or ben.forta@forta.com. For questions about book use support@forta.com.
正则表达式： [\w.]+@[\w.]+\.\w+
结果：
Send personal email to (ben@forta.com) or (ben.forta@forta.com). For questions about book use (support@forta.com).
 分析：
[\w.]+将匹配字符集合[\w.](字母数字字符、下划线和.)的一次或者多次重复出现；
```
```
文本：
    Hello .ben@forta.com is my email address.
正则表达式：\w[\w.]*@[\w.]+\.\w+
结果：
    Hello .ben@forta.com is my email address.

分析：
  \w负责匹配电子邮件地址里的第一个字符；
  [\w.]*负责匹配电子邮件里第一个字符之后，@字符之前的所有字符；
  （若使用正则表达式：[\w.]+@[\w.]+\.\w+, 则结果为：
  Hello .(ben@forta.com) is my email address “.”作为电子邮件里的第一个字符不合法）

```
```
文本：
   The URL is http://www.forta.com/, to connect securely use      https://www.forta.com/ instead.
正则表达式：https?://[\w./]+
结果：
   The URL is http://www.forta.com/ , to connect securely use https://www.forta.com/ instead.
分析：
  ?表示前面的字符（s）要么不出现，要么最多出现一次.
 （若使用正则表达式： http://[\w./]+，则结果为：
   The URL is http://www.forta.com/, to connect       securely use      https://www.forta.com/ instead. ）
```

##### 5、位置匹配 #####
    使用说明：
      1 正则表达式不仅可以用来匹配任意长度的文本块，还可以用来匹配出现在字符串中特定位置的文本；
      2 单词边界：\b用来匹配一个单词的开始或结尾；
      3 字符串边界：^用来匹配字符串开头，$用来匹配字符串的结尾；
      4 分行匹配：^用来匹配行的开始，$用来匹配行的结尾。

例子：

    文本：
       The cat scattered his food all over the room.
    正则表达式：\bcat\b
    结果：
       The cat scattered his food all over the room.

    分析：
      如果正则表达式为cat,那么结果为：
      The cat scattered his food all over the room.

##### 使用子表达式 #####

    使用说明：

      1 子表达式是一个更大的表达式的一部分；（把一个表达式划分为一系列子表达式的目的是为了把那些子表达式作为一个独立元素来使用。）
      2 子表达式必须用（和）括起来；

例子：

    文本：
      Hello, my name is Ben&nbsp;Forta, and I am the    author of books on SQL, ColdFusion, WAP, Windows &nbsp;&nbsp; 2000, and other subjects.
    正则表达式: (&nbsp;){2,}
    结果：
      Hello, my name is Ben&nbsp;Forta, and I am the    author of books on SQL, ColdFusion, WAP, Windows &nbsp;&nbsp; 2000, and other subjects.
    分析：
      (&nbsp;)是一个子表达式，它将被视为一个独立元素，
      紧跟其后的{2，}表示将寻找此独立元素的至少2次连续出现。

##### 6、回溯引用 #####

      文本：
        This is a block of of text, several words here are are repeated, and and they should not be.
      正则表达式：
      [ ]+(\w+)[ ]+\1
      结果：
      This is a block (of of) text, several words here (are are) repeated, (and and) they should not be.

     分析：
      \1代表什么？它代表第一个子表达式。
      以此类推，\2代表第二个子表达式，\3代表第三个子表达式。所以，上述正则表达式将匹配同一个单词的连续两次重复出现。 




