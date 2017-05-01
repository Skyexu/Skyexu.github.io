---
title: Jena学习笔记（二） SPARQL
date: 2016/11/29 22:11:29  
tags: Jena
categories: paper

---


> 官方描述：Apache Jena（或简称Jena）是一个用于构建语义Web和关联数据应用程序的自由和开源的Java框架。 该框架由不同的API组成，用于处理RDF数据。
> 
Jena是一个用于Java语义Web应用程序的API（应用程序编程接口）。它不是一个程序或工具，如果这是你正在寻找，我建议或许TopBraid Composer作为一个好的选择。因此，Jena的主要用途是帮助您编写处理RDF和OWL文档和描述的Java代码。

> SPARQL是用于访问由W3C RDF数据访问工作组设计的RDF的查询语言和协议。

> 作为一种查询语言，SPARQL是“数据导向的”，因为它只查询模型中保存的信息;在查询语言本身没有推理。当然，Jena模型是“聪明的”，因为它提供了某些三元组存在的印象，即按需创建它们，包括OWL推理。除了以查询的形式获取应用程序想要的描述外，SPARQL不执行任何操作，并以一组bindings或RDF图形的形式返回该信息。

官方网站：http://jena.apache.org/index.html
SPARQL教程：http://jena.apache.org/tutorials/sparql.html
<!-- more -->
---

### 设置环境变量
```
Setting up your Environment
An environment variable JENAROOT is used by all the command line tools to configure the class path automatically for you. You can set this up as follows:

On Linux / Mac

export JENAROOT=the directory you downloaded Jena to
export PATH=$PATH:$JENAROOT/bin
On Windows

SET JENAROOT=the directory you downloaded Jena to
SET PATH=%PATH%;%JENAROOT%\bat
```
### 数据格式
首先，我们需要清楚查询要查询的数据。 SPARQL查询RDF图。 RDF图是一组三元组（Jena调用RDF图“模型”和三元组“语句”，因为这是他们在第一次设计Jena API时调用的）。

重要的是要意识到三元组的重要性，而不是序列化。序列化只是一种写三元组的方式。 RDF / XML是W3C的建议，但是可能很难看到序列化形式的三元组，因为有多种方法来编码同一个图。在本教程中，我们使用了一个更像“三元组”的序列化，称为Turtle（另请参阅W3C语义网络引文中描述的N3语言）。

我们将从vc-db-1.rdf中的简单数据开始：此文件包含用于多个vCard人员描述的RDF。 vCard在RFC2426中描述，并且RDF翻译在W3C笔记“在RDF / XML中表示vCard对象”中描述。我们的示例数据库只包含一些名称信息。

图形上，数据看起来像：
![](http://i.imgur.com/Nd0fFky.png)

在三元组中，如下所示：
```
@prefix vCard:   <http://www.w3.org/2001/vcard-rdf/3.0#> .
@prefix rdf:     <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix :        <#> .

<http://somewhere/MattJones/>
    vCard:FN    "Matt Jones" ;
    vCard:N     [ vCard:Family
                              "Jones" ;
                  vCard:Given
                              "Matthew"
                ] .

<http://somewhere/RebeccaSmith/>
    vCard:FN    "Becky Smith" ;
    vCard:N     [ vCard:Family
                              "Smith" ;
                  vCard:Given
                              "Rebecca"
                ] .

<http://somewhere/JohnSmith/>
    vCard:FN    "John Smith" ;
    vCard:N     [ vCard:Family
                              "Smith" ;
                  vCard:Given
                              "John"
                ] .

<http://somewhere/SarahJones/>
    vCard:FN    "Sarah Jones" ;
    vCard:N     [ vCard:Family
                              "Jones" ;
                  vCard:Given
                              "Sarah"
                ] .
```
更加明确的：
```
@prefix vCard:   <http://www.w3.org/2001/vcard-rdf/3.0#> .
@prefix rdf:     <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .

<http://somewhere/MattJones/>  vCard:FN   "Matt Jones" .
<http://somewhere/MattJones/>  vCard:N    _:b0 .
_:b0  vCard:Family "Jones" .
_:b0  vCard:Given  "Matthew" .


<http://somewhere/RebeccaSmith/> vCard:FN    "Becky Smith" .
<http://somewhere/RebeccaSmith/> vCard:N     _:b1 .
_:b1 vCard:Family "Smith" .
_:b1 vCard:Given  "Rebecca" .

<http://somewhere/JohnSmith/>    vCard:FN    "John Smith" .
<http://somewhere/JohnSmith/>    vCard:N     _:b2 .
_:b2 vCard:Family "Smith" .
_:b2 vCard:Given  "John"  .

<http://somewhere/SarahJones/>   vCard:FN    "Sarah Jones" .
<http://somewhere/SarahJones/>   vCard:N     _:b3 .
_:b3 vCard:Family  "Jones" .
_:b3 vCard:Given   "Sarah" .
```
重要的是要意识到这些是相同的RDF图，并且图中的三元组没有特定的顺序。计算机不在乎其顺序。

### 第一个SPARQL查询
看一个简单的查询并展示如何使用Jena执行它。
```
SELECT ?x
WHERE { ?x  <http://www.w3.org/2001/vcard-rdf/3.0#FN>  "John Smith" }
```

用命令行查询应用程序执行所述查询:
```
---------------------------------
| x                             |
=================================
| <http://somewhere/JohnSmith/> |
---------------------------------
```

这通过将WHERE子句中的三元模式与RDF图中的三元组进行匹配来实现。三元组的谓词和对象是固定值，因此模式将只匹配与这些值的三元组。主体是一个变量，并且对变量没有其他限制。模式匹配任何三元组与这些谓词和对象值，它匹配x的结果。

<>中包含的项目是一个URI（实际上是一个IRI），而包含在“”中的项目是一个普通的字面量。就像Turtle，N3或N-triples一样，输入的文字用\ ^ \ ^编写，语言标签可以用@添加。

？x是一个称为x的变量。？不会形成名称的一部分，这就是为什么它不会出现在表的输出中。
该查询返回x查询变量中的匹配项。所示的输出是通过一条ARQ的命令获得的。

#### 执行查询
```
Windows setup
Execute:

bat\sparql.bat --data=doc\Tutorial\vc-db-1.rdf --query=doc\Tutorial\q1.rq
You can just put the bat/ directory on your classpath or copy the programs out of it.

bash scripts for Linux/Cygwin/Unix
Execute:

bin/sparql --data=doc/Tutorial/vc-db-1.rdf --query=doc/Tutorial/q1.rq
```
在Linux中执行结果
![](http://i.imgur.com/HvWk6xW.png)

#### 获取所有人的FullName
```
SELECT ?x ?fname
WHERE {?x  <http://www.w3.org/2001/vcard-rdf/3.0#FN>  ?fname}
```
```
//执行
ubuntu@master:~/paper/doc$ sparql --data=vc-db-1.rdf --query=q-bp1.rq
----------------------------------------------------
| x                                | fname         |
====================================================
| <http://somewhere/JohnSmith/>    | "John Smith"  |
| <http://somewhere/SarahJones/>   | "Sarah Jones" |
| <http://somewhere/MattJones/>    | "Matt Jones"  |
| <http://somewhere/RebeccaSmith/> | "Becky Smith" |
----------------------------------------------------

```



#### 指定前缀
```
ubuntu@master:~/paper/doc$ cat q2.rq 
PREFIX vCard:      <http://www.w3.org/2001/vcard-rdf/3.0#>

SELECT ?x
WHERE
 { ?x vCard:FN "John Smith" }
```
```
ubuntu@master:~/paper/doc$ sparql --data=vc-db-1.rdf --query=q2.rq
---------------------------------
| x                             |
=================================
| <http://somewhere/JohnSmith/> |
---------------------------------

```
