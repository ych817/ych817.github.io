---
title: 翻译-EOPL-01-递归的数据集
subtitle:
date: 2025-04-02T00:03:07+13:00
slug: 5cce8df
draft: false
author:
  name: 沉积岩
  link:
  email:
  avatar:
description:
keywords:
license:
comment: false
weight: EOPL01
tags:
  - 翻译
  - EOPL
  - 函数式编程
  - Racket
  - Lisp
collections:
  - 翻译-EOPL
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromRelated: false
hiddenFromFeed: false
summary: 
toc: true
math: true
lightgallery: false
# password: 
# message: 
repost:
  enable: true
  url:
linkToMarkdown: false

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

$$
\rm\LaTeX\textrm{ Definitions are here.}
\def\Int{\textrm{Int}}
\def\Expression{\textrm{Expression}}
\def\SList{\textrm{S-list}}
\def\SExp{\textrm{S-exp}}
\def\Symbol{\textrm{Symbol}}
\def\Bintree{\textrm{Bintree}}
\def\LcExp{\textrm{LcExp}}
\def\Iden{\textrm{Identifier}}
\def\BinSearchTree{\textrm{Binary-search-tree}}
\def\InductHyp{\textit{IH}}
\def\List{\textrm{List}}
\def\ScmVal{\textrm{Scheme\_value}}
\def\ListOfInt{\textrm{List-of-Int}}
\def\ListOfSymbol{\textrm{List-of-Symbol}}
\def\ListOf#1{\textrm{List-of-(#1)}}
\def\VectorOf#1{\textrm{Vector-of(#1)}}
\def\implement#1{\lceil#1\rceil}
\def\DiffTree{\textrm{Diff-Tree}}
\def\EnvExp{\textrm{Env-exp}}
\def\N{\mathbb{N}}
$$

解释器、检查器及其他类似程序是编程语言处理器的核心 。本章将介绍这些程序用到的基本编程工具 。

由于编程语言的语法通常具有嵌套或树状结构 ,
因此递归将成为我们的主要手段 。[1.1 节](#S1.1)及 
[1.2 节](#S1.2)将介绍递归定义数据结构的方法
并展示如何利用这类定义来指导递归程序的构建 ; 而 
[1.3 节](#S1.3) 则会介绍如何将这些技术推广到更复杂的问题上 。
本章末尾提供了大量练习 , 它们是本章的核心内容 : 它们能为掌握
递归编程技巧提供必要的实践经验 , 而这正是阅读后续内容所依赖的 。

## 1.1 递归定义的数据 {#S1.1}

在书写一段程序或一个过程时你必须清楚地认识到 : 
什么样的值能作为过程的实参 , 而什么样的值又能作为合法的返回值 。
这些值构成的集通常很复杂 。本节将介绍定义这些集合的形式化技术 。

### 1.1.1 递归定义 {#S1.1.1}

**递归定义 Inductive specification** 是一种定义集合的强大方法 。为了介绍它
我们先试着用它描述自然数 $\N=\{0,1,2,\ldots\}$ 的某一子集 $S$ 。

{{< raw >}}<a id="Def1.1.1"></a>{{< /raw >}}**定义 1.1.1** 规定 $n\in S$ 当且仅当

1. $n=0$ , 或者
2. $n-3\in S$

看看如何用上述定义判断哪些自然数会落在 $S$ 中 。
首先由 $0\in S$ 可知 $3\in S$ ( 因为 $3-3=0\in S$ ) ,
另外 $6\in S$ ( 因为 $6-3=3\in S$ ) 。继续这样做下去便可得知
所有能被 $3$ 整除的自然数都落在 $S$ 中 。

那其他自然数呢 ? $1\in S$ 是否成立呢 ? 
首先由 $1\neq 0$ 可知条件 1 不成立 ; 
另外 $1-3=-2$ 非自然数 , 故 $-2$ 非 $S$ 中元素 , 条件 2 也不成立 ; 
既然两个条件对 $1$ 都不成立 , 故 $1\notin S$ 。
同样地我们有 $2\notin S$ 。
那 $4$ 呢 ? $4\in S$ 当且仅当 $1\in S$ , 但 $1\notin S$ , 
从而 $4\notin S$ 。用类似的方法不难得知 : 若 $n$ 是自然数并且 
$n$ 非 $3$ 倍数则 $n\notin S$ 。

综上 $S$ 便是所有能被 $3$ 整除的自然数所构成的集 。

我们可根据[定义 1.1.1](#Def1.1.1) 编写一个过程以判断自然数是否落在 $S$ 中 :

```scheme {data-open=true}
;; 过程合约 :  in-S? : Natural   → Bool
;; 过程用途 : (in-S?     n   )   = #t 仅当 n 属于 S 的话 , 否则为 #f
;; 实参语法 :          Natural ::=  0 | (succ Natural)
(define in-S? 
  (lambda (n)
    (if (zero? n) #t
        (if (>= (- n 3) 0) (in-S? (- n 3))
            #f))))
```

刚才我们根据[定义 1.1.1](#Def1.1.1) 编写了一个递归过程 , 
代码中的 `in-S? : Natural → Bool` 是一条注释 , 称作过程的**合约 contract** : 
它表示过程 `in-S?` 应以一个以自然数为实参并返回一个布尔值 。
这样的注释对阅读和编写代码很有帮助 。

在判断 $n\in S$ 是否成立前 , 首先应判断 $n=0$ 是否成立 : 
如果成立则返回真 , 否则则需要判断 $n-3\in S$ 是否成立 ; 
要实现后者首先需判断 $n-3\geq 0$ 是否为真 : 
若为真则可调用上述过程判断其是否属于 $S$ , 
否则 $n\notin S$ 。

下面是另一种定义 $S$ 的方法 。

{{< raw >}}<a id="Def1.1.2"></a>{{< /raw >}}**定义 1.1.2** 
规定 $S$ 为自然数 $\N$ 子集且为满足下述条件的最小集 :

1. $0\in S$ 
2. 若 $n\in S$ 则 $n+3\in S$

刚才的 " 最小集 " 意味着该集合不仅满足上述两条性质 , 
并且还是其他任何满足上述两条性质的集的子集 。
很明显这样的集唯一 : 如果 $S_1$ , $S_2$ 都满足上述条件且都为最小集 , 
那么 $S_1\subseteq S_2$ ( 因为 $S_1$ 是最小集 ) 
并且 $S_1\supseteq S_2$ ( 因为 $S_2$ 是最小集 ) , 
如此 $S_1=S_2$ 。该条件不可或缺 , 否则 $S$ 就不唯一了 
( 见[练习 1.3](#Xer1.3) ) 。

下面是上述定义的另一种记法 : 

1. $\begin{prooftree}
   \AXC{}
   \UIC{$0\in S$}
   \end{prooftree}$
2. $\begin{prooftree}
   \AXC{$n\in S$}
   \UIC{$(n+3)\in S$}
   \end{prooftree}$ 

这不过是[定义 1.1.2](#Def1.1.2) 的简便记法罢了 。上面的每个公式称作**推理规则 rule of inference** , 简称**规则 rule** ;
水平线可以被解释为 " 若 . . . 则 . . . " ; 水平线上方的部分称作**假设 hypothesis** 或者是**前件 antecedent** , 
下方部分则称作是**结论 conclusion** 或者是**后件 consequent** 。若有多个假设 , 则使用隐藏的 " 且 " ( 见
[定义 1.1.5](#Def1.1.5) ) 来连接它们 ; 没有假设的规则称作**公理 axiom** , 在书写公理时常常会省略水平线 , 即

$\qquad\begin{prooftree}
\AXC{$0\in S$}
\end{prooftree}$

上述两条规则可解释为如下 : " 自然数 $n$ 属于 $S$ 当且仅当断言 ' $n\in S$ ' 可通过在公理上有限次地应用
推理规则得出 " 。这一解释使得 $S$ 可以很自然地被人理解为闭合于该规则的最小集合 。

刚才的两个定义描述的都是同一个对象 。我们通常会把版本 1 称作是**自顶向下 top-down** 的 , 并且把
版本 2 称作是**自底向上 bottom-up** 的 。版本 3 则称作是**推理规则 rules-of-inference** 形式的 。

来看看上述三种定义方式在其他例子中的使用 。

{{< raw >}}<a id="Def1.1.3"></a>{{< /raw >}}**定义 1.1.3** ( 整数列表 , 自顶向下 ) Scheme 中的列表为整数列表当且仅当其

1. 要么是空列表
2. 要么是个配对 , 满足 `car` 为自然数且 `cdr` 为整数列表

{{< raw >}}<a id="Def1.1.4"></a>{{< /raw >}}**定义 1.1.4** ( 整数列表 , 自底向上 ) $\ListOfInt$ 为满足下述条件的最小集 :

1. $\texttt{()}\in \ListOfInt$ , 并且
2. 若 $n\in \Int$ 并且 $l\in \ListOfInt$ 则 $\texttt{($n$ . $l$)}\in \ListOfInt$ 

这里我们用中缀符 " $\texttt{.}$ " 来代表 Scheme 中 `cons` 的输出结果 ; 语句 $\texttt{($n$ . $l$)}$ 表示一个头 ( `car` ) 为 $n$ 
且尾 ( `cdr` ) 为 $l$ 的 Scheme 序对 。

{{< raw >}}<a id="Def1.1.5"></a>{{< /raw >}}**定义 1.1.5** ( 整数列表 , 推理规则 ) 

1. $\begin{prooftree}
   \AXC{$\texttt{()} \in \ListOfInt$}
   \end{prooftree}$
2. $\begin{prooftree}
   \AXC{$n\in \Int$}
   \AXC{$l\in \ListOfInt$} 
   \BIC{$\texttt{($n$ . $l$)}\in \ListOfInt$}
   \end{prooftree}$ 

上述三种定义是彼此等价的 。

接下来我们来看看如何用这些定义来生成整数列表 。

1. $\texttt{()}$ 是整数列表 , 由[定义 1.1.4](#Def1.1.4) / [定义 1.1.5](#Def1.1.5) 的条件 1 可得知 ;

2. $\texttt{(14 . ())}$ 是整数列表 —— 由于 $\texttt{14}$ 是整数且 $\texttt{()}$ 是
   整数列表 , 如此利用[定义 1.1.4](#Def1.1.4) 里的条件 2 便可以得知 ;

3. $\texttt{(3 . (14 . ()))}$ 是整数列表 —— 因为 $\texttt3$ 是整数且 
   $\texttt{(14 . ())}$ 是整数列表 。我们可将该推理过程写成是
   $\ListOfInt$ 的第二条推理规则的一个实例 , 即

   $\qquad\begin{prooftree}
   \AXC{$\texttt{3}\in \Int$}
   \AXC{$\texttt{(14 . ())}\in \ListOfInt$}
   \BIC{$\texttt{(3 . (14 . ()))}\in \ListOfInt$}
   \end{prooftree}$

4. $\texttt{(-7 . (3 . (14 . ())))}$ 是整数列表 : 由于 $\texttt{-7}$ 是
   整数且 $\texttt{(3 . (14 . ()))}$ 是整数列表 , 因此基于定义 
   1.1.4 的条件 2 便可得知 。

5. 不遵循上述风格方式构造出的对象都不是整数列表 。

如果是将点表示法转为列表表示法则易知 $\texttt{()}$ , $\texttt{(14)}$ , $\texttt{(3 14)}$ , $\texttt{(-7 3 14)}$ 皆为 $\ListOfInt$ 中元素 。

我们甚至还可将所有用到的推理规则结合起来以获得 $\texttt{(-7 . ( 3 . (14 . ()))}\in \ListOfInt$ 的
演绎推理链 。下面的树状图称作是**推导 derivation** 或者是**证明树 deduction tree** 。

$\qquad \begin{prooftree}
\AXC{$\texttt{-7}\in \N$}
\AXC{$\texttt{3} \in \N$}
\AXC{$\texttt{14}\in \N$}
\AXC{$\texttt{()} \in \ListOfInt$}
\BIC{$\texttt{(14 . ())}\in \ListOfInt$}
\BIC{$\texttt{(3 . (14 . ()))}\in \ListOfInt$}
\BIC{$\texttt{(-7 . (3 . (14 . ())))}\in \ListOfInt$}
\end{prooftree}$

{{< raw >}}<a id="Xer1.1"></a>{{< /raw >}}**练习 1.1** `[**]`\
写出下述集合的递归定义 。以三种方式 ( 自顶向下 / 自底向上 / 推理规则 ) 写出每个定义 , 并且用你的规则推导出下述集合可能包含的元素 。

1.  $\{3n+2\mid n\in \N\}$
  
    >   [!note]- 答 : 
    > 
    > - 自顶向下 : $n\in S$ 当且仅当下述两个条件中有一个成立
    > 
    >   1. $n = 2$
    >   2. $n - 3 \in S$
    > 
    > - 自底向上 : $S$ 为 $\N$ 的子集且为满足下述两个条件的最小集 :
    > 
    >   1. $2\in S$
    >   2. 若 $n\in S$ 则 $n+3\in S$
    > 
    > - 推理规则 : 
    > 
    >   1. $\begin{prooftree}
            \AXC{}
            \UIC{$2\in S$}
            \end{prooftree}$
    >   2. $\begin{prooftree}
            \AXC{$n\in S$}
            \UIC{$n+3\in S$}
            \end{prooftree}$

2.  $\{2n+3m+1\mid n,m\in \N\}$

    > [!note]- 答 :
    > 
    > - 自顶向下 : $n\in S$ 当且仅当下述三个条件有一个成立
    > 
    >   1. $n=1$
    >   2. $n-2\in S$ 或 $n-3\in S$
    > 
    > - 自底向上 : $S$ 为 $\N$ 的子集且为满足下述两个条件的最小集 :
    > 
    >   1. $1\in S$
    >   2. 若 $n\in S$ 则 $n+2\in S$ 且 $n+3\in S$
    > 
    > - 推理规则 : 
    > 
    >   1. $\begin{prooftree}
           \AXC{}
           \UIC{$1\in X$}
           \end{prooftree}$
    >   2. $\begin{prooftree}
           \AXC{$n\in S$}
           \UIC{$n+2\in S,~ n+3\in S$}
           \end{prooftree}$ 

3.  $\{(n,2n+1)\mid n\in \N\}$

    >  [!note]- 答 :
    >
    >  - 自顶向下 : 一对自然数 $(n,m)\in S$ 当且仅当其下述两个条件中有一个成立 :
    >    1. $n=0$ 且 $m=1$ , 
    >    2. $(n-1,m-2)\in S$ 
    > 
    >  - 自底向上 : $S$ 为 $\N$ 的子集且为满足下述两个条件的最小集 :
    >  
    >    1. $(0,1)\in S$
    >    2. 若 $(n,m)\in S$ 则 $(n+1,m+2)\in S$ 
    >  
    >  - 推理规则 : 
    >    1. $\begin{prooftree}
            \AXC{}
            \UIC{$(0,1)\in S$}
            \end{prooftree}$
    >    2. $\begin{prooftree}
            \AXC{$(n,m)\in S$}
            \UIC{$(n+1,m+2)\in S$}
            \end{prooftree}$

4.  $\{(n,n^2\mid n\in \N)\}$ —— 请勿在您的规则中提到平方运算 。
    提示 : 方程 $(n+1)^2=n^2+2n+1$ 会派上用场 。

    > [!note]- 答 :
    > 
    >  - 自顶向下 : 一对自然数 $(n,m)\in S$ 当且仅当其下述两个条件中有一个成立 :
    > 
    >    1. $n=0$ 且 $m=0$ , 
    >    2. $(n-1,m-2n+1)\in S$ 
    > 
    >  - 自底向上 : $S$ 为满足下述两个条件的包含 $\N$ 的最小集 :
    > 
    >    1. $(0,0)\in S$
    >    2. 若 $(n,m)\in S$ 则 $(n+1,m+2n+1)\in S$ 
    > 
    >  - 推理规则 : 
    >
    >    1. $\begin{prooftree}
            \AXC{}
            \UIC{$(0,1)\in S$}
            \end{prooftree}$
    >    2. $\begin{prooftree}
            \AXC{$(n,m)\in S$}
            \UIC{$(n+1,m+2n+1)\in S$}
            \end{prooftree}$

{{< raw >}}<a id="Xer1.2"></a>{{< /raw >}}**练习 1.2** `[**]`\
下面这些推理规则所定义的集合应是什么样子的呢 ? 为什么呢 ?

1. $\begin{prooftree}
   \AXC{}
   \UIC{$(0,1)\in S$}
   \end{prooftree} \qquad\begin{prooftree}
   \AXC{$(n,k)\in S$}
   \UIC{$(n+1,k+7)\in S$}
   \end{prooftree}$ 

   > [!note]- 答 :
   > 
   > $\{(n,7n+1)\mid n\in \N\}$

2. $\begin{prooftree}
   \AXC{}
   \UIC{$(0,1)\in S$}
   \end{prooftree} \qquad\begin{prooftree}
   \AXC{$(n,k)\in S$}
   \UIC{$(n+1,2k)\in S$}
   \end{prooftree}$

    > [!note]- 答 :
    > 
    > $\{(n,2^n)\mid n\in \N\}$

3. $\begin{prooftree} \AXC{} \UIC{$(0,0,1)\in S$} \end{prooftree}
   \qquad\begin{prooftree}
   \AXC{$(n,i,j)\in S$}
   \UIC{$(n+1,j,i+j)\in S$}
   \end{prooftree}$ 

    > [!note]- 答 :
    > 
    > $\{(n, F(n), F(n+1))\mid n\in \N\}$
    > 
    > 想知道这是怎么得出的 ? 见**差分方程组 difference equations**

4. $\begin{prooftree} \AXC{} \UIC{$(0,1,0)\in S$} \end{prooftree}
    \qquad\begin{prooftree} \AXC{$(n,i,j)\in S$} \UIC{$(n+1,i+2,i+j)\in S$} \end{prooftree}$ `[***]`

    > [!note]- 答 :
    >  
    > $\{(n,2n+1,n^2)\mid n\in \N\}$
    >
    > 想知道这是怎么得出的 ? 见**差分方程组 difference equations**

{{< raw >}}<a id="Xer1.3"></a>{{< /raw >}}**练习 1.3** `[*]`\
找出自然数的子集 $T$ 满足 $0\in T$ , $T\neq S$ 且对任何 $n\in T$ 都有 $n+3\in T$ —— 其中 $S$ 为[定义 1.1.2](#Def1.1.2) 
里提到的集合 。

> [!note]- 答 :
> 
> $T=\N$

### 1.1.2 用语法定义集合 {#S1.1.2}

刚才的例子还算简单 , 但是不难想象在描述更复杂的数据类型时会有多么麻烦 ;
为缓解这一问题 , 我们将展示如何用**语法 grammar** 来声明集合 。
语法常用来描述字符串构成的集合 , 但我们也可用它来定义
值构成的集合 。比如我们可以通过下述语法定义集 $\ListOfInt$ : 

$\qquad \begin{aligned}[t]
\ListOfInt &::= \texttt{()} \\
\ListOfInt &::= \texttt{(\Int  . \ListOfInt)}
\end{aligned}$

上面两条规则分别对应[定义 1.1.4](#Def1.1.4) 里的两条性质 :
规则 1 声称空列表 $\texttt{()}$ 属于 $\ListOfInt$ ,
规则 2 声称若 $n\in \Int$ 且 $l\in \ListOfInt$ 则 $\texttt{($n$ . $l$)}\in \ListOfInt$ 。两条规则构成的集合称作**语法 grammar** 。

现在来仔细看看上述语法 。在刚才的语法中我们有

- **非终结符 non-terminal symbol** :
  这些是被定义的集合的名称 。本例中虽然只提到了一个集合 ,
  但是在一般情况下可能会定义好几个集合 。这些集合也称作
  **语法类别 syntactic categories** 。

  在书中我们会将非终结符和集合名的首字母大写 ,
  但在提到它们的元素时则仍然使用小写字母 。这其实并不复杂 , 比如 :
  虽然 $\Expression$ 是非终结符 , 但我们依然会写出如 $e\in \Expression$ 
  或 " $e$ is an expression " 这样的语句 。

  另一种书写惯例叫做 **Backus-Naur Form** , 简称 **BNF** ,
  即在单词周围加尖括号 , 如 $\langle \textrm{expression} \rangle$ 。

- **终结符 terminal symbol** : 这些是集合外在表示中的字符 ,
  本例中则是 " $\texttt.$ " 、" $\texttt($ " 、" $\texttt)$ " 。
  这些字符都以打字机字体显示 , 如 " $\texttt{lambda}$ " 。

- **派生式 production** : 
  前面的推理规则在这里称作是派生式 。每个派生式中左侧是一个非终结符 ,
  右侧则包含终结符和非终结符 。左右两侧通常用符号 " $::=$ " 来分隔 ,
  解读为 " 是 " 或者 " 可以是 " 。
  派生式右侧声明了一种基于其他语法类别和终结符 
  ( 如 " $\texttt($ " 、" $\texttt)$ " 、" $\texttt.$ " ) 构建当前语法类别的元素的方式 。

若某些语法类别的含义在语境中足够清晰
则在派生式中提到它们时通常不作定义 ,
如 $\Int$ 。

语法通常以某种简写形式描述 。当一个派生式的左侧与前一派生式的左侧完全相同时 ,
我们通常会将其左侧省略 。因此上述语法基于此惯例可以写作

$\qquad \begin{aligned}[t]
\ListOfInt ::={}& \texttt{()} \\
          ::={}& \texttt{(\Int  . \ListOfInt)}
\end{aligned}$

也可只写一次 $::=$ 和左侧的内容并用特殊符号 " $\mid$ " ( 竖线 , 读作 " 或 " ) 分隔派生式右侧内容 , 即

$\qquad \begin{aligned}[t]
\ListOfInt
::={} & \texttt{()} %\\
\mid{} \texttt{(\Int  . \ListOfInt)}
\end{aligned}$ 

{{< raw >}}<a id="S1.1.2-Kleene"></a>{{< /raw >}}
另一种简写方式叫**克莱尼星号 Kleene star** , 记作 $\{\ldots\}^*$ 。当它位于派生式右侧时则意味着会有一串由
任意数目个花括号间内容下的实例构成的序列 。若用克莱尼星号 , 则前面 $\ListOfInt$ 的定义可简写为

$\qquad\begin{aligned}[t]
\ListOfInt ::={}& \{\Int\}^*
\end{aligned}$

这也涵盖了没有实例存在的情况 : 如果内容出现 $0$ 次则会获得空串 。

星号记法的另一种变体叫**克莱尼加号 Kleene plus** , 记作 $\{\ldots\}^+$ , 表示一串由单或多个实例构成的序列 。
上例中若把 $^+$ 换成 $^*$ , 则其定义的语法类型是非空的整数列表 。

星号记法的另一种变体叫**分隔符 separated list** 表示法 。例如 : $\{\Int\}^{*(c)}$ 表示一串含任意数目的 $\Int$ 中
的实例 , 并且它们以非空字符串 $c$ 分割 。同样该表示法也涵盖了没有实例的情况 : 如果有 $0$ 个实例那么
我们将得到空字符串 —— 例如 $\{\Int\}^{*(,)}$ 会含有述字符串 `8` , `14,12` , `7,3,14,16` 而 $\{\Int\}^{*(\texttt{;})}$ 则会含有
下述字符串 ` 8` , `14;12` , `7;3;14;16` 。这些简写并不是必须的 , 没有它们我们照样可以重写语法 。

假如集合由语法定义 , 那么可用**语法推导 syntactic derivation** 证明给定值为其成员 ; 推导从集合对应的
非终结符开始 : 在每一步 ( 以箭头 $\Rightarrow$ 指示 ) 中非终结符会被替换为相应规则的右侧或者是被替换为某个
语法类别的已知成员 —— 如果该语法类别未被明确定义的话 。例如前面关于 $\texttt{(14 . ())}$ 是整数列表
的证明若是用语法推导便可写成

$\qquad \begin{aligned}[t] &
\ListOfInt \\\Rightarrow{}& 
\texttt{(\Int  . \ListOfInt)} \\\Rightarrow{}& 
\texttt{(14 . \ListOfInt)}    \\\Rightarrow{}& 
\texttt{(14 . ())}
\end{aligned}$

非终结符的替换顺序不影响结果 。下面是 $\texttt{(14 . ())}$ 的另一种推导 :

$\qquad \begin{aligned}[t] &
\ListOfInt \\\Rightarrow{}& 
\texttt{(\Int  . \ListOfInt)} \\\Rightarrow{}& 
\texttt{(\Int . ())}    \\\Rightarrow{}& 
\texttt{(14 . ())}
\end{aligned}$

{{< raw >}}<a id="Xer1.4"></a>{{< /raw >}}**练习 1.4** `[*]`\
写出 $\ListOfInt$ 推导 $\texttt{(-7 . (3 . (14 . ())))}$ 的完整步骤 。

> [!note]- 答 :
> 
> $\begin{aligned}[t] &
\ListOfInt                                \\\Rightarrow{}&
\texttt{(\Int  . \ListOfInt)}             \\\Rightarrow{}& 
\texttt{(-7 . \ListOfInt}                 \\\Rightarrow{}& 
\texttt{(-7 . (\Int . \ListOfInt))}       \\\Rightarrow{}&
\texttt{(-7 . (3 . (\Int . \ListOfInt)))} \\\Rightarrow{}&
\texttt{(-7 . (3 . (14 . \ListOfInt)))}   \\\Rightarrow{}&
\texttt{(-7 . (3 . (14 . ())))} 
\end{aligned}$

现在来看一些常用集合的定义 。

1. 不少符号操作过程被设计为处理只包含符号或者是其他具有同样约束条件的列表 。这样的列表叫
   $\SList$ , 定义如下 :

   {{< raw >}}<a id="Def1.1.6"></a>{{< /raw >}}**定义 1.1.6** ( $\SList$ , $\SExp$ )

   $\qquad\begin{aligned}[t]
   \SList ::={}& \texttt{($\{\SExp\}^*$)} \\
   \SExp  ::={}& \Symbol \mid \SList
   \end{aligned}$

   $\SList$ 是串 $\SExp$ 序列 , 而 $\SExp$ 要么是个 $\SList$ 要么则是个符号 。下面是一些 $\SList$ :

   ```scheme {data-open=true}
   (a b c)
   (an (((s-list)) (with () lots ((of) nesting))))
   ```

   我们有时也使用更宽泛的 $\SList$ 定义 , 即除了允许符号外还允许整数 。

2. 对于一个叶节点都是数字且非叶节点都为符号的二叉树 , 其内部节点可以用一个含三个元素的列表
   来表示 , 即

   {{< raw >}}<a id="Def1.1.7"></a>{{< /raw >}}**定义 1.1.7** ( 二叉树 )

   $\qquad \begin{aligned}[t]
   \Bintree ::={}& \Int \mid \texttt{(\Symbol  \Bintree  \Bintree)}
   \end{aligned}$

   下面是一些二叉树的例子 : 

   ```scheme {data-open=true}
   1
   2
   (foo 1 2)
   (bar 1 (foo 1 2))
   (baz (bar 1 (foo 1 2)) (biz 4 5))
   ```

3. **Lambda 演算**是种很简单的语言 , 常用于研究编程语言理论 。这种语言仅含变量引用 , 单参数过程
   以及过程调用三种变体 。我们可通过下述语法对其进行定义 :

   {{< raw >}}<a id="Def1.1.8"></a>{{< /raw >}}**定义 1.1.8** ( Lambda 演算 )

   $\qquad \begin{aligned}[t]
   \LcExp   ::={}& 
   \Iden \\ ::={}&
   \texttt{(lambda (\Iden) \LcExp)} \\::={}&
   \texttt{(\LcExp  \LcExp)}
   \end{aligned}$

   其中 $\Iden$ 表示任何非 $\texttt{lambda}$ 的符号 。

   第二个派生式中的 $\Iden$ 是 $\texttt{lambda}$ 演算表达式**体 body** 内某个变量的名称 。该变量称作约束变量 , 
   因为它绑定 ( 或称捕获 ) 自己在表达式体中的所有出现 , 这些出现都指向其该变量 。

   要弄清楚是什么回事 , 考虑带有算术运算的 $\texttt{lambda}$ 演算表达式 。在此语言中

   ```scheme {data-open=true}
   (lambda (x) (+ x 5))
   ```

   是个表达式且 `x` 是一个约束变量 , 该表达式描述了一个给实参加 $5$ 的过程 。因此在

   ```scheme {data-open=true}
   ((lambda (x) (+ x 5) (- x 7)))
   ```

   中 , `x` 的最后一次出现指向的并非 $\texttt{lambda}$ 演算表达式中的 `x` 。我们将在 [1.2.4 节](#S1.2.4)中详细讨论这一点 , 
   在那里我们将引入 `occurs-free?` 。

   该语法将 $\LcExp$ 中元素定义为 Scheme 的值 , 因此写出操作它们的程序将会很容易 。

上述语法之所以被称作**语境无关的 context-free** 是因为每条规则所定义的语法类别可以在任何提及它的
语境中使用 。而这并不是很具有约束性 。考虑二叉搜索树 , 其节点要么为空 , 要么则包含一个整数以及
两棵子树

{{< raw >}}<a id="Def_BST"></a>{{< /raw >}}$\qquad \begin{aligned}[t]
\BinSearchTree ::={}& \texttt{()} \mid \texttt{(\Int  \BinSearchTree  \BinSearchTree)}
\end{aligned}$

上述语法如实反映了每个节点的结构 , 但忽略了二叉搜索树的一个要求 : 所有左子树键的值都必须小于 ( 
或等于 ) 当前节点 , 而所有右子树的键值都应大于当前节点 。

由于这条额外限制的存在 , 基于 $\BinSearchTree$ 的语法推导未必都是正确的 ; 要想判断一个派生式
可否用于特定的语法推导 , 我们就必须检查派生式被用于哪一种语境之中 。这样的限制叫**语境有关约束
 context-sensitive constraint** , 或称**不变条件 invariant** 。

语境有关约束也会在定义编程语言的语法时出现 , 如不少编程语言中变量在使用前都必须先声明 。这种
对变量使用的约束便对其语境产生有关性 。虽然形式化方法可以定义语境有关限制 , 但这些方法比本章
所考虑的更为复杂 。实际情况往往是 : 我们首先定义语境无关语法 , 然后再通过其他方法添加语境有关
约束 。这些技巧的相关案例将会在第 7 章出现 。


### 1.1.3 数学归纳法 {#S1.1.3}

递归描述完集合后这些递归定义有两种用途 : 要么拿它们去证那些判断元素与集合之间从属关系的定理 , 
要么拿它们编写程序以操作集合元素 。这里先提供一个此类证明的例子 , 编写程序则留作[下一节](#S1.2)的主题 。

**定理 1.1.1** 若 $t$ 为一个二叉树 ( 如[定义 1.1.7](#Def1.1.7) 所述 ) , 则 $t$ 含奇数个节点 。

*证* : 证明是通过对 $t$ 的**尺寸 size** 进行归纳实现的 , 这里 $t$ 的尺寸为 $t$ 所含的节点个数 。归纳假设 $\InductHyp(k)$ 
为 " 任何尺寸 $\leq k$ 的树都只含有奇数个节点 " 。按照归纳法的惯例我们首先证 $\InductHyp(0)$ 成立 , 然后再证 " 
若 $\InductHyp(k)$ 对任何整数 $k$ 都成立则亦有 $\InductHyp(k+1)$ " 。

1. 没有不含节点的树 , 因此 $\InductHyp(0)$ 成立 ; 

2. 假设 $\InductHyp(k)$ 对正整数 $k$ 成立 —— 即任何节点数目 $\leq k$ 的树都只含奇数个节点 , 
   则证 $\InductHyp(k+1)$ 也成立  —— 即任何节点数目 $\leq k+1$ 的树都有奇数个节点 。
   如果 $t$ 拥有的节点数 $\leq k+1$ , 那么根据前面二叉树的定义会有下述两种情况 : 

   1. $t$ 可能形如 $n$ , 这里 $n$ 是整数 —— 此时 $t$ 只有一个节点 , 且 $1$ 是个奇数 ;

   2. $t$ 可能形如 $\texttt{({\rm sym} $t_1$ $t_2$)}$ —— 这里 $\texttt{sym}$ 为符号 , $t_1$ , $t_2$ 为树 , 
      并且它们拥有的节点数肯定比原先的 $t$ 更少 ; 而 $t$ 所含的节点数小于等于 $k+1$ , 故 $t_1$ , $t_2$ 拥有的节点数比 $k$ 少 ; 
      如此它们便在 $\InductHyp(k)$ 的描述范围内 , 它们所拥有的节点数只能是奇数 —— 如分别为 $2n_1+1$ 和 $2n_2+1$ 。
      因此 $t$ 拥有的节点数就应为

      $\qquad (2n_1+1)+(2n_2+1)+1=2(n_1+n_2+1)+1$

      而这同样也是个奇数 。

如此我们便完成了对 $\InductHyp(k+1)$ 的证明 , 也就完成了归纳证明 。

> **结构归纳法的证明步骤**
>
> 要证明性质 $\textrm{\InductHyp}(s)$ 对所有结构 $s$ 都成立 , 请证明
>
> 1. $\InductHyp$ 对所有简单结构 ( 即没有子结构的对象 ) 都成立 ; 
> 2. $\InductHyp$ 若对所有 $s$ 的子结构都成立那么 $\InductHyp$ 对 $s$ 也成立 。

{{< raw >}}<a id="Xer1.5"></a>{{< /raw >}}**练习 1.5** `[**]`\
证明 $e\in \LcExp$ 蕴含 $e$ 中开闭括号数目相等 。

> [!note]- 证 : 对 $\texttt{lambda}$ 演算表达式 $e$ 的高度 $h(e)$ 进行归纳 。
> 
> 对任何 $\texttt{lambda}$ 演算表达式 $e$ , $e_1$ , $e_2$ , 我们规定
> 
>    1. $h(e)=1$ 仅当 $e$ 形如 $\Iden$ ;
>    2. $h(e)=d(e_1)+1$ 仅当 $e$ 形如 $\texttt{(lambda  (\Iden) $e_1$)}$ ;
>    3. $h(e)=\max(d(e_1,e_2))$ 仅当 $e$ 形如 $\texttt{($e_1$  $e_2$)}$ 。
> 
> 如果我们的归纳假设 $IH(k)$ 为 " 任意满足 $h(e)\leq k$ 的 $\texttt{lambda}$ 演算表达式 $e$ 所拥有的开闭括号数相等 " 。那么易知
> 
>    1. $IH(0)$ 必为真 —— 因为高度为 $0$ 的 $\texttt{lambda}$ 演算表达式都形如 $\Iden$ ;
>    2. 假如 $IH(k)$ 对正整数 $k$ 也成立 —— 即任何高度 $\leq k$ 的 $\texttt{lambda}$ 演算表达式所拥有的开闭括号数相等 , 则证 $IH(k+1)$ 也成立 —— 即任何高度 $\leq k+1$ 的 $\texttt{lambda}$ 演算表达式拥有的开闭括号数相等 。如果 $\texttt{lambda}$ 演算表达式 $e$ 的高度 $\leq k+1$ , 那么根据前面 $\texttt{lambda}$ 演算表达式的定义会有下述两种情况 :
>       1. $e$ 可能形如 $\texttt{(lambda  (\Iden) $e_1$)}$ , 这里 $e_1$ 也为 $\texttt{lambda}$ 演算表达式 —— 不难看出此时新增的开闭括号数是持平的 ;
>       2. $e$ 可能形如 $\texttt{($e_1$  $e_2$)}$ , 这里 $e_1$ 和 $e_2$ 也为 $\texttt{lambda}$ 演算表达式 —— 同样这里新增的开闭括号数依然是持平的 。
> 
>    如此我们便完成了对 $IH(k+1)$ 的证明 , 也就完成了归纳证明 。

## 1.2 推导递归程序 {#S1.2}

我们已学会用递归定义构建复杂集合 , 且见识到通过分析这些集合的单个元素便可窥探集合的整体面貌 ; 
我们用这一技巧写出了过程 `in-S?` 以判断自然数是否为集合 $S$ 的成员 。我们将会用同样的思想定义更
通用的过程以便在这些集合上进行计算 。递归定义的过程依赖于下述重要原理 : 

> **较小子问题原则**
>
> 若能将问题归约为更小的子问题 , 则可以调用求解原问题的过程来求解子问题 。

已获得的子问题的解随后可用于求解原问题 。这一切之所以可行是因为每次我们调用过程都是针对一个
更小的子问题 , 直到最后一个可直接求解且无需进一步调用过程自身的子问题为止 。

我们会用一些例子来解释这一思想 。

### 1.2.1 `list-length` {#S1.2.1}

标准的 Scheme 过程 `length` 用于获取列表中元素个数 。

```scheme {data-open=true}
> (length '(a b c))
3
> (length '((x) ()))
2
```

让我们来实现自己的 `length` 并称之为 `list-length` 。

首先写下过程的**合约 contract** 。合约指明了过程所有实参所构成的集合以及所有可能的返回值构成的集 。
合约也可以包含过程应当具有的功能以及行为 。这有助于我们在编写程序及未来调试时追踪我们的意图 。
我们会将其作为一条注释加入代码中以便阅读 。

```scheme {data-open=true}
;; 过程合约 : list-length : List-of-(SchemeValue)   → Natural
;; 过程用途 : (list-length            l         )   = l 的长度
;; 实参语法 :               List-of-(SchemeValue) ::= () | (SchemeValue . List-of-(SchemeValue))
(define list-length
  (lambda (lst)
    ...))
```

我们可以将列表构成的集合定义为

$\qquad \List ::= \texttt{()}\mid  \texttt{(\ScmVal  . \List)}$

现在来考虑列表的各种情况 : 如果列表为空则长度为 $0$ 。

```scheme {hl_lines=["5-6"], data-open=true}
;; 过程合约 : list-length : List-of-(SchemeValue)   → Natural
;; 过程用途 : (list-length            l         )   = l 的长度
;; 实参语法 :               List-of-(SchemeValue) ::= () | (SchemeValue . List-of-(SchemeValue))
(define list-length
  (lambda (lst)
    (if (null? lst) 0                                 
        ...)))
```

如果列表非空则其长度比其 `cdr` 长度多 $1$ 。这便推导出了完整的代码 。

```scheme {hl_lines=[6], data-open=true}
;; 过程合约 : list-length : List-of-(SchemeValue)   → Natural
;; 过程用途 : (list-length            l         )   = l 的长度
;; 实参语法 :               List-of-(SchemeValue) ::= () | (SchemeValue . List-of-(SchemeValue))
(define list-length
  (lambda (lst)
    (if (null? lst) 0
        (+ 1 (list-length (cdr lst)))))) 
```

我们可以根据 `list-length` 的定义窥探其计算的原理 。

```scheme {data-open=true}
(list-length '(a (b c) d))          = 
(+ 1 (list-length '((b c) d)))      = 
(+ 1 (+ 1 (list-length '(d))))      = 
(+ 1 (+ 1 (+ 1 (list-length '())))) = 
(+ 1 (+ 1 (+ 1 0)))                 = 
3
```

### 1.2.2 `nth-element` {#S1.2.2}

标准的 Scheme 过程 `list-ref` 以一个列表 `lst` 及一个从 $0$ 开始计数的索引 `n` 为实参并返回 `lst` 的第 
`n` 个元素 。

```scheme {data-open=true}
> (list-ref '(a b c) l) 
b
```

让我们写出自己的 `list-ref` 并称之为 `nth-element` 。同样沿用前面 $\List$ 的定义 。

$\texttt{(nth-element $lst$ $n$)}$ 在实参 $lst$ 为空时应返回什么结果呢 ?
此时 $\texttt{(nth-element $lst$ $n$)}$ 想找出
空列表的元素 , 因此应给出报错 。

$\texttt{(nth-element $lst$ $n$)}$ 在实参 $lst$ 非空时又应该返回什么结果呢 ?
此时的返回值将取决于 $n$ : 若 $n = 0$ 
则返回值应为 $lst$ 的 `car` 。

$\texttt{(nth-element $lst$ $n$)}$ 在实参 $lst$ 非空且 $n\neq 0$ 时又应返回什么呢 ?  此时返回值将会是 $lst$ 的 `cdr` 的
第 $n-1$ 个元素 。由 $n\in \N$ 且 $n\neq 0$ 可知 $n-1$ 一定是属于 $\N$ , 因此可递归地调用 `nth-element` 来
找出第 $n-1$ 个元素 。如此便推导出了完整的代码 : 

```scheme {data-open=true}
;; 过程合约 : nth-element : List-of-(SchemeValue) × Natural   → SchemeValue
;; 过程用途 : (nth-element            lst              n  )   = lst 的第 n 个元素
;; 实参语法 :               List-of-(SchemeValue)           ::= () | (SchemeValue . List-of-(SchemeValue))
;; 实参语法 :                                       Natural ::=  0 | (succ Natural)
(define nth-element
  (lambda (lst n)
    (if (null? lst) (report-list-too-short n)
        (if (zero? n) (car lst)
            (nth-element (cdr lst) (- n 1))))))

(define report-list-too-short
  (lambda (n) 
    (eopl:error 'nth-element
                "List too short by ~s elements.~%" (+ n 1))))
```

上面的注释 `nth-element : List × Int → SchemeVal`  表示过程  `nth-element`  取两个参数 ( 一个列表和
一整数 ) 并返回一个 Scheme 值 。这与在数学中写下公式 $f : A \times B \to C$ 时所使用的记法有类似之处 。

`report-list-too-short` 通过调用 `eopl:error` 实现报错 , 后者会终止计算 。该过程第一个参数是个符号 , 
用于在错误信息中指出调用 `eopl:error` 的过程 ; 而第二个参数则是一个字符串 , 即为待打印的错误信息 ;  
对于该字符串中的每个字符序列 `~s` 都必须提供一个额外实参 , 实参的值将用于替换对应的 `~s` ;  而 `~%` 
则会被视作换行 。错误信息打印完后计算将会被停止 。过程 `eopl:error` 并不是标准 Scheme 的一部分 , 
但大多数 Scheme 的实现都有提供类似的功能 ; 本书将始终采用名称带前缀 `report-` 的过程来实现类似
风格的报错 。现在来看看 `nth-element` 如何计算结果 : 

```scheme {data-open=true}
(nth-element '(a b c d e) 3) = 
(nth-element   '(b c d e) 2) = 
(nth-element     '(c d e) 1) = 
(nth-element       '(d e) 0) = 
d
```

这里 `nth-element` 递归处理的列表和数字都在变得越来越小 。

若错误检查被省略 , 那么就得依赖 `car` 和 `cdr` 的报错来知悉空列表被作为实参传入 ; 但它们的错误信息
提供不了太多信息 。假如报错是从 `car` 发出的 , 那可能得找遍程序中所有用到 `car` 的地方才能定位错误 。

{{< raw >}}<a id="Xer1.6"></a>{{< /raw >}}**练习 1.6** `[*]`\
若是调换 `nth-element` 中条件语句里的测试顺序 , 则会有什么样的问题呢 ?

> [!note]- 答 :
>
> `car` 可能会作用于 `'()` 。

{{< raw >}}<a id="Xer1.7"></a>{{< /raw >}}**练习 1.7** `[**]`\
`nth-element` 的错误信息仍不够详尽 , 请重写 `nth-element` 并给出更详细的错误信息 ——
如 " `(a b c)` 拥有的元素数目不到 8 个 " 。

> [!note]- 答 : 
> 
> `report-list-too-short` 修改后结果如下 :
> 
> ```scheme {data-open=true}
> (define report-list-too-short
>   (lambda (n) 
>     (eopl:error 'nth-element
>                 "List too short by ~s elements.~%" (+ n 1))))
> ```

### 1.2.3 `remove-first` {#S1.2.3}

过程 `remove-first` 取两个参数 : 一个符号 $s$ 及一个符号列表 $los$ 。其应返回一个列表 , 该列表除了不含 
$s$ 在 $los$ 中的首个出现外其余元素及排列顺序都应与 $los$ 相同 。若 $s$ 在 $los$ 中没有出现则原样返回 $los$ 。

```scheme {data-open=true}
> (remove-first 'a '(a b c))
(b c)
> (remove-first 'b '(e f g))
(e f g)
> (remove-first 'a4 '(c1 a4 c1 a4))
(c1 c1 a4)
> (remove-first 'x '())
()
```

在写下此过程的完整定义前 , 我们必须通过定义符号列表所构成的集合 $\ListOfSymbol$ 以给出问题的
完整描述 。与上一节介绍的 $\SList$ 所不同的是 , 符号列表并不包含子列表 。

$\qquad \ListOfSymbol ::= \texttt{()} \mid \texttt{(\Symbol  . \ListOfSymbol)}$

符号列表要么是空列表 , 要么则满足 `car` 是单个符号且 `cdr` 也是一个符号列表 。

如果列表为空 , 则没有待移除的 $s$ 的出现 , 因此返回值为空列表 。

```scheme {data-open=true}
;; 过程合约 : remove-first : Symbol × List-of-(Symbol)   → List-of-(Symbol)
;; 过程用途 : (remove-first     s          los       )   = 删去 s 在 los 中的首次出现
;; 实参语法 :                         List-of-(Symbol) ::= () | (Symbol . List-of-(Symbol))
(define remove-first
  (lambda (s los)
    (if (null? los) '()
        ...)))
```

在写合约时我们用的是 $\ListOf{\Symbol}$ 而非 $\ListOfSymbol$ ; 之所以采用这样的记法是因为其可以
免除许多类似前面提到的定义 。

如果 $los$ 非空 , 那么有没有哪种情况可允许我们立刻得出结果呢 ? 假如 $los$ 的首个元素是 $s$ —— 如 $los $
$= \texttt{($s$ $s_1$ $...$ $s_{n-1}$)}$ , 那么 $s$ 的首个出现便是 $los$ 的 `car` ; 因此将其剔除后结果应该为 $\texttt{($s_1$ $...$ $s_{n-1}$)}$ 。

```scheme {hl_lines=["6-7"], data-open=true}
;; 过程合约 : remove-first : Symbol × List-of-(Symbol)   → List-of-(Symbol)
;; 过程用途 : (remove-first     s          los       )   = 删去 s 在 los 中的首次出现
;; 实参语法 :                         List-of-(Symbol) ::= () | (Symbol . List-of-(Symbol))
(define remove-first
  (lambda (s los)
    (if (null? los) '()
        (if (eqv? (car los) s) (cdr los)          
            ...))))            
```

如果 $los$ 的首个元素不是 $s$ —— 比如 $los = \texttt{($s_0$ $s_1$ $...$ $s_{n-1}$)}$ , 那么便可得知 $s_0$ 不是第一个出现的 $s$ , 
因此返回值的第一个元素一定会是 $s_0$ —— 也就是 `(car los)` 的值 。如此 $los$ 里 $s$ 的首个出现肯定会落在
$\texttt{($s_1$ $...$ $s_{n-1}$)}$ 中 , 说明返回值的 `cdr` 肯定是通过移除 $los$ 的 `cdr` 的首个 $s$ 的出现获得的 ; 而 $los$ 的 `cdr` 
肯定比 $los$ 短 ,  因此可通过递归调用 `remove-first` 从 $los$ 的 `cdr` 中移除 $s$ 的出现 , 即返回值的 `cdr` 可由 `(remove-first s (cdr los))` 求得 。既然已经知道如何找出返回值的 `car` 和 `cdr` , 便可用 `cons` 结合二者 , 
通过代码 `(cons (car los) (remove-first s (cdr los)))` 即得返回值 。故 `remove-first` 的完整定义应为

```scheme {hl_lines=[7], data-open=true}
;; 过程合约 : remove-first : Symbol × List-of-(Symbol)   → List-of-(Symbol)
;; 过程用途 : (remove-first     s          los       )   = 删去 s 在 los 中的首次出现
;; 实参语法 :                         List-of-(Symbol) ::= () | (Symbol . List-of-(Symbol))
(define remove-first
  (lambda (s los)
    (if (null? los) '()
        (if (eqv? (car los) s) (cdr los)           
            (cons (car los) (remove-first s (cdr los))))))) 
```

{{< raw >}}<a id="Xer1.8"></a>{{< /raw >}}**练习 1.8** `[*]`\
若是将 `remove-first` 定义的最后一行改为 `(remove-first s (cdr los))` , 那么得到的过程将进行什么样的运算呢 ? 请为修改后的过程给出合约以及其用法 。

> [!note]- 答 : 可以将新的过程称作 `drop-until` 。
> 
> ```scheme {data-open=true}
> ;; 过程合约 : drop-until : Symbol × List-of-(Symbol)   → List-of-(Symbol)
> ;; 过程用途 : (drop-until     s         los        )   = 删去 los 中 s 的首次出现之前的所有元素
> ;; 实参语法 :                       List-of-(Symbol) ::= () | (Symbol . List-of-(Symbol))
> (define drop-until
>   (lambda (s los)
>     (if (null? los) '()
>         (if (eqv? (car los) s) (cdr los)           
>             (cons (car los) (drop-until s (cdr los))))))) 
> ```

{{< raw >}}<a id="Xer1.9"></a>{{< /raw >}}**练习 1.9** `[**]`\
定义 `remove` : 它类似 `remove-first` , 但会从符号列表中移除给定符号的所有出现而不仅仅是第一个 。

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : remove : Symbol × List-of-(Symbol)   → List-of-(Symbol)
> ;; 过程用途 : (remove    s            los      )   = 删去 los 中 s 的所有出现
> ;; 实参语法 :                   List-of-(Symbol) ::= () | (Symbol . List-of-(Symbol))
> (define (remove s los)
>   (if (null? los) '()
>       (if (eqv? (car los) s) (remove s (cdr los)) ;; 此即与 remove-first 所不同之处
>           (cons (car los) (remove s (cdr los))))))
> ```

### 1.2.4 `occurs-free?` {#S1.2.4}

过程 `occurs-free?` 取一个由 Scheme 符号表示的变量 $var$ 以及一个如[定义 1.1.8](#Def1.1.8) 所述的 $\texttt{lambda}$ 演算表达式 
$exp$ 并判断 $var$ 在 $exp$ 中是否存在自由出现 。我们称变量在 $exp$ 中**自由出现 occurs free** 当且仅当其于 
$exp$ 中存在不位于与之同名的变量的所有 $\texttt{lambda}$ 绑定下的出现 。如 : 

```scheme {data-open=true}
> (occurs-free? 'x 'x)                                   
#t
> (occurs-free? 'x 'y)                                   
#f
> (occurs-free? 'x '(lambda (x) (x y)))                  
#f
> (occurs-free? 'x '(lambda (y) (x y)))                  
#t
> (occurs-free? 'x '((lambda (x) x) (x y)))              
#t
> (occurs-free? 'x '(lambda (y) (lambda (z) (x (y z))))) 
#t
```

我们可以遵照 lambda 演算表达式的语法来解决此问题：

$\qquad \begin{aligned}[t]
\LcExp   ::={}& 
\Iden \\ ::={}&
\texttt{(lambda (\Iden) \LcExp)} \\::={}&
\texttt{(\LcExp  \LcExp)}
\end{aligned}$

我们可以总结出规则里的各种情况：

- 若表达式 $e$ 是变量 , 则变量 $x$ 在 $e$ 中有自由出现
  当且仅当 $x$ 与 $e$ 完全一致 。
- 若表达式 $e$ 形如 $\texttt{(lambda ($y$) $e'$)}$ , 则变量 $x$ 在 $e$ 中有自由出现
  当且仅当 $y$ 不同于 $x$ 且 $x$ 在 $e'$ 中有自由出现 。
- 若表达式 $e$ 形如 $\texttt{($e_1$ $e_2$)}$ , 则变量 $x$ 在 $e$ 中有自由出现
  当且仅当 $x$ 在 $e_1$ 或 $e_2$ 中都有自由出现 。这里的 ” 或 " 意为**兼有或 inclusive or** , 
  表示其也涵盖了 $x$ 在 $e_1$ 和 $e_2$ 中同时有自由出现的情况 。我们通常用 " 或 " 来表示这个意思 。

你应确信上述规则精准捕获了 " $x$ 的出现不在 $x$ 的所有 $\texttt{lambda}$ 绑定之下 " 所描述的概念 。

{{< raw >}}<a id="Xer1.10"></a>{{< /raw >}}**练习 1.10** `[*]`\
我们常用 " 或 " 表示 " 逻辑或 " 。" 或 " 还有什么含义？

> [!note]- 答 :
>
> 异或 XOR , 同英文里的 " either ... or ... " 。

后面定义 `occurs-free?` 就很容易了 。由于有三种情况要检查 , 故这里用的是 Scheme 的 `cond` 而非 `if` 。
在 Scheme 中 $\texttt{(or $exp_1$ $exp_2$)}$ 会在 $exp_1$ 和 $exp_2$ 里有一个为真的情况下返回真 。

```scheme {data-open=true}
;; 过程合约 : occurs-free? : Symbol × LcExp   → Bool
;; 过程用途 : (occurs-free?   var      exp)   = 布尔值 , 若符号 var 在 exp 中自由出现则返回 #t 否则返回 #f
;; 实参语法 :                         LcExp ::= Symbol | (lambda (Symbol) LcExp) | (LcExp LcExp)
(define occurs-free?
  (lambda (var exp)
    (cond ((symbol? exp) (eqv? var exp))
          ((eqv? (car exp) 'lambda) (and (not (eqv? var (car (cadr exp))))
                                         (occurs-free? var (caddr exp))))
          (else (or (occurs-free? var (car exp))
                    (occurs-free? var (cadr exp)))))))
```

这一过程略显晦涩 : 如我们很难猜到 `(car (cadr exp))` 指的是 $\texttt{lambda}$ 演算表达式中的变量声明 ; 又或者是 `(caddr exp)` 指的是 $\texttt{lambda}$ 演算表达式的主体 。在[第 2.5 节](/posts/1568c34/#S2.5)里我们将会展示如何可以显著地改善这种状况 。

### 1.2.5 `subst` {#S1.2.5}

过程 `subst` 取三个参数 : 两个符号 `new` 和 `old` 及一个 $\SList$ `slist` , 它会检查 `slist` 的所有元素并返回
一个类似 `slist` 的新列表 —— 只不过新列表中所有的 `old` 都会被替换为 `new` 。

```scheme {data-open=true}
> (subst 'a 'b '((b c) (b () d)))
((a c) (a () d))
```

由于 `subst` 是基于 $\SList$ 定义的 , 因此其结构应体现出 $\SList$ 的定义 ( 见[定义 1.1.6](#Def1.1.6) )

$\qquad\begin{aligned}[t]
\SList ::={}& \texttt{(\(\{\SExp\}^*\))} \\
\SExp  ::={}& \Symbol \mid  \SList
\end{aligned}$

克莱尼星号给出了集合 $\SList$ 的精简表示 , 但这对写程序没什么用 ; 因此我们的第一步是删去克莱尼星号
并重写语法 。重写后的语法指明我们的过程应该递归处理 $\SList$ 的首项和余项 。

$\qquad\begin{aligned}[t]
\SList ::={}& \texttt{()} %\\
       %::={}& 
       \mid  \texttt{(\SExp  . \SList)} \\
\SExp  ::={}& \Symbol \mid  \SList
\end{aligned}$

这个例子比之前的更为复杂 —— 因为过程的其中一个输入对应的语法含有两个非终结符 $\SList$ 和 $\SExp$ 。
因此我们需要两个过程 , 一个用于处理 $\SList$ 而另一个则用于 $\SExp$ 。

```scheme {data-open=true}
;; 过程合约 : subst : Symbol × Symbol × S-list   → S-list
;; 过程用途 : (subst   new      old     slist)   = 将 slist 中所有 old 替换为 new
;; 实参语法 :                           S-list ::= () | (S-exp . S-list)
(define subst 
  (lambda (new old slist)
    ...))

;; 过程合约 : subst-in-s-exp : Symbol × Symbol × S-exp   → S-exp
;; 过程用途 : (subst-in-s-exp   new      old     sexp)   = 将 sexp 中所有 old 替换为 new
;; 实参语法 :                                    S-exp ::= Symbol | S-list
(define subst-in-s-exp 
  (lambda (new old sexp)
    ...))
```

我们首先处理 `subst` 。如果列表为空则无需替换 `old` 。

```scheme {data-open=true}
;; 过程合约 : subst : Symbol × Symbol × S-list   → S-list
;; 过程用途 : (subst   new      old     slist)   = 将 slist 中所有 old 替换为 new
;; 实参语法 :                           S-list ::= () | (S-exp . S-list)
(define subst 
  (lambda (new old slist)
    (if (null? slist) '()
    	...)))
```

如果 `slist` 非空 , 那么它的 `car` 是一个 $\SExp$ 而 `cdr` 是另一 $\SList$ ; 这时返回值应是一个列表 , 它的 `car` 应
等于 `slist` 的 `car` , 只不过其中的 `old` 全部被替换为 `new` ; 而它的 `cdr` 则应该等于 `slist` 的 `cdr` , 只不过其中的 `old` 全部都被替换为 `new` 。由于 `slist` 的 `car` 是 $\SExp$ 的元素 , 因此考虑用 `subst-in-s-exp` 解决
该子问题 ; 由于 `slist` 的 `cdr` 是 $\SList$ 的元素 , 因此考虑递归调用 `subst` 处理它 : 

```scheme {data-open=true, hl_lines=["4-6"]}
;; 过程合约 : subst : Symbol × Symbol × S-list   → S-list
;; 过程用途 : (subst   new      old     slist)   = 将 slist 中所有 old 替换为 new
;; 实参语法 :                           S-list ::= () | (S-exp . S-list)
(define subst 
  (lambda (new old slist)
    (if (null? slist) '()
        (cons (subst-in-s-exp new old (car slist)) 
              (subst new old (cdr slist))))))      
```

现在处理 `subst-in-s-exp` 。
由语法可知符号表达式 `sexp` 要么是符号要么则是 $\SList$ 。
如果是符号则检查它与符号 `old` 是否一致 :
若一致则返回值为 `new` ,
否则仍为 `sexp` ;
若 `sexp` 是 $\SList$ 则递归调用`subst` 求解 。

```scheme {data-open=true, hl_lines=["4-6"]}
;; 过程合约 : subst-in-s-exp : Symbol × Symbol × S-exp   → S-exp
;; 过程用途 : (subst-in-s-exp   new      old     sexp)   = 将 sexp 中所有 old 替换为 new
;; 实参语法 :                                    S-exp ::= Symbol | S-list
(define subst-in-s-exp 
  (lambda (new old sexp)
    (if (symbol? sexp) (if (eqv? sexp old) new 
                                           sexp)  
        (subst new old sexp))))                  
```

由于该过程是严格按照 $\SList$ 和 $\SExp$ 的定义来编写的 , 因此这个递归最终一定会停止 。其中 `subst` 和 `subst-in-s-exp` 递归调用彼此 , 因此便称之为**互为递归的 mutually recursive** 。

把 `subst` 拆解为两个过程分别处理一种语法类别是一种重要的技巧 。
该技巧允许我们每次只专注于一种语法类别 , 从而极大地简化我们对于复杂程序的思考 。

{{< raw >}}<a id="Xer1.11"></a>{{< /raw >}}**练习 1.11** `[*]`\
`subst-in-s-exp` 的最后一行中递归是针对 `sexp` 而非更小的子结构 , 为什么它照样可以停机 ?

> [!note]- 答 :
> 
> 因为实参 `subst` 和 `subst-in-s-exp` 的长度会不断变小 。

{{< raw >}}<a id="Xer1.12"></a>{{< /raw >}}**练习 1.12** `[*]`\
用 `subst-in-s-exp` 的定义替换 `subst` 中对自己的调用以将这次调用排除 ,
然后再简化最终的过程 。在最后的结果中 `subst` 应当不需要 `subst-in-s-exp` 。
这种技巧叫**内联 inlining** , 用于优化编译器 。

> [!note]- 答 :
>
> 
> ```scheme {data-open=true}
> ;; 过程合约 : subst : Symbol × Symbol × S-list   → S-list
> ;; 过程用途 : (subst   new      old     slist)   = 将 slist 中所有 old 替换为 new
> ;; 实参语法 :                           S-list ::= () | (S-exp . S-list)
> (define subst
>   (lambda (new old slist)
>     (if (null? slist) '()
>         (cons (let ([sexp (car slist)])
>                 (if (symbol? sexp) (if (eqv? sexp old) new sexp)
>                                        (subst new old sexp)))
>                                    (subst new old (cdr slist))))))
> ```

{{< raw >}}<a id="Xer1.13"></a>{{< /raw >}}**练习 1.13** `[**]`\
前面的例子中我们先从排除 $\SList$ 语法内的克莱尼星号开始 。现请依照原本的语法用 `map` 重写 `subst` 。

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : subst : Symbol × Symbol × S-list   → S-list
> ;; 过程用途 : (subst   new      old     slist)   = 将 slist 中所有 old 替换为 new
> ;; 实参语法 :                           S-list ::= () | (S-exp . S-list)
> (define (subst new old slist)
>   (map (lambda (sexp) ;; 借用练习 1.16 的 map
>          (subst-in-s-exp new old sexp))
>        slist))
> ```

现在我们已拥有编写过程处理归纳数据集的套路 , 让我们把它总结成一句口诀 。

> **遵循语法 !**
>
> 若要基于递归定义的数据集编写过程 , 则过程的结构应与数据集的结构保持一致 。

更准确地说：

- 为语法中的每个非终结符都编写一个过程 。该过程只负责处理相应非终结符的数据 。
- 在每个过程中为相应非终结符的每一派生式写一分支 。你可能需要创建更多的分支 , 
  但只有这样才能起步 。然后对派生式右侧出现的每个非终结符递归地调用对应过程 。

## 1.3 辅助过程和语境参数 {#S1.3}

前面的**遵循语法**窍口诀虽然很强大 , 但还是有失效的时候 。现在来考虑过程 `number-elements` : 任何形如 
$\texttt{($v_0$ $v_1$ $v_2$ ...)}$ 的列表都会被其变成 $\texttt{((0 $v_0$) (1 $v_1$) (2 $v_2$) ...)}$ 。

前面那种直接分解的方法在这里并不奏效 , 因为没有明显的方法可使我们从 `(number-elements (cdr lst))` 得出 `(number-elements lst)` ( 但是 , 请看[练习 1.36](#Xer1.36) ) 。

要解决这个问题我们需要考虑更一般的情况 。现在来写一个叫 `number-elements-from` 的新过程 : 相较于
原先的过程 , 它取一个额外实参 $n$ 以指定编号的起始数字 。该过程可以很轻松地写出 。

```scheme {data-open=true}
;; 过程合约 : number-elements-from : List-of-(SchemeValue) × Natural   → List-of-((Natural SchemeValue))
;; 过程用途 : (number-elements-from   '(v0 v1 v2 ...)          n   )   = ((n v0) (n+1 v1) (n+2 v2) ...)
;; 实参语法 :                        List-of-(SchemeValue)           ::= () | (SchemeValue . List-of-(SchemeValue))
;; 实参语法 :                                                Natural ::=  0 | (succ Natural)
(define number-elements-from 
  (lambda (lst n)
    (if (null? lst) '()
        (cons (list n (car lst))
              (number-elements-from (cdr lst) (+ n 1))))))
```

这里合约告诉我们该过程取一个列表 ( 包含任意 Scheme 值 )
及一个整数作为实参并返回一个列表 , 列表的各个元素都是包含两个元素的列表 :
一个是整数 , 而另一个则是 Scheme 值 。

一旦定义好了 `number-elements-from` 便可很轻松地写出所需的过程 。

```scheme {data-open=true} 
;; 过程合约 : number-elements : List-of-(SchemeValue)   → List-of-((Natural SchemeValue))
;; 过程用途 : (number-elements     '(v0 v1 v2 ...))     = ((0 v0) (1 v1) (2 v2) ...)
;; 实参语法 :                   List-of-(SchemeValue) ::= () | (SchemeValue . List-of-(SchemeValue))
(define number-elements 
  (lambda (lst)
    (number-elements-from lst 0)))
```

有两个点需要注意 : 首先 `number-elements-from` 的定义独立于 `number-elements` 。程序员往往需要写一个过程来调用某个辅助过程并固定某个额外的常量作为实参 。除非我们理解辅助过程会对每个实参对应的值施加什么效果 , 否则我们很难理解调用它究竟可以做什么 。这便给了我们一条口诀 : 

> **没有神秘的辅助过程 ！**
>
> 定义辅助过程时请指明它对任何可能的实参所进行的处理 , 而不仅仅是初始值 。

其次 `number-elements-from` 的两个参数各自扮演着不同的角色 :
第一个参数是待处理的列表 , 其随着每次递归调用不断减小 ;
而第二个参数则是对我们当前任务的**语境 context** 的抽象 。本例中 ,
调用 `number-elements` 最终都会变成调用 `number-elements-from` 处理原列表的每个子列表 。第二个参数告知我们子列表在原列表中的位置 , 随着递归调用它并不会递减反而是逐渐增加 —— 因为我们每次都会经过原列表的一个元素 。该参数我们称作为**语境参数 context argument** 或**继承属性 inherited attribute** 。

另一个案例是将向量的各个分量相加 ( 简称对向量求和 ) 。

若要算列表中各个元素的和则可遵循语法递归处理列表的 `cdr` 。如此我们的过程就应为

```scheme {data-open=true}
;; 过程合约 : list-sum : List-of-(Int)   → Int
;; 过程用途 : (list-sum      loi     )   = loi 里所有整数的和
;; 实参语法 :            List-of-(Int) ::= () | (Int . List-of-(Int))
(define list-sum 
  (lambda (loi)
    (if (null? loi) 0
        (+ (car loi)
           (list-sum (cdr loi)))))) 
```

但是我们无法按照这种方式处理向量 , 因为它们不能直接被拆解 。

由于我们无法拆解向量 , 因此现在考虑更一般的问题 : 为向量的一部分求和 。该问题的描述是计算

$\qquad\quad \displaystyle\sum_\mathclap{i=0}^\mathclap{i=length(v)-1} v_i\quad$ 

其中 $v$ 是整数构成的向量 。我们可通过将上界修改为一个形参 $n$ 来将问题一般化 , 如此新的任务便是求

$\qquad\quad \displaystyle\sum_{i=0}^{i=n} v_i$

其中 $0 \leq n < length(v)$ 。对应的过程可以很轻松地写出 , 通过对第二个实参 $n$ 使用归纳法即可完成 。

```scheme {data-open=true}
;; 过程合约 :  partial-vector-sum : Vector-of-(Int) × Natural → Int
;; 过程用途 : (partial-vector-sum          v             n  ) = v0 到 vn 的和 , 若 0 <= n < length(v) 
;; 实参语法 : ?
(define partial-vector-sum 
  (lambda (v n)
    (if (zero? n) (vector-ref v 0)
        (+ (vector-ref v n)
           (partial-vector-sum v (- n 1))))))
```

由于 $n$ 会一直递减到 $0$ , 因此对此程序正确性的证明可以通过对 $n$
使用归纳法来完成 。由 $0\leq n$ 且 $n\neq 0$ 知 $0\leq n-1$ ,
故递归调用 `partial-vector-sum` 仍可满足其合约 。

现在要解决原问题就简单得多了 。
过程 `partial-vector-sum` 无法应用于长度为 $0$ 的向量 ,
因此我们还得单独处理上述情况 。

```scheme {data-open=true}
;; 过程合约 : vector-sum : Vector-of-(Int)   → Int
;; 过程用途 : (vector-sum         v      )   = 对向量 v 进行求和
;; 实参语法 : ?
(define vector-sum 
  (lambda (v)
    (let ((n (vector-length v)))
      (if (zero? n) 0
          (partial-vector-sum v (- n 1))))))
```

还有很多情况可通过引入辅助变量来解决或改善 。
只要能为新过程给出独立的定义 , 就请尽情地尝试吧 ! 

{{< raw >}}<a id="Xer1.14"></a>{{< /raw >}}**练习 1.14** `[*]`\
基于 $0 \leq n < length(v)$ 证明 `partial-vector-sum` 的正确性 。

> [!note]- 证 : 对向量 $v$ 的长度 $length(v)$ 进行归纳 。这里不再赘述 。


## 1.4 习题

想获得编写递归程序的诀窍离不开大量的练习 。那就让我们以一系列习题作为本章的结束把 。

在下面的每道练习题中 : `s` 为符号 , `n` 为非负整数 , `lst` 为列表 , `loi` 为整数列表 , `slist` 为 $\SList$ , `x` 为任意 $\ScmVal$ ; 类似地 `s1` 为符号 , `los2` 为符号列表 , `x1` 为 $\ScmVal$ , 等等 。另外假定 `pred` 是个谓词 —— 即仅返回 `#t` 或 `#f` 的过程 。请勿对数据做出额外假设 , 除非问题另有额外限制 。在这些习题中您不需要检查输入是否符合描述 ; 对每个过程我们都假定其输入值是指定集合的成员 。

请定义 , 测试并调试每个过程 。你的定义应包含如前面所述的合约和用途注释 。您可以随便定义辅助过程 , 但请为每个辅助过程补充说明 , 正如 [1.3 节](#S1.3) 那样 。

在测试这些程序时 , 请先尝试给出所有已提供的例子 , 然后再用其他例子测试 ; 因为书本提供的例子不足以涵盖所有可能的错误 。

{{< raw >}}<a id="Xer1.15"></a>{{< /raw >}}**练习 1.15** `[*]`\
`(duple n x)` 返回一个包含 `n` 个 `x` 的列表 。

```scheme {data-open=true}
> (duple 2 3)
(3 3)
> (duple 4 '(ha ha))
((ha ha) (ha ha) (ha ha) (ha ha))
> (duple 0 '(blah))
()
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : duple : Natural × S-exp   → List-of-(S-exp)
> ;; 过程用途 : (duple     n      sexp)   = 将 sexp 重复 n 遍
> ;; 实参语法 :         Natural         ::= 0 | (succ Natural)
> (define (duple n sexp)
>   (if (zero? n) '()
>       (cons sexp (duple (- n 1) sexp))))
> ```

{{< raw >}}<a id="Xer1.16"></a>{{< /raw >}}**练习 1.16** `[*]`\
`(invert lst)` 会将 `lst` 里面每个 $\textrm{2-List}$ 的顺序颠倒 。
其中 `lst` 是由 $\textrm{2-List}$ ( 长度为 $2$ 的列表 ) 构成的列表 。

```scheme {data-open=true}
> (invert '((a 1) (a 2) (1 b) (2 b)))
((1 a) (2 a) (b 1) (b 2))
```

> [!note]- 答 :
>
> 
> ```scheme (data-open=true)
> ;; 过程合约 : invert : List-of-(2-list)   → List-of-(2-list)
> ;; 过程用途 : (invert        lst      )   = 将 lst 中的每个元素颠倒顺序
> ;; 实参语法 :          List-of-(2-list) ::= ({2-list}*)
> (define (invert lst)
>   (map (lambda (pair) 
>          (list (cadr pair) (car pair)))
>        lst))
> 
> ;; 过程合约 : map :  (A → B) × List-of-(A)   → List-of-(B)
> ;; 过程用途 : (map    proc         lst   )   = 返回 proc 在 lst 上的像
> ;; 实参语法 :                  List-of-(A) ::= () | (A . List-of-(A))
> (define (map proc lst)
>   (if (null? lst)
>       '()
>       (cons (proc (car lst))
>             (map proc (cdr lst)))))
> ```

{{< raw >}}<a id="Xer1.17"></a>{{< /raw >}}**练习 1.17** `[*]`\
`(down lst)` 会为 `lst` 中的元素再加一层括号 。

```scheme {data-open=true}
> (down '(1 2 3))
((1) (2) (3))
> (down '((a) (fine) (idea)))
(((a)) ((fine)) ((idea)))
> (down '(a (more (complicated)) object))
((a) ((more (complicated))) (object))
```

> [!note]- 答 :
> 
> 
> ```scheme {data-open=true}
> ;; 过程合约 : down : List-of-(SchemeValue)   → List-of-(List-of-(SchemeValue))
> ;; 过程用途 : (down          lst         )   = 为 lst 中的元素再加一层括号
> ;; 实参语法 :        List-of-(SchemeValue) ::= ({SchemeValue}*)
> (define (down lst)
>   (map list lst)) ;; 借用练习 1.16 的 map
> ```

{{< raw >}}<a id="Xer1.18"></a>{{< /raw >}}**练习 1.18** `[*]`\
`(swapper s1 s2 slist)` 会将 `slist` 中 `s1` 和 `s2` 的所有出现都分别换成 `s2` 和 `s1` 。

```scheme {data-open=true}
> (swapper 'a 'd '(a b c d))
(d b c a)
> (swapper 'a 'd '(a d () c d))
(d a () c a)
> (swapper 'x 'y '((x) y (z (x))))
((y) x (z (y)))
```

> [!note]- 答 :
>
> 
> ```scheme {data-open=true}
> ;; 过程合约 : swapper : Symbol × Symbol × S-list   → S-list
> ;; 过程用途 : (swapper    s1       s2     slist)   = 将 slist 中所有 s1 和 s2 的出现对调
> ;; 实参语法 :                             S-list ::= ({S-exp}*)
> (define (swapper s1 s2 slist)
>   (map (lambda (sexp) ;; 借用练习 1.16 的 map
>          (swapper-in-s-sexp s1 s2 sexp))
>        slist))
> 
> ;; 过程合约 : swapper-in-s-exp : Symbol  ×  Symbol  ×  S-exp   → S-exp
> ;; 过程用途 : (swapper-in-s-exp    s1         s2       sexp)   = 将 sexp 中所有 s1 和 s2 的出现对调
> ;; 实参语法 :                                          S-exp ::= Symbol | S-list
> (define (swapper-in-s-sexp s1 s2 sexp)
>   (cond
>     [(symbol? sexp) (cond
>                       [(eqv? s1 sexp) s2]
>                       [(eqv? s2 sexp) s1]
>                       [else sexp])]
>     [else (swapper s1 s2 sexp)]))
> ```

{{< raw >}}<a id="Xer1.19"></a>{{< /raw >}}**练习 1.19** `[**]`\
`(list-set lst n x)` 会将 `lst` 中的第 `n` 个元素 ( 从 `0` 算起 ) 设置为 `x` 。

```scheme {data-open=true}
> (list-set '(a b c d) 2 '(1 2))
(a b (1 2) d)
> (list-ref (list-set '(a b c d) 3 '(1 5 10)) 3)
(1 5 10)
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : list-set : List-of-(SchemeValue) × Natural × SchemeValue   → List-of-(SchemeValue)
> ;; 过程用途 : (list-set           lst               n          x     )   = 将 lst 中的第 n 个元素设置为 x
> ;; 实参语法 :                                    Natural               ::=  0 | (succ Natural)
> (define (list-set lst n x)
>   (if (zero? n) (cons x (cdr lst))
>       (cons (car lst)
>             (list-set (cdr lst) (- n 1) x))))
> ```

{{< raw >}}<a id="Xer1.20"></a>{{< /raw >}}**练习 1.20** `[*]`\
`(count-occurrences s slist)` 返回 `s` 在 `slist` 中的出现次数 。

```scheme {data-open=true}
> (count-occurrences 'x '((f x) y (((x z) x))))
3
> (count-occurrences 'x '((f x) y (((x z) () x))))
3
> (count-occurrences 'w '((f x) y (((x z) x))))
0
```

> [!note]- 答 :
>
> ```scheme {data-open=true}
> ;; 过程合约 : count-occurences : Symbol × S-list   → Natural
> ;; 过程用途 : (count-occurences    s      slist)   = 返回 s 在 slist 中的出现次数
> ;; 实参语法 :                             S-list ::= () | (S-exp . S-list)
> (define (count-occurrences s slist)
>   (if (null? slist) 0
>       (+ (count-occurrences-in-sexp s (car slist))
>          (count-occurrences s (cdr slist)))))
> 
> ;; 过程合约 : count-occurences-in-sexp : Symbol × S-exp   → Natural
> ;; 过程用途 : (count-occurences-in-sexp    s       exp)   = 返回 s 在 sexp 中的出现次数
> ;; 实参语法 :                                     S-exp ::= Symbol | S-list
> (define (count-occurrences-in-sexp s sexp)
>   (if (symbol? sexp)
>       (if (eqv? sexp s) 1 0)
>       (count-occurrences s sexp)))
> ```

{{< raw >}}<a id="Xer1.21"></a>{{< /raw >}}**练习 1.21** `[**]`\
`(product sos1 sos2)` 返回一个元素且皆为 $\textrm{2-List}$ 的列表 , 其代表了 
`sos1` 和 `sos2` 这两个不含重复元素的列表的笛卡尔积 。元素排列顺序任意 。

```scheme {data-open=true}
> (filter-in number? '(a 2 (1 3) b 7))
(2 7)
> (filter-in symbol? '(a (b c) 17 foo))
(a foo)
```

> [!note]- 答 :
> 
> 
> ```scheme {data-open=true}
> ;; 过程合约 :  product : List-of-(SchemeValue) × List-of-(SchemeValue)   → List-of-(2-list)
> ;; 过程用途 : (product            sos1                    sos2       )   = 求 sos1 和 sos2 的笛卡尔积
> ;; 实参语法 :                                    List-of-(SchemeValue) ::= ({SchemeValue}*)
> (define (product sos1 sos2)
>   (flat-map (lambda (e1)
>               (map (lambda (e2) ;; 借用练习 1.16 的 map
>                      (list e1 e2))
>                    sos2))
>             sos1))
> 
> ;; 过程合约 : flat-map : (List-of-(A) → List-of-(B)) × List-of-(List-of-(A))   → List-of-(B)
> ;; 过程用途 : (flat-map              proc                      lst         )   = 获得 List-of-(List-of-(B)) 后把列表压扁
> ;; 实参语法 :                                          List-of-(List-of-(A)) ::= () | (A . List-of-(List-of-(A)))
> (define (flat-map proc lst)
>   (if (null? lst) '()
>       (append (proc (car lst)) ;; 注意 : 若是 cons 则效果同 map
>               (flat-map proc (cdr lst)))))
> ```

{{< raw >}}<a id="Xer1.22"></a>{{< /raw >}}**练习 1.22** `[**]`\
`(filter-in pred lst)` 利用 `pred` 来筛选出 `lst` 中所有符合条件的元素 。

```scheme {data-open=true}
> (filter-in number? '(a 2 (1 3) b 7))
(2 7)
> (filter-in symbol? '(a (b c) 17 foo))
(a foo)
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 :  filter-in : (SchemeValue → Bool) × List-of-(SchemeValue)   → List-of-(SchemeValue)
> ;; 过程用途 : (filter-in               pred               lst         )   = 筛选出 lst 中所有满足 `pred` 的元素
> ;; 实参语法 :                                     List-of-(SchemeValue) ::= () | (SchemeValue . List-of-(SchemeValue))
> (define (filter-in pred lst)
>   (if (null? lst) '()
>       (let ([head (car lst)]
>             [filtered-tail (filter-in pred (cdr lst))])
>         (if (pred head) (cons head filtered-tail)
>             filtered-tail))))
> ```
> 
> 下面是第二种解法 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 :  filter-in : (SchemeValue → Bool) × List-of-(SchemeValue)   → List-of-(SchemeValue)
> ;; 过程用途 : (filter-in               pred               lst         )   = 筛选出 lst 中所有满足 `pred` 的元素
> ;; 实参语法 :                                     List-of-(SchemeValue) ::= ({SchemeValue}*)
> (define (filter-in pred lst)
>   (flat-map ;; 借用练习 1.21 的 flat-map
>     (lambda (x) 
>       (if (pred x) (list x)
>           '()))
>     lst))
> ```

{{< raw >}}<a id="Xer1.23"></a>{{< /raw >}}**练习 1.23** `[**]`\
`(list-index pred lst)` 返回 `lst` 中第一个满足 `pred` 的元素的索引 ( 从 `0` 开始算起 ) 。
如果 `lst` 没有符合条件的元素则返回 `#f` 。

```scheme {data-open=true}
> (list-index number? '(a 2 (1 3) b 7))
1
> (list-index symbol? '(a (b c) 17 foo))
0
> (list-index symbol? '(1 2 (a b) 3))
#f
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 :  list-index: (SchemeValue → Bool) × List-of-(SchemeValue)   → N
> ;; 过程用途 : (list-index          pred                  ls t         )   = lst 中第一个满足 pred 的元素的索引
> ;; 实参语法 :                                     List-of-(SchemeValue) ::= () | (SchemeValue . List-of-(SchemeValue))
> (define (list-index pred lst)
>   (cond [(null? lst) #f]
>         [(pred (car lst)) 0]
>         [else
>          (let ([index (list-index pred (cdr lst))])
>            (if index (+ 1 index) 
>                #f))]))
> ```

{{< raw >}}<a id="Xer1.24"></a>{{< /raw >}}**练习 1.24** `[**]`\
`(every? pred lst)` 返回 `#t` 仅当 `lst` 中的所有元素皆满足 `pred` , 否则返回 `#f` 。

```scheme {data-open=true}
> (every? number? '(a b c 3 e))
#f
> (every? number? '(1 2 3 5 4))
#t
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : every? : (SchemeValue → Bool) × List-of-(SchemeValue)   → Bool
> ;; 过程用途 : (every?           pred                 lst          )   = 判断 lst 中所有元素是否都满足 pred
> ;; 实参语法 :                                 List-of-(SchemeValue) ::= () | (SchemeValue . List-of-(SchemeValue))
> (define (every? pred lst)
>   (cond [(null? lst) #t]
>         [(pred (car lst)) (every? pred (cdr lst))]
>         [else #f]))
> ```
> 
> 下面是第二种解法 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : every? : (SchemeValue → Bool) × List-of-(SchemeValue)   → Bool
> ;; 过程用途 : (every?           pred                 lst          )   = 判断 lst 中是否存在元素满足 pred
> ;; 实参语法 :                                 List-of-(SchemeValue) ::= ({SchemeValue}*)
> (define (every? pred lst)
>   (foldr (lambda (x b) (and (pred x) b)) #t lst)) 
> 
> ;; 过程合约 : foldr : (A -> B -> B) × B × List-of-(A) -> B
> ;; 过程用途 : (foldr       f2R        i       xs    ) = ...
> ;; 实参语法 :                             List-of-(A) = () | (A . List-of-(A))
> (define (foldr f2R i xs)
>   (if (null? xs) i
>       (f2R (car xs) (foldr f2R i (cdr xs)))))
> ```

{{< raw >}}<a id="Xer1.25"></a>{{< /raw >}}**练习 1.25** `[**]`\
`(exists? pred lst)` 返回 `#t` 仅当 `lst` 中存在元素满足 `pred` , 否则返回 `#f` 。

```scheme {data-open=true}
> (exists? number? '(a b c 3 e))
#t
> (exists? number? '(a b c d e))
#f
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : exists? : (SchemeValue → Bool) × List-of-(SchemeValue)   → Bool
> ;; 过程用途 : (exists?           pred                   lst        )   = 判断 lst 中是否存在元素满足 pred
> ;; 实参语法 :                                  List-of-(SchemeValue) ::= () | (SchemeValue . List-of-(SchemeValue))
> (define (exists? pred lst)
>   (cond [(null? lst) #f]
>         [(pred (car lst)) #t]
>         [else (exists? pred (cdr lst))]))
> ```
> 
> 下面是第二种方案
> 
> ```scheme {data-open=true}
> ;; 过程合约 : exists? : (SchemeValue → Bool) × List-of-(SchemeValue)   → Bool
> ;; 过程用途 : (exists?           pred                   lst        )   = 判断 lst 中是否存在元素满足 pred
> ;; 实参语法 :                                  List-of-(SchemeValue) ::= ({SchemeValue}*)
> (define (exists? pred lst)
>   (foldr (lambda (x b) (or (pred x) b)) #f lst)) ;; 借用练习 1.24 的 foldr
> ```

{{< raw >}}<a id="Xer1.26"></a>{{< /raw >}}**练习 1.26** `[**]`\
`(up lst)` 会移除 `lst` 中每个元素最外层的括号 —— 如果可以的话 , 否则则保留元素 。
注意 : `(up (down lst))` 等同于 `lst` 但 `(down (up lst))` 未必就是 `lst` ( 见[练习 1.17](#Xer1.17) ) 。

```scheme {data-open=true}
> (up '((1 2) (3 4)))
(1 2 3 4)
> (up '((x (y)) z))
(x (y) z)
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : up : List-of-(SchemeValue)   → List-of-(SchemeValue)
> ;; 过程用途 : (up          lst         )   = 尽可能去掉 lst 元素的一层括号
> ;; 实参语法 :      List-of-(SchemeValue) ::= () | (SchemeValue . List-of-(SchemeValue))
> (define (up lst)
>   (cond [(null? lst) '()]
>         [(list? (car lst)) (append (car lst)
>                                    (up (cdr lst)))]
>         [else (cons (car lst)
>                     (up (cdr lst)))]))
> ```
> 
> 下面是第二种解法 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : up : List-of-(SchemeValue)   → List-of-(SchemeValue)
> ;; 过程用途 : (up          lst         )   = 尽可能去掉 lst 元素的一层括号
> ;; 实参语法 :      List-of-(SchemeValue) ::= () | ({SchemeValue}*)
> (define (up lst)
>   (foldr ;; 借用练习 1.24 的 foldr
>    (lambda (todo done)
>      (if (list? todo) (append todo done) (cons todo done)))
>    '()
>    lst))
> ```

{{< raw >}}<a id="Xer1.27"></a>{{< /raw >}}**练习 1.27** `[**]`\
`(flatten slist)` 会移除 `slist` 中元素的所有括号 。

```scheme {data-open=true}
> (flatten '(a b c))
(a b c)
> (flatten '((a) () (b ()) () (c)))
(a b c)
> (flatten '((a b) c (((d)) e)))
(a b c d e)
> (flatten '(a b (() (c))))
(a b c)
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : flatten : S-list   → List-of-(SchemeValue)
> ;; 过程用途 : (flatten  slist)   = 压扁 slist
> ;; 实参语法 :           S-list ::= () | (S-exp . S-list)
> ;; 实参语法 :           S-exp  ::= SchemeValue | S-list
> (define (flatten slist)
>   (cond [(null? slist) '()]
>         [(list? (car slist)) (append (flatten (car slist))
>                                      (flatten (cdr slist)))]
>         [else (cons (car slist)
>                     (flatten (cdr slist)))]))
> ```
> 
> 下面是第二种做法 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : flatten : S-list   → List-of-(SchemeValue)
> ;; 过程用途 : (flatten  slist)   = 压扁 slist
> ;; 实参语法 :           S-list ::= ({S-exp}*)
> ;; 实参语法 :           S-exp  ::= SchemeValue | S-list
> (define (flatten slist)
>   (foldr ;; 借用练习 1.24 的 foldr
>    (lambda (todo done)
>      (if (list? todo) (append (flatten (up todo)) done) (cons todo done)))
>    '()
>    slist))
> ```

{{< raw >}}<a id="Xer1.28"></a>{{< /raw >}}**练习 1.28** `[**]`\
`(merge loi1 loi2)` 会合并递增整数列表 `loi1` 和 `loi2` 并将结果递增排序 。

```scheme {data-open=true}
> (merge '(1 4) '(1 2 8))
(1 1 2 4 8)
> (merge '(35 62 81 90 91) '(3 83 85 90))
(3 35 62 81 83 85 90 90 91)
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : merge : List-of-(Int) × List-of-(Int)   → List-of-(Int)
> ;; 过程用途 : (merge      loi1            loi2    )   = 合并 loi1 和 loi2 并递增排序
> ;; 实参语法 :                         List-of-(Int) ::= () | (Int . List-of-(Int))
> (define (merge loi1 loi2)
>   (cond [(null? loi1) loi2]
>         [(null? loi2) loi1]
>         [(< (car loi1) (car loi2)) (cons (car loi1)
>                                          (merge (cdr loi1) loi2))]
>         [else (cons (car loi2)
>                     (merge loi1 (cdr loi2)))]))
> ```
> 
> 下面是第二种做法 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : merge : List-of-(Int) × List-of-(Int)   → List-of-(Int)
> ;; 过程用途 : (merge      loi1            loi2    )   = 合并 loi1 和 loi2 并递增排序
> ;; 实参语法 :                         List-of-(Int) ::= ({Int}*)
> (define (merge loi1 loi2)
>   (foldr test loi2 loi1)) ;; 借用练习 1.24 的 foldr
> (define (test x lst)
>   (cond [(null? lst) (cons x lst)]
>         [(< x (car lst)) (cons x lst)]
>         [else (cons (car lst) (test x (cdr lst)))]))
> ```

{{< raw >}}<a id="Xer1.29"></a>{{< /raw >}}**练习 1.29** `[**]`\
`(sort loi)` 会对 `loi` 进行递增排序 。

```scheme {data-open=true}
> (sort '(8 2 5 2 3))
(2 2 3 5 8)
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : sort : List-of-(Int)   → List-of-(Int)
> ;; 过程用途 : (sort      loi     )   = 对 loi 进行排序
> ;; 实参语法 :        List-of-(Int) ::= () | (Int . List-of-(Int))
> (define (sort loi)
>   (define (merge-sort lst) ;; merge 见上一题的第二种解法
>     (cond [(null? lst) '()]
>           [(null? (cdr lst)) lst]
>           [else (merge-sort (cons (merge (car lst) (cadr lst))
>                                   (merge-sort (cddr lst))))]))
> ```
> 
> 下面是第二种解法 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : sort : List-of-(Int)   → List-of-(Int)
> ;; 过程用途 : (sort      loi     )   = 对 loi 进行排序
> ;; 实参语法 :        List-of-(Int) ::= (Int . List-of-(Int))
> (define (sort loi)
>   (cond [(null? loi) loi]
>         [else (merge (list (car loi)) (sort (cdr loi)))])) ;; merge 见上一题的第二种解法
> ```

{{< raw >}}<a id="Xer1.30"></a>{{< /raw >}}**练习 1.30** `[**]`\
`(sort/predicate pred loi)` 会根据谓词 `pred` 对 `loi` 进行排序 。

```scheme {data-open=true}
> (sort/predicate < '(8 2 5 2 3))
(2 2 3 5 8)
> (sort/predicate > '(8 2 5 2 3))
(8 5 3 2 2)
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : sort/predicate : (Int → Bool) × List-of-(Int)   → List-of-(Int)
> ;; 过程用途 : (sort/predicate       pred          loi     )   = 根据 pred 对 loi 进行排序
> ;; 实参语法 :                                 List-of-(Int) ::= () | (Int . List-of-(Int))
> (define (sort/predicate pred loi)
>   (define (merge-sort lst)
>     (cond [(null? lst) '()]
>           [(null? (cdr lst)) lst]
>           [else (merge-sort (cons (merge/predicate pred (car lst) (cadr lst))
>                                   (merge-sort (cddr lst))))])) 
>   (car (merge-sort (map list loi)))) ;; 借用练习 1.16 的 map
> 
> ;; 过程合约 : merge/predicate : (Int → Bool) × List-of-(Int) × List-of-(Int)   → List-of-(Int)
> ;; 过程用途 : (merge/predicate      pred           loi1            loi2    )   = ...
> ;; 实参语法 :                                                  List-of-(Int) ::= () | (Int . List-of-(Int))
> (define (merge/predicate pred loi1 loi2)
>   (cond [(null? loi1) loi2]
>         [(null? loi2) loi1]
>         [(pred (car loi1) (car loi2)) (cons (car loi1)
>                                             (merge/predicate pred (cdr loi1) loi2))]
>         [else (cons (car loi2)
>                     (merge/predicate pred loi1 (cdr loi2)))]))
> ```

{{< raw >}}<a id="Xer1.31"></a>{{< /raw >}}**练习 1.31** `[*]`\
请提供下述与二叉树 $\Bintree$ ( 见[定义 1.1.7](#Def1.1.7) ) 相关的过程 : 
`leaf` 和 `interior-node` 用于构建二叉树 , 
`leaf?` 用于判断二叉树是否为叶节点 , 
`lson` , `rson` 和 `contents-of` 则用于提取节点的内容 。
`contents-of` 应对任何节点都有效 。

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 类型语法 : Bintree ::= Int | (Symbol Bintree Bintree)
> ;; 类型构造器 : 
> (define (leaf n)                       ;; 创建叶节点
>   n)
> (define (interior-node name lson rson) ;; 创建非叶节点
>   (list name lson rson))
> 
> ;; 类型谓词 :
> (define (leaf? tree)                   ;; 判断节点是否为叶节点
>   (number? tree))
> 
> ;; 类型观测器 :
> (define lson                           ;; 获取节点左子树
>   cadr)
> (define rson                           ;; 获取节点右子树
>   caddr)
> (define (contents-of tree)             ;; 获取节点内容
>   (cond [(leaf? tree) tree]
>         [else (car tree)]))
> ```

{{< raw >}}<a id="Xer1.32"></a>{{< /raw >}}**练习 1.32** `[*]`\
请给出过程 `double-tree` 的定义 : 
其以一个二叉树 ( 见[定义 1.1.7](#Def1.1.7) ) 为实参
并输出一个结构相同的二叉树 —— 只不过这里每个节点的内容数目都翻倍 。

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : double-tree : Bintree   → Bintree
> ;; 过程用途 : (double-tree   tree )   = 将 tree 中节点的内容翻倍
> ;; 实参语法 :               Bintree ::= Int | (Symbol Bintree Bintree)
> (define (double-tree tree)
>   (maptree (lambda (x) (* x 2)) tree))
> 
> ;; 过程合约 : maptree : (A → B) × Bintree   → Bintree(B)
> ;; 过程用途 : (maptree   proc      tree )   = tree 中所有节点内容都被应用 proc
> ;; 实参语法 :                     Bintree ::= Int | (Symbol Bintree Bintree)
> (define (maptree proc tree)
>   (if (leaf? tree) (leaf (proc tree))
>       (interior-node (contents-of tree)
>                      (maptree proc (lson tree))
>                      (maptree proc (rson tree)))))
> ```

{{< raw >}}<a id="Xer1.33"></a>{{< /raw >}}**练习 1.33** `[**]`\
请给出过程 `mark-leaves-with-red-depth` 的定义 : 
其以一个二叉树 ( 见[定义 1.1.7](#Def1.1.7) ) 为实参
并输出一个结构相同的二叉树 —— 只不过这里每个节点包含的内容为
从自身到根节点之间所有带符号 `red` 的节点的数目 。举个例子 : 
由[练习 1.31](#Xer1.31) 里提到的过程所构建的表达式

```scheme {data-open=true}
(mark-leaves-with-red-depth
 (interior-node 'red
    (interior-node 'bar
                   (leaf 26)
                   (leaf 12))
    (interior-node 'red
                   (leaf 11)
                   (interior-node 'quux
                                  (leaf 117)
                                  (leaf 14)))))
```

应返回下述 $\Bintree$ :

```scheme {data-open=true}
(red
 (bar 1 1)
 (red 2 (quux 2 2)))
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : mark-leaves-with-red-depth : Bintree   → Bintree
> ;; 过程用途 : (mark-leaves-with-red-depth   tree )   =   ...
> ;; 实参语法 :                              Bintree ::= Int | (Symbol Bintree Bintree)     
> (define (mark-leaves-with-red-depth tree)
>   (let mark [(n 0) (node tree)]
>     (cond [(leaf? node) (leaf n)]
>           [(eqv? (contents-of node) 'red)
>            (interior-node 'red
>                           (mark (+ n 1) (lson node))
>                           (mark (+ n 1) (rson node)))]
>           [else (interior-node (contents-of node)
>                                (mark n (lson node))
>                                (mark n (rson node)))])))
> ```


{{< raw >}}<a id="Xer1.34"></a>{{< /raw >}}**练习 1.34** `[***]`\
请给出过程 `path` 的定义 : 
其以一个整数 `n` 以及一个包含 `n` 的二叉搜索树 `bst` ( 见[此处](#Def_BST) ) 作为实参
并返回一个包含 `left` 和 `right` 的列表以指示如何才能找到包含 `n` 的节点 。
如果 `n` 在根结点处被发现 , 那么其将会返回一个空列表 。

```scheme {data-open=true}
> (path 17 '(14 (7 () (12 () ()))
                (26 (20 (17 () ())
                        ())
                    (31 () ()))))
(right left left)
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : path : Int × BinarySearchTree   → List-of-(Symbol)
> ;; 过程用途 : (path   n          tree     )   = 至内容为 n 的节点的路线
> ;; 实参语法 : ?
> (define (path n bst)
>   (let [(value-of car)
>         (lson cadr)
>         (rson caddr)]
>     (cond [(null? bst)           (eopl:error 'path "Element ~s not found" n)]
>           [(= (value-of bst) n) '()]
>           [(< n (value-of bst))  (cons 'left  (path n (lson bst)))]
>           [else                  (cons 'right (path n (rson bst)))])))
> ```

{{< raw >}}<a id="Xer1.35"></a>{{< /raw >}}**练习 1.35** `[***]`\
请给出过程 `number-leaves` 的定义 : 
其以一个二叉树为实参并输出一个类似的二叉树 —— 
不过这里的节点是按照广度优先搜索从 `0` 开始进行标记 。比如

```scheme {data-open=true}
(number-leaves
 (interior-node 'foo
    (interior-node 'bar
                   (leaf 26)
                   (leaf 12))
    (interior-node 'baz
                   (leaf 11)
                   (interior-node 'quux
                                  (leaf 117)
                                  (leaf 14))
```

应当返回

```scheme {data-open=true}
(foo
 (bar 0 1)
 (baz
  2
  (quux 3 4)))
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : number-leaves: Bintree   → Bintree
> ;; 过程用途 : (number-leaves  tree )   = 根据广度优先搜索对 tree 进行标记
> ;; 实参语法 :                Bintree ::= Int | (Symbol Bintree Bintree)
> (define (number-leaves tree)
>   (define (number n node)
>     (if (leaf? node)
>       (list (+ n 1) (leaf n))
>       (let* [(lson-result (number n (lson node)))
>              (lson-n (car lson-result))
>              (new-lson (cadr lson-result))
>              (rson-result (number lson-n (rson node)))
>              (rson-n (car rson-result))
>              (new-rson (cadr rson-result))]
>         (list rson-n (interior-node (contents-of node)
>                                     new-lson
>                                     new-rson)))))
>   (cadr (number 0 tree)))
> ```

{{< raw >}}<a id="Xer1.36"></a>{{< /raw >}}**练习 1.36** `[***]`\
请给出辅助过程 `g` 以使得前面提到的 [`number-elements`](#S1.3) 可通过如下方式定义 : 

```scheme {data-open=true}
;; 过程合约 : number-elements : List-of-(SchemeValue)   → List-of-((Natural SchemeValue))
;; 过程用途 : (number-elements     '(v0 v1 v2 ...))     = ((0 v0) (1 v1) (2 v2) ...)
;; 实参语法 :                   List-of-(SchemeValue) ::= () | (SchemeValue . List-of-(SchemeValue))
(define number-elements
  (lambda (lst)
    (if (null? lst) '()
        (g (list 0 (car lst)) (number-elements (cdr lst))))))
```

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 过程合约 : g : SchemeValue × List-of-(SchemeValue)   → List-of-((Int SchemeValue))
> ;; 过程用途 : (g      head              tail        )   = ...
> ;; 实参语法 :                   List-of-(SchemeValue) ::= () | (SchemeValue . List-of-(SchemeValue))
> (define (g head tail)
>   (if (null? tail) (list head)
>     (let* [(n (car head))
>            (next (car tail))
>            (new-next (cons (+ n 1) (cdr next)))]
>       (cons head (g new-next (cdr tail)))))) ;; 结构有点像 foldr 但并不是 foldr , 除非保证存在一个 f2R 能同时概括 cons 和 new-next
> ```
