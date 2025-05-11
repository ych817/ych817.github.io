---
title: 翻译-EOPL-02-数据抽象
subtitle:
date: 2025-04-02T23:11:17+13:00
slug: 1568c34
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
weight: EOPL02
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
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: true
lightgallery: false
# password:
# message:
repost:
  enable: true
  url:

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
\def\Var{\textrm{Var}}
\def\Bool{\textrm{Bool}}
\def\NodeInSeq{\textrm{NodeInSequence}}
\def\RedBlueTree{\textrm{Red-blue-tree}}
\def\RedBlueSubtree{\textrm{Red-blue-subtree}}
\def\PrefixList{\textrm{Prefix-list}}
$$

## 2.1 通过接口声明数据 {#S2.1}

每当我们以某种方式描述集合时 , 一种新数据类型便会随之产生 。
对这些数据类型下的实体而言 ,
其值为其外在的表示 ,
而基于其的运算便是处理这些实体的过程 。

这些实体的表示通常会很复杂 。如果允许的话 , 我们不会关心它们具体的实现细节 ,
但是我们可能会改变数据的表示 。完美的表示往往难以实现 ,
因此我们会先尝试一些简单的实现 , 只有在系统整体性能与之攸关时才改用更高效的表示 。
如果决定要改变某些数据的表示方式 , 我们需要能定位出程序中所有依赖该表示的部分 。
这就需要借助**数据抽象 data abstraction** 技术 。

数据抽象将数据类型分为两部分 : **接口 interface** 和**实现 implementation** 。接口告诉我们类型表示什么数据 ,
数据可进行什么操作 ,
这些操作可得出什么性质 , 等等 ;
而实现则会给出数据的具体表示以及处理数据表示的代码 。

这样抽象出的数据类型称作**抽象数据类型 abstract data type** 。程序的其余部分 ( 即数据类型
的**用户 client** ) 只能通过接口中指定的操作来处理新数据 。这样一来如果希望改变数据的表示
我们只需改变数据处理接口的实现即可 。

这一想法并不陌生 : 我们写程序处理文件时一般只关心能否调用过程以对文件进行打开 , 关闭 , 
读取或其他操作 。同样地在大多数时候我们并不关心整数在机器中究竟怎样表示 , 只关心能否
可靠地执行算术操作 。

如果数据类型下的项只能通过接口提供的过程进行处理, 那么我们便可将代码称作**表示无关的 representation independent** —— 因为这些代码不依赖这些项的值的表示  。

如此所有关于数据表示的信息必然在实现对应的代码之中 ; 实现中最重要的部分便是声明数据
的表示方式 。我们用记号 $\implement v$ 表示 " $v$ 的表示 " 。

为使这一切更明了 , 考虑一个简单的例子 —— 自然数类型 :  待表示数据为自然数 , 接口由四个
过程组成 : `zero` , `is-zero?` , `successor` , `predecessor` ; 当然不是随便几个过程都能够
作为该接口的实现 , 一组过程只有同时满足下述四个方程才可作为下述接口的实现 : 

$\qquad \begin{aligned}[t]
&\texttt{(zero)} 
&{}={}
&\implement 0 
\\
&\texttt{(is-zero? \implement {$n$})}
&{}={}
&\begin{cases}
\texttt{\#t} & n=0 \\
\texttt{\#f} & n\neq 0
\end{cases}
\\
&\texttt{(successor \implement {$n$})}
&{}={}
&\implement{n+1} & (n\geq 0) 
\\
&\texttt{(predecessor \implement {$n+1$})}
&{}={}
&\implement{n} & (n\geq 0)
\end{aligned}$

该定义并未指明自然数应当如何表示 , 它只要求这些过程产生指定的行为 —— 如 `zero` 必须
返回 `0` 的表示 , `successor` 须返回 `n+1` 的表示 , 等等 ;  该定义并没有对 `zero` 做出说明 , 
所以按照该定义任何行为都是可接受的 。

现在我们可以写出处理自然数的用户程序 ,
并且不论用哪种表示方式都可以保证得出正确结果 。如 , 

```scheme
(define plus
  (lambda (x y)
    (if (is-zero? x) y
        (successor (plus (predecessor x) y)))))
```

都将满足 $\texttt{(plus \implement {$x$}  \implement {$y$})} = \implement {x+y}$ , 无论采用的表示方式为如何 。

大多数接口都包含若干用于创建数据类型的元素的**构造器 constructor** 及若干用于从数据类型
的元素中提取信息的**观测器 observer** 。这里也不例外 : 有三个构造器 `zero` , `successor` 及 
`predecessor` 外加一个观测器 `is-zero?` 。

可以用多种方式表示这套接口 。我们考虑其中三种 :

1. **一元表示法 Unary representation** :
   在一元表示法中自然数 `n` 表示为 `n` 个 `#t` 构成的列表 ; 如此

   $\qquad \begin{aligned}[t]
   &\implement 0 
   &{}={}
   &\texttt{()}
   \\
   &\implement 1
   &{}={}
   &\texttt{(\#t)}
   \\
   &\implement 2
   &{}={}
   &\texttt{(\#t \#t)}
   \end{aligned}$

   如此便可递归定义该表示法 : 

   $\qquad \begin{aligned}[t]
   &\implement 0 
   &{}={}
   &\texttt{()}
   \\
   &\implement {n+1}
   &{}={}
   &\texttt{(\#t . n)}
   \\
   \end{aligned}$

   在该表示法中我们可通过下述代码实现接口

   ```scheme
   (define zero        (lambda ()  '()))
   (define is-zero?    (lambda (n) (null? n)))
   (define successor   (lambda (n) (cons #t n)))
   (define predecessor (lambda (n) (cdr n)))
   ```

2. **Scheme 数字表示法 Scheme number representation** :
   该表示仅涉及 Scheme 内置的数字表示法 ( 本身可能十分复杂 ! ) 。
   如果令 $\implement n$ 为 Scheme 整数 $n$ , 则所需的四个过程可以定义为：

   ```
   (define zero        (lambda ()  0))
   (define is-zero?    (lambda (n) (zero? n)))
   (define successor   (lambda (n) (+ n 1)))
   (define predecessor (lambda (n) (- n 1)))
   ```

3. **大数表示法 Bignum representation** : 
   此表示法中数值以 $N$ 进制表示 , 其中 $N$ 是某个整数 ; 该方法以 $0$ 到 $N-1$ 之间的数字 ( 
   有时不称数位而称**大位 bigit** ) 组成的表来表示数值 , 这就很容易表示那些远超机器字长
   的整数 。为了便使用 , 这里把最低位放在列表最前端 。该表示法可通过归纳法如下定义 : 

   $\qquad \begin{aligned}[t]
   &\implement n 
   &{}={}
   & 0 && \textrm{若 $n=0$} \\
   &&
   & \texttt{(r . \implement {$q$})} &&\textrm{若 $n=qN+r$ 且 $0\leq r < N$}
   \end{aligned}$

   因此若 $N=16$ 则 $\implement {33}=\texttt{(1 2)}$ 且 $\implement {258} =\texttt{(2 0 1)}$ —— 因为

   $\qquad 258 = 2\times 16^0+0\times 16^1+1\times 16^2$ 

上述表示法都没有强制数据抽象 ,
没有东西可以阻止用户程序查验表示法用的究竟是列表还是 Scheme 整数 。
有些语言对数据抽象有直接支持 , 它们允许程序员创建新接口并确保只能通过
接口提供的过程处理新数据 。若类型表示被隐藏且不会因任何操作而暴露 ( 包括打印 ) 则称作**模糊 opaque** 的 , 否则称为**透明 transparent** 的 。

Scheme 并未提供标准机制来创建新的模糊类型 , 因此我们采取一种折衷的抽象级别 —— 定义
接口并劳烦用户程序的作者谨慎行事 , 只使用接口中定义的过程 。

在第 8 章 ( 模块 ) 将会研究一些方法以强化语言的这种行为准则 。

{{< raw >}}<a id="Xer2.1"></a>{{< /raw >}}**练习 2.1** `[*]`\
尝试实现大数表示法的四种接口并用你的实现求 10 的阶乘 。执行时间将会随着参数 / 进制
的改变如何变化 ? 请解释原因 。

> [!note]- 答 :
> 
> ```scheme {data-open=true}
> ;; 设置大数表示法的基底
> (define base 10)
> (define (set-base! n)
>   (set! base n))
> 
> ;; 下面是大数表示法的四个接口
> (define (zero) '())
> (define is-zero? null?)
> (define (successor n)
>   (cond [(is-zero? n) '(1)]
>         [(< (car n) (- base 1)) (cons (+ 1 (car n))
>                                       (cdr n))]
>         [else (cons 0 (successor (cdr n)))]))
> (define (predecessor n)
>   (cond [(= 1 (car n)) (if (null? (cdr n)) (zero)
>                            (cons 0 (cdr n)))]
>         [(> (car n) 0) (cons (- (car n) 1) (cdr n))]
>         [else (cons (- base 1) (predecessor (cdr n)))]))
> 
> 
> ;; 整数与大数表示法的互相转换
> (define (integer->bigits n)
>   (if (zero? n) (zero)
>       (successor (integer->bigits (- n 1)))))
> (define (bigits->integer n)
>   (if (is-zero? n) 0
>       (+ 1 (bigits->integer (predecessor n)))))
> 
> ;; 加法和乘法
> (define (plus x y)
>   (if (is-zero? x) y
>       (successor (plus (predecessor x) y))))
> (define (multiply x y)
>   (cond [(is-zero? x) (zero)]
>         [(is-zero? (predecessor x)) y]
>         [else (plus y (multiply (predecessor x) y))]))
> 
> ;; 阶乘
> (define (factorial n)
>   (if (is-zero? n) (successor (zero))
>       (multiply n (factorial (predecessor n)))))
> 
> 
> ;; 测试
> (define (with-base n thunk)
>   (let [(orig-base base)]
>     (begin
>       (set-base! n)
>       (thunk)
>       (set-base! orig-base))))
> (define (print-factorial base n)
>   (with-base
>     base
>     (lambda ()
>       (collect-garbage)
>       (eopl:printf
>         "Base ~s: ~s! = ~s~%"
>         base
>         n
>         (bigits->integer (factorial (integer->bigits n)))))))
> ```
> 
> 注意到 :
> 
> - `(factorial n)` 中的 `n` 与执行时间成正比 ( 原因在于阶乘运算本身的时间复杂度 ) ;
> - `(set-base! n)` 中的 `n` 与执行时间成反比 ( 大基数在表示数字时所需的位数更少 ) 。

{{< raw >}}<a id="Xer2.2"></a>{{< /raw >}}**练习 2.2** `[**]`\
详细分析上述表示法 : 从满足数据类型定义的角度来说 , 在何种程度上它们是成功 / 失败的 ?

> [!note]- 答 :
> 
> | 表示法       | 优点                                                         | 缺点                                                         |
> | -----------  | ------------------------------------------------------------ | ------------------------------------------------------------ |
> | 一元         | 实现简单 , 完全符合规范 , 仅受物理内存容量的限制       | 性能差 , 另外在表示大数字时几乎不可读                       |
> | Scheme  数字 | 实现简单 , 表示方式易于阅读 , 性能良好 ( 完全由底层具体的 Scheme 实现决定 ) | 对于没有内建大数支持的 Scheme 实现计算结果可能会溢出，因此不能完全符合规范 |
> | 大数         | 完全符合规范 , 性能相对较好 ( 较大的基数会带来更好的性能 ) | 依赖于基数 , 不容易实现                                   |


{{< raw >}}<a id="Xer2.3"></a>{{< /raw >}}**练习 2.3** `[**]`\
用差分树表示所有整数 ( 负数和非负数 ) 。差分树是一个列表 , 对应的语法定义如下：

$\qquad \begin{aligned}[t]\DiffTree ::={}& \texttt{(one)} \mid \texttt{(diff \DiffTree  \DiffTree)}\end{aligned}$

列表 $\texttt{(one)}$ 表示 $1$ ; 另外若 $t_1$ 表示 $n_1$ 且 $t_2$ 表示 $n_2$ 则 $\texttt{(diff $t_1$ $t_2$)}$ 表示 $n_1 - n_2$ 。

故 $\texttt{(one)}$ 和 $\texttt{(diff (one) (diff (one) (one)))}$ 都表示 $1$ ; 
而 $\texttt{(diff (diff (one) (one)) (one))}$ 表示的数则是 $-1$ 。

1. 证明此系统中的每个数都有无限种表示方式 。

   > [!note]- 证 : 反证法 。
   > 
   > 假如 $S$ 为自然数 $n$ 的所有表示所构成的集 , 那么对任意 $x\in S$ 我们定义集合 $S'$ 为满足下述两条推理规则的集合 :
   > 
   > 1. $\begin{prooftree}\AXC{}\UIC{\(x\in S'\)}\end{prooftree}$
   > 
   > 2. $\begin{prooftree}\AXC{\(y\in S'\)}\UIC{\(\texttt{(diff \(y\) (diff (one) (one)))}\)}\end{prooftree}$
   > 
   > 不难看出 $S'$ 是无穷集 ; 而 $S'\subseteq S$ , 故 $S$ 亦为无穷集 。

2. 完整实现这种整数表示法  ——  即写出整数定义的 `zero` , `is-zero?` , `successor` 
   和 `predecessor` 。此外还要能表示负数 。此表示法中整数的任何合法表示都可以
   作为你过程的参数 : 如过程 `successor` 可接受无限多种 `1` 的合法表示且最终都能
   给出一个 `2` 的合法表示 ; 对 `1` 的不同合法表示皆可给出对应的 `2` 的合法表示 。
   
   > [!note]- 答 : ==TODO==
   > 
   > ```scheme {data-open=true}
   > 
   > ```

3. 写出过程 `diff-tree-plus` 并用这种表示做加法 。你的过程应针对不同的差分树
   进行优化并在常数时间内得出结果 ( 即与输入大小无关 ) 。注意本题不能使用递归 。

## 2.2 数据类型的表示策略 {#S2.2}

一旦使用了数据抽象 , 程序便拥有了表示无关性 —— 其与用来实现抽象数据类型的具体的
表示方式无关 。我们甚至还可以通过重新定义接口中的一小部分来改变数据的表示 。在
后面的章节会经常用到此性质 。

本节将介绍几种数据类型的表示策略 : 我们将通过**环境 environment** 数据类型对这些策略
进行探究 ; 环境会将有限个变量与这些变量对应的值逐个关联起来 。在编程语言的实现中
环境可用来维系变量与值之间的关系 , 编译器也可借由环境将变量名与变量信息关联起来 。

变量可用任何我们喜欢的方式来表示 , 只要能够检查两个变量是否相等即可 。这里我们用 
Scheme 符号表示变量 , 但在没有符号数据类型的语言中变量也可用字符串 、哈希表引用
甚至是数字来表示 ( 见 [3.6 节](/posts/a3b7a27/#S3.6) ) 。

### 2.2.1 环境的接口 {#S2.2.1}

环境是个函数 , 其定义域为变量构成的有限集 , 值域为所有 Scheme 值构成的集 。数学中
有限函数常被解释为由有序对组成的有限集 ;  而本书决定采用该释义 , 因此就必须表示出
所有形如 $\texttt{(($var_1$,$val_1$),...,($var_n$,$val_n$))}$ 的集 ,  其中 $var_i$ 为某一变量而 $val_i$ 为任意的
Scheme 值 。有时也将环境 $env$ 中变量 $var$ 的值 $val$ 称作是 $var$ 在 $env$ 中的**绑定 binding** 。

该数据类型的接口有三个过程 , 定义如下 :

$\qquad\begin{aligned}[t]
&\texttt{(empty-env)}
&{}={}
&\implement \varnothing
\\
&\texttt{(apply-env \implement {$f$}  $var$)}
&{}={}
&f(var)
\\
&\texttt{(extend-env $var$ $val$ \implement {$f$})}
&{}={}
&\implement g , &其中~
g(var_1) ={}
\begin{cases}
val      & 如果~ var_1=var\\
f(var_1) & 否则
\end{cases}
\end{aligned}$


过程 `empty-env` 不接受参数并返回空环境的表示 ;  `apply-env` 将环境 $\implement f$ 应用到 $env$ ; 
组合式 $\texttt{(extend-env $var$ $val$ $env$)}$ 将产生一个行为同 $env$ 的新环境 ,  只不过新环境里
变量 $var$ 的值会被设为 $val$ 。例如 : 表达式

```scheme
(define e
  (extend-env 'd 6
    (extend-env 'y 8
      (extend-env 'x 7
        (extend-env 'y 14)))))
```

定义了一个环境 $e$ 满足 $e(\texttt{d}) = 6$ , $e(\texttt{x}) = 7$ , $e(\texttt{y}) = 8$ 并且 $e$ 对任何其他变量都是未定义的 ; 
当然这只是生成该环境的众多方法之一 。在刚才的例子中 , $\tt y$ 先被绑定到 $14$ 后被绑定到 $8$ 。

与上一个例子类似 , 我们可将接口的过程分为构造器和观测器 ; 本例中的 `empty-env` 以及
`extend-env` 是构造器 , 而 `apply-env` 则是唯一的观测器 。

{{< raw >}}<a id="Xer2.4"></a>{{< /raw >}}**练习 2.4** `[**]`\
考虑**栈 stack** 数据类型 , 其接口有 `empty-stack` , `push` , `pop` , `top` , `empty-stack?` 。
请模仿示例中的方式写出这些过程的定义 。另外上述过程中哪些是构造器 , 哪些是观测器 ?  

### 2.2.2 数据结构表示法 {#S2.2.2}

易知每个环境都是从空环境开始搭建并由 $n$ 次调用 `extend-env` 以获得 , 其中 $n\geq 0$ 。如

```scheme
(extend-env varn valn
  ...
  (extend-env var1 val1
    (empty-env)) ... )
```

由此便可获得环境的一种表示方法 。

$\qquad \begin{aligned}[t]
\EnvExp 
::={} & \texttt{(empty-env)} \\
::={} & \texttt{(extend-env \Iden  \ScmVal  \EnvExp)}
\end{aligned}$

我们可用描述列表集合 $\List$ 的递归语法来描述环境 , 如此便可获得[图 2.1](#G2.1) 中的实现 。过程 
`apply-env` 会查看表示环境的数据结构以判断它表示哪种环境并做出适当操作 : 若它表示
空环境则会报错 ; 若它表示的是由 `extend-env` 生成的环境则判断待查找的变量是否会与
环境中绑定的某一变量相同 : 如果是则返回保存的值 , 否则则在保存的环境中查找变量 。

这是种常见的代码模式 , 我们管它叫**解释器配方 interpreter recipe** :

> **解释器配方**
>
> 1. 查看一段数据 。
> 2. 判断它表示什么样的数据 。
> 3. 提取数据的各个部分并对它们做适当操作 。

下方为图 2.1 —— 环境的数据结构表示 。{{< raw >}}<a id="G2.1"></a>{{< /raw >}}

```scheme {data-open=true}
;; Env = (empty-env) | (extend-env Var SchemeVal Env)
;; Var = Sym

;; empty-env : () → Env
(define empty-env
  (lambda () 
    (list 'empty-env)))

;; extend-env : Var × SchemeVal × Env → Env
(define extend-env
  (lambda (var val env)
    (list 'extend-env var val env)))

;; apply-env : Env × Var → SchemeVal
(define apply-env
  (lambda (env search-var)
    (cond
      [(eqv? (car env) 'empty-env)
       (report-no-binding-found search-var)]
      [(eqv? (car env) 'extend-env)
       (let ((saved-var (cadr env))
             (saved-val (caddr env))
             (saved-env (cadddr env)))
         (if (eqv? search-var saved-var)
             saved-val
             (apply-env saved-env search-var)))]
      [else
       (report-invalid-env env)])))
(define report-no-binding-found
  (lambda (search-var)
    (eopl:error 'apply-env "No binding for ~s" search-var)))
(define report-invalid-env
  (lambda (env)
    (eopl:error 'apply-env "Bad environment: ~s" env)))
```

{{< raw >}}<a id="Xer2.5"></a>{{< /raw >}}**练习 2.5** `[*]`\
我们可以用任何数据结构表示空环境 —— 只要能区分空 / 非空环境并且可以从后者中提取出
数据片段 。请按下述要求实现环境 : 空环境由空列表表示 , 而 `extend-env` 将生成如下环境：
$
% \qquad \begin{xy}0;p+/r+10pt/:p+/u+10pt/::
% (0,0)*+=<10pt>[F-]{}="point-to-var1-pair";
% (1,0)*+=<10pt>[F-]{}="point-to-env1"     ;
% {\ar{-};(1,0);p!R+/r+1.5cm/|(1.6){saved-env}};
% "point-to-var1-pair"+/d+1.5cm/;p+/r+10pt/:p+/d+10pt/::
% (0,0)*+=<10pt>[F-]{}="var1-pair-var";
% (1,0)*+=<10pt>[F-]{}="var1-pair-val";
% {\ar{-};0;"var1-pair-var"};
% {\ar{-};(0,0);p!R+/dl+1.5cm/|(1.2){saved-var}};
% {\ar{-};(1,0);p!R+/dr+1.5cm/|(1.2){saved-val}};
% \end{xy}
$

{{< svg-file src="EOPL/Xer2.5.svg" >}}


这叫做**关联列表 association-list 表示法**或 **a-list 表示法** 。

{{< raw >}}<a id="Xer2.6"></a>{{< /raw >}}**练习 2.6** `[*]`\
请发明至少三种环境接口表示 , 并给出它们具体的实现 。

{{< raw >}}<a id="Xer2.7"></a>{{< /raw >}}**练习 2.7** `[*]`\
重写[图 2.1](#G2.1) 中的 `apply-env` 并给出更详细的错误信息 。

{{< raw >}}<a id="Xer2.8"></a>{{< /raw >}}**练习 2.8** `[*]`\
为环境接口添加观测器 `empty-env?` 并用 a-list 表示法实现它 。

{{< raw >}}<a id="Xer2.9"></a>{{< /raw >}}**练习 2.9** `[*]`\
为环境接口添加观测器 `has-binding?` : 其取一个环境 $env$ 以及一个变量 $s$ 并判断 $s$ 在 
$env$ 中是否有绑定值 。请用 a-list 表示法实现它 。

{{< raw >}}<a id="Xer2.10"></a>{{< /raw >}}**练习 2.10** `[**]`\
为环境接口添加构造器 `extend-env*` 并用 a-list 表示法实现它 : 该构造器接受的参数有 : 
一个变量列表 , 一个长度相等的值列表以及一个环境 。其定义为 :

$\qquad \texttt{(extended-env* ($var_1$ ... $var_k$) ($val_1$ ... $val_k$) \implement {$f$})} = \implement g$ , 其中

$\qquad g = \begin{cases}
val_i & \textrm{若存在 $i$ 满足 $1\leq i\leq k$ 且 $var=var_i$} \\
f(var)& \textrm{否则}
\end{cases}$

{{< raw >}}<a id="Xer2.11"></a>{{< /raw >}}**练习 2.11** `[**]`\
考虑前一题中 `extend-env*` 较为拙劣的实现 , 其运行时间与 $k$ 成正比 。有一种表示可使得 
`extend-env*` 的运行时间为常数 : 及用空列表表示空环境并用下面的数据结构表示非空环境 : 
$
%\qquad \qquad \begin{xy}0;p+/r+10pt/:p+/d+10pt/::
%(0,0)="var1-pair-c"*+=<10pt>[F-]{}="var1-pair";
%(1,0)="env1-c"     *+=<10pt>[F-]{}="env1"     ;
%"env1"+/r+5cm/;p+/r+10pt/:p+/d+10pt/::
%(0,0)="var2-pair-c"*+=<10pt>[F-]{}="var2-pair";
%(1,0)="env2-c"     *+=<10pt>[F-]{}="env2"     ;
%{\ar{-};"env1-c";"var2-pair"^{\text{backbone}}};
%"env2"+/r+5cm/;p+/r+10pt/:p+/d+10pt/::
%(0,0)="var3-pair-c"*+=<10pt>[F-]{}="var3-pair";
%(1,0)="env3-c"     *+=<10pt>[F-]{}="env3"     ;
%{\ar{-};"env2-c";"var3-pair"^{\text{backbone}}};
%{\ar{-};"env3-c";p!R+/r+3cm/|(1.3){{\text{rest of env}}}};
%"var1-pair"+/d+1.5cm/;p+/r+10pt/:p+/d+10pt/::
%(0,0)*+=<10pt>[F-]{}="var1-pair-var";
%(1,0)*+=<10pt>[F-]{}="var1-pair-val";
%{\ar{-};"var1-pair-c";"var1-pair-var"};
%{\ar{-};(0,0);p!R+/dl+1.5cm/|(1.2){\texttt{(a b c)}}};
%{\ar{-};(1,0);p!R+/dr+1.5cm/|(1.2){\texttt{(11 12 13)}}};
%"var2-pair"+/d+1.5cm/;p+/r+10pt/:p+/d+10pt/::
%(0,0)*+=<10pt>[F-]{}="var2-pair-var";
%(1,0)*+=<10pt>[F-]{}="var2-pair-val";
%{\ar{-};"var2-pair-c";"var2-pair-var"};
%{\ar{-};(0,0);p!R+/dl+1.5cm/|(1.2){\texttt{(x z)}}};
%{\ar{-};(1,0);p!R+/dr+1.5cm/|(1.2){\texttt{(66 77)}}};
%"var3-pair"+/d+1.5cm/;p+/r+10pt/:p+/d+10pt/::
%(0,0)*+=<10pt>[F-]{}="var3-pair-var";
%(1,0)*+=<10pt>[F-]{}="var3-pair-val";
%{\ar{-};"var3-pair-c";"var3-pair-var"};
%{\ar{-};(0,0);p!R+/dl+1.5cm/|(1.2){\texttt{(x y)}}};
%{\ar{-};(1,0);p!R+/dr+1.5cm/|(1.2){\texttt{(88 99)}}};
%\end{xy}
$

{{< svg-file src="EOPL/Xer2.11.svg" >}}

该表示法称作是**肋排 ribcage** 表示法 , 环境由名为**肋骨 rib** 的有序对列表表示 , 每根左肋指向
变量列表而右肋则指向变量列表对应的值的列表 。

### 2.2.3 过程表示法 {#S2.2.3}

注意到环境接口有一个重要性质 : 它的观测器只有一个 , 即 `apply-env`  。
这意味着仅利用以变量为实参并返回绑定值的 Scheme 过程便可以表示环境 。

若要使用此方式表示环境则 `empty-env` 和 `extend-env` 就应该返回一个过程 :
这个过程在被调用时行为将和[上一节提到的 `apply-env`](#G2.1) 一样 。
如此我们便可得出下述实现 :

```scheme {data-open=true}
;; Env = Var -> SchemeVal

;; empty-env : () -> Env
(define empty-env
  (lambda ()
    (lambda (search-var)
      (report-no-binding-found search-var))))

;; extend-env : Var × SchemeVal × Env → Env
(define extend-env
  (lambda (saved-var saved-val saved-env)
    (lambda (search-var)
      (if (eqv? search-var saved-var)
          saved-val
          (apply-env saved-env search-var)))))

;; apply-env : Env × Var → SchemeVal
(define apply-env
  (lambda (env search-var)
    (env search-var)))
```

如果空环境由 `empty-env` 所创建且被传入了任何可能的参数 , 那么该环境便会抛出错误并
指明给定的变量不位于该环境中 。过程 `extend-env` 返回的过程则是代表了被扩充的环境 : 
若该环境被传入变量 `search-var`  则该环境会检查该变量是否与环境中已绑定的变量相同 :
如果是则返回保存的值 , 否则则在已保存的环境中查找它 。

上述表示法称作**过程表示法 procedual representation** , 其中的数据皆被表示为 `apply-env` 
所执行的动作 。

只有一个观测器的数据类型并非大家所想的那般罕见 , 如 : 当数据被表示为一组函数时 , 那么
它们就可以用自身被应用的这个动作作为表示 。这种情况下可以遵循下述步骤提取出对应的
接口以及对应过程的表示 : 

1. 找出用户代码中那些返回结果类型为指定类型的 $\texttt{lambda}$ 演算表达式 , 并为每个这样的 
   $\texttt{lambda}$ 演算表达式都写一个构造器过程 :   构造器过程接收的参数将会是 $\texttt{lambda}$ 演算
   表达式中的自由变量 。在用户代码中用构造器调用替换对应的 $\texttt{lambda}$ 演算表达式 。
2. 像定义 `apply-env` 那样去定义一个 `apply-` 过程 。找出用户代码中所有有用到指定
   类型值的地方 , 包括构造器过程的主体 。所有用到指定类型值的地方都改用  `apply-` 过程 。

一旦完成上面的步骤 , 那么接口将只含构造器过程以及 `apply-` 过程 , 并且用户代码将会是
表示无关的 : 它将不再依赖表示 , 我们可以随意换用其他接口的实现 —— 正如 [2.2.2 节](S2.2.2)所述 。

若用于实现的语言不支持高阶过程则得再做进一步处理 : 用数据结构表示法和解释器配方来
实现所需的接口 , 就像[上一节](#G2.1)所描述的那样 。这样的操作叫做**去函数化 defunctionalization** ,
环境的各种不同的数据结构表示法都是去函数化的简单例子 。过程表示法与去函数化表示法
间的关系将会是本书反复出现的主题 。

{{< raw >}}<a id="Xer2.12"></a>{{< /raw >}}**练习 2.12** `[*]`\
用过程表示法实现[练习 2.4](#Xer2.4) 中的栈数据类型 。

{{< raw >}}<a id="Xer2.13"></a>{{< /raw >}}**练习 2.13** `[**]`\
扩展过程表示法以实现 `empty-env?` —— 利用两个过程组成的列表来表示环境 :
一个过程像前面那样返回变量所对应的绑定值 , 另一个则判断环境是否为空环境 。

{{< raw >}}<a id="Xer2.14"></a>{{< /raw >}}**练习 2.14** `[**]`\
扩展上题中的表示法 , 加入第三个过程并用它实现 `has-binding?` ( 见[练习 2.9](#Xer2.9) ) 。

## 2.3 递归数据类型的接口 {#S2.3}

第一章我们都在操纵递归数据类型 , 比如 : 在[定义 1.1.8](/posts/5cce8df/#Def1.1.8) 给出了 $\texttt{lambda}$ 演算表达式的语法 :

$\qquad \begin{aligned}[t]
\LcExp   ::={}& 
\Iden \\ ::={}&
\texttt{(lambda (\Iden) \LcExp)} \\::={}&
\texttt{(\LcExp  \LcExp)}
\end{aligned}$

并且还给出了如 `occurs-free?` 这样的过程 。正如当时所述 , [1.2.4 节](/posts/5cce8df/#S1.2.4)的 `occurs-free?` 
的定义可能并非那么易懂 , 如 : 我们很难猜到 `(car (cadr exp))` 指代的是 $\texttt{lambda}$ 演算
表达式的变量声明 , 以及 `(caddr exp)` 指代的是 $\texttt{lambda}$ 演算表达式的主体 , 等等 。

我们可通过为 $\texttt{lambda}$ 演算表达式添加一套接口以改善这种现状 : 我们的接口将会有一系列
构造器以及两种观测器 : **谓词 predicate** 和**提取器 extractor** 。

构造器有 : 

$\qquad\begin{aligned}[t]
&\texttt{var-exp}
&{}:{}
&\Var\to \LcExp 
\\
&\texttt{lambda-exp}
&{}:{}
&\Var\times\LcExp \to \LcExp
\\
&\texttt{app-exp}
&{}:{}
&\LcExp\times\LcExp \to \LcExp
\end{aligned}$

谓词有 :

$\qquad\begin{aligned}[t]
&\texttt{var-exp?}
&{}:{}
&\LcExp \to \Bool
\\
&\texttt{lambda-exp?}
&{}:{}
&\LcExp \to \Bool
\\
&\texttt{app-exp?}
&{}:{}
&\LcExp \to \Bool
\end{aligned}$

最后 , 提取器有 :

$\qquad\begin{aligned}[t]
&\texttt{var-exp->var}
&{}:{}
&\LcExp \to \Var
\\
&\texttt{lambda-exp->bound-var}
&{}:{}
&\LcExp \to \Var
\\
&\texttt{lambda-exp->body}
&{}:{}
&\LcExp \to \LcExp
\\
&\texttt{app-exp->rator}
&{}:{}
&\LcExp \to \LcExp
\\
&\texttt{app-exp->rand}
&{}:{}
&\LcExp \to \LcExp
\end{aligned}$

各个提取器对应 $\texttt{lambda}$ 演算表达式中的各个组成部分 。
我们现在可以写出一个仅依赖接口的 `occurs-free?` :{{< raw >}}<a id="S2.3-occurs-free"></a>{{< /raw >}}

```scheme
;; occurs-free? : Sym × LcExp -> Bool
(define occurs-free?
  (lambda (search-var exp)
    (cond [(var-exp? exp) 
           (eqv? search-var (var-exp->var exp))]
          [(lambda-exp? exp)
           (and (not (eqv? search-var (lambda-exp->bound-var exp)))
                (occurs-free? search-var (lambda-exp->body exp)))]
          [else
           (or (occurs-free? search-var (app-exp->rator exp))
               (occurs-free? search-var (app-exp->rand  exp)))])))
```

这对任何的 $\texttt{lambda}$ 演算表达式的表示法都行得通 —— 只要它们是通过上述构造器构建的 。

我们可以写出设计递归数据类型接口的通用步骤 : 

> **为递归数据类型设计接口**
>
> 1. 为数据类型的每种变体添加一个构造器 。
> 2. 为数据类型的每种变体添加一个谓词 。
> 3. 为每段传入数据类型构造器的数据都添加一个提取器 。

{{< raw >}}<a id="Xer2.15"></a>{{< /raw >}}**练习 2.15** `[*]`\
请实现上述 $\texttt{lambda}$ 演算表达式表示法的各个接口 。

{{< raw >}}<a id="Xer2.16"></a>{{< /raw >}}**练习 2.16** `[*]`\
修改实现以使其适用于另一种表示法 —— 其中 $\texttt{lambda}$ 演算表达式中绑定变量周围不带括号 。

{{< raw >}}<a id="Xer2.17"></a>{{< /raw >}}**练习 2.17** `[*]`\
请发明至少两种表示法来呈现 $\LcExp$ 数据类型并实现它们 。

{{< raw >}}<a id="Xer2.18"></a>{{< /raw >}}**练习 2.18** `[*]`\
我们通常将值的序列呈现为列表 , 如此允许我们轻松地从序列的一个元素移动到下一个元素 ; 
但是若不借助语境参数我们很难实现上述操作 。请实现非空的双向整数序列 , 对应的语法为

$\qquad \begin{aligned}[t]\NodeInSeq ::={}& \texttt{(\Int  \ListOf{\Int} \ListOf{\Int})}\end{aligned}$

第一个整数列表是当前元素之前的序列 , 反向排列 ; 第二个列表是当前元素之后的序列 。例 :
`(6 (5 4 3 2 1) (7 8 9))` 表示列表 `(1 2 3 4 5 6 7 8 9)` , 当前元素为 `6` 。

对于此表示法 , 请实现过程 `number->sequence` —— 它取一数字并生成只含该数字的序列 。
然后再实现 `current-element` , `move-to-left` ,  `move-to-right` , `insert-to-left` , 
`insert-to-right` , `at-left-end?` , `at-right-end?` 。

例如 : 

```scheme
> (number->sequence 7)
(7 () ())
> (current-element '(6 (5 4 3 2 1) (7 8 9)))
6
> (move-to-left '(6 (5 4 3 2 1) (7 8 9)))
(5 (4 3 2 1) (6 7 8 9))
> (insert-to-left  13 '(6 (5 4 3 2 1) (7 8 9)))
(6 (13 5 4 3 2 1) (7 8 9))
> (insert-to-right 13 '(6 (5 4 3 2 1) (7 8 9)))
(6 (5 4 3 2 1) (13 7 8 9))
```

如果参数在序列的最右端那么过程 `move-to-right` 应失败 ; 
如果参数在序列的最左端那么过程 `move-to-left` 应失败 。

{{< raw >}}<a id="Xer2.19"></a>{{< /raw >}}**练习 2.19** `[*]`\
无叶节点的二叉树以及非叶节点全被整数标记的二叉树可表示为下述递归语法

$\qquad \begin{aligned}[t]
\BinSearchTree ::={}& \texttt{()} \mid \texttt{(\Int  \BinSearchTree  \BinSearchTree)}
\end{aligned}$

对于此表示法 , 请实现过程 `number->bintree` —— 其取一个整数并产生一颗新的二叉树 , 
该树的唯一节点被该数字标记 ; 接着实现 `current-element` , `move-to-left-son` 以及
`move-to-right-son` , `at-leaf?` , `insert-to-left` 和 `insert-to-right` 。例如：

```scheme {data-open=true}
> (number->bintree 13)
(13 () ())
> (define t1 (insert-to-right 14 
              (insert-to-left 12 
              	(insert-to-left 13))))
> t1
(13
 (12 () ())
 (14 () ()))
> (move-to-left t1)
(12 () ())
> (current-element (move-to-left t1))
12
> (at-leaf? (move-to-right (move-to-left t1)))
#t
> (insert-to-left 15 t1)
(13
 (15 (12 () ())  ())
 (14 () ()))
```

{{< raw >}}<a id="Xer2.20"></a>{{< /raw >}}**练习 2.20** `[***]`\
在[上一道练习](#Xer2.18)的二叉树中我们很容易从父节点移动到某个子节点 ; 但若无语境参数的辅助
无法从子节点移动到父节点 。请扩展[练习 2.18](#Xer2.18) 中的列表表示法使其表示二叉树中的节点 。
提示 : 二叉树中当前节点上方的部分可以考虑试着用逆序列表来表示 , 正如[练习 2.18](#Xer2.18) 那样 。

对于此表示法 , 请实现[练习 2.19](#Xer2.19) 中的过程 , 然后实现 `move-up` , `at-root?` 及 `at-leaf?` 。

## 2.4 一个定义递归数据类型的工具 {#2.4}

对复杂的数据类型应用上述步骤设计接口很容易让人疲乏 。本节将我们将引入一个工具
用于自动构建和实现诸如上述 Scheme 接口 : 该工具生成的接口虽然与上一节的非常相似 , 
但它们并非完全一致 。

同样考虑[上一节的 $\texttt{lambda}$ 演算表达式数据类型](#S2.3) , 我们可以如下实现 $\texttt{lambda}$ 演算表达式
类型的接口 : {{< raw >}}<a id="S2.4-lc-exp"></a>{{< /raw >}}

```scheme {data-open=true}
(define-datatype lc-exp lc-exp?
   (var-exp
    	(var identifier?))
   (lambda-exp
    	(bound-var identifier?)
    	(body      lc-exp?))
   (app-exp
    	(rator lc-exp?)
    	(rand  lc-exp?)))
```

这里的 `var-exp` , `var` , `bound-var` , `app-exp` , `rator` 以及 `rand` 分别为下述单词
的缩写 : **变量表达式 variable expression** , **变量 variable** , **绑定变量 bound variable** , 以及
**应用表达式 application expression** , **操作符 operator** 以及**操作数 operand** 。

该表示法声明了三种构造器 : `var-exp` , `lambda-exp` , `app-exp` 及一个谓词 `lc-exp?` 。
三个构造器用谓词 `identifier?` 和 `lc-exp?` 以实现对实参合法性的检查 , 故若 $\LcExp$ 
仅通过这几个构造器构建 , 我们就可确保其及其所有子表达式都是合法的 $\LcExp$ 。这允许
我们在处理 $\texttt{lambda}$ 演算表达式时跳过许多检查 。

相较于各种谓词和提词器 , 我们还是采用 `cases` 形式来判断数据类型的实例属于哪种变体
并提取出它们的组件 。为解释该形式 ,
我们将用 $\LcExp$ 数据类型重写[过程 `occurs-free?` ( 
代码在第 43 页](#S2.3-occurs-free) ) : 

```scheme {data-open=true}
;; occurs-free? : Sym × LcExp -> Bool
(define occurs-free? 
  (lambda (search-var exp)
    (cases lc-exp exp
           (var-exp (var) (eqv? var search-var))
           (lambda-exp (bound-var body)
                       (and
                        (not (eqv? search-var bound-var))
                        (occurs-free? search-var body)))
           (app-exp (rator rand)
                    (or
                     (occurs-free? search-var rator)
                     (occurs-free? search-var rand))))))
```

要明白这一切 , 需假设 `exp` 是个由 `app-exp` 构建的 $\texttt{lambda}$ 演算表达式 :  对于这样的
`exp` 的值 , 分支 `app-exp` 将被选中并且 `rator` 和 `rand` 则会被绑定到两个子表达式 ; 
紧接着表达式

```scheme
(or
 (occurs-free? search-var rator)
 (occurs-free? search-var rand))
```

将会被求值 , 就如同代码

```scheme
(if (app-exp? exp)
    (let ((rator (app-exp->rator exp))
          (rand (app-exp->rand exp)))
      (or
       (occurs-free? search-var rator)
       (occurs-free? search-var rand)))
    ...)
```

最后 `occurs-free?` 的递归调用也将以类似的方式完成计算 。

一般情况下 `define-datatype` 声明具有如下形式 :{{< raw >}}<a id="S2.4-define-datatype"></a>{{< /raw >}}

$\qquad 
\begin{aligned}[t]
\texttt{(define-datatype \textit{type-name} \textit{type-predicate-name}}\\
\texttt{$\{\texttt{(\textit{variant-name}
	$\{\texttt{(\textit{field-name ~predicate})}\}^*$
)}\}^+$)}
\end{aligned}$

这便定义了一种名为 $\textit{type-name}$ 的数据类型 , 它有一些**变体 variant** : 每个变体有变体名
以及 $0$ 或多个字段 , 每个字段都有相应的字段名及谓词 。任两个类型都不允许同名并且
任两个变体都不允许同名 —— 即便它们可能属于同一个类型 。类型名不可被用作变体名 。
每个字段的谓词必须是一个 Scheme 谓词 。

对每个变体都会有一个新的构造器被创建 , 其将用于创建该变体下的项 。如果一个变体
有 $n$ 个字段那么它的构造器将取 $n$ 个实参 , 将它们分别交给对应的谓词进行处理并返回
变体值 —— 值的第 $i$ 个字段为第 $i$ 个参数值 。

$\textit{type-predicate-name}$ 被绑定到一个谓词上 。该谓词判断实参的值是否属于相应的类型 。

**记录表 record** 为仅含有单种变体的数据类型 。为了区分这种只含单种变体的数据类型
我们将遵循下述惯例 : 若只有一个变体则以 $\texttt{a-}\textit{type-name}$ 或 $\texttt{an-}\textit{type-name}$ 命名构造器 ,
否则将以 $\textit{variant-name-type-name}$ 命名构造器 。

通过 `define-datatype` 生成的数据结构可能是互为递归的 , 如 [1.1 节提到的 $\SList$](/posts/5cce8df/#Def1.1.6) :

$\qquad\begin{aligned}[t]
\SList ::={}& \texttt{($\{\SExp\}^*$)} \\
\SExp  ::={}& \Symbol \mid \SList
\end{aligned}$

$\SList$ 中的数据可通过数据类型 `s-list` 如下表示：

```scheme
(define-datatype s-list s-list?
                 (empty-s-list)
                 (non-empty-s-list
                  (first s-exp?)
                  (rest  s-list?)))

(define-datatype s-exp s-exp?
                 (symbol-s-exp
                  (sym symbol?))
                 (s-list-s-exp
                  (slst s-list?)))
```

数据类型 `s-list` 通过 `(empty-s-list)` 和 `non-empty-s-list` 而非 `()` 和 `cons` 
来实现自己特有的列表表示方式 。如果我们依然想用 Scheme 列表表示 , 那么可以写成

```scheme
(define-datatype s-list s-list?
                 (an-s-list
                  (sexps (list-of s-exp?))))

(define list-of
  (lambda (pred)
    (lambda (val)
      (or (null? val)
          (and (pair? val)
               (pred (car val))
               ((list-of pred) (cdr val)))))))
```

这里 $\texttt{(list-of \textit{pred})}$ 将生成一个谓词 , 该谓词用来检查实参是否构成一个列表 , 并且
列表每个元素是否都满足 $\textit{pred}$ 。

`cases` 语法的一般形式为

$\qquad
\begin{aligned}[t]
&\texttt{(cases \textit{type-name} \textit{expression}}\\
&\texttt{\ \ \(\{\texttt{(}\textit{variant-name}\ \,\texttt{(}\{\textit{field-name}\}^*\texttt{)}\ \,\textit{consequent}\texttt{)}\}^*\)} \\
&\texttt{\ \ (else \textit{default}))}
\end{aligned}$

该形式声明类型 , 一个待求值待检查的表达式以及一些语句 ; 每个语句都以指定类型的
某一变体名以及相应字段名作为标识 。`else` 语句可省略 。首先 $\textit{expression}$ 会被求值 ,
如此便得到 $\textit{type-name}$ 的某个值 $v$ ; 如果 $v$ 对应于名为 $\textit{variant-name}$ 的变体 , 那么就
选中对应的语句 。各个 $\textit{field-name}$ 会被绑定到 $v$ 中对应的字段值 。此后 $\textit{consequent}$ 
的值会在这些绑定的作用域内求值并被返回 。若 $v$ 不属于任何变体且出现 `else` 语句
则求取并返回 $\textit{de\!fault}$ 的值 ; 若没有 `else` 语句则必须为指定数据类型的每一个变体都
指定一个语句 。

`cases` 形式根据位置绑定变量 : 第 $i$ 个变量绑定到第 $i$ 个字段 ; 因此我们就可以这样写

```scheme
(app-exp (exp1 exp2)
         (or
          (occurs-free? search-var exp1)
          (occurs-free? search-var exp2)))
```

而不是这样

```scheme
(app-exp (rator rand)
         (or
          (occurs-free? search-var rator)
          (occurs-free? search-var rand)))
```

`define-datatype` 和 `cases` 形式提供了一种简洁的方式来定义递归数据类型 , 但
但这种方式并不是唯一的 。其他应用场景可能需要更专门的表示方式 : 它们更紧凑且
更高效 , 可以利用上数据的特殊性质 。这些优势的代价是必须手动实现接口中的过程 。

`define-datatype` 形式是**领域特定语言 domain-specific language** 的其中一个例子 。
此种语言是一种小巧的语言 , 用于描述一系列小而明确的任务中的单一任务 。本例中
我们的任务是定义一种递归数据类型 。这种语言可能存在于用途更广泛的语言中 , 就 
如同 `define-datatype` 一样 ; 也有可能是独自构成一门单独的语言 , 别有一套工具 。
一般来说创造这类语言首先需找出任务的各种变体 , 然后再设计语言并描述这些变体 。
这种策略通常非常有效 。

{{< raw >}}<a id="Xer2.21"></a>{{< /raw >}}**练习 2.21** `[*]`\
请用 `define-datatype` 实现如 [2.2.2 节中提到的环境数据类型](#G2.1) ,
然后实现[练习 2.9 中的 `has-binding?`](#Xer2.9) 。

{{< raw >}}<a id="Xer2.22"></a>{{< /raw >}}**练习 2.22** `[*]`\
请用 `define-datatype` 实现[练习 2.4 中的栈数据类型](#Xer2.4) 。

{{< raw >}}<a id="Xer2.23"></a>{{< /raw >}}**练习 2.23** `[*]`\
`lc-exp` 的定义忽略了[定义 1.1.8](/posts/5cce8df/#Def1.1.8) 中的要求 : " $\Iden$ 表示任何非 $\texttt{lambda}$ 的符号 " ; 
修改 `identifier?` 的定义以满足这一需求 。提示 : 任何谓词 ( 包括你自己定义的 ) , 
都可以在 `define-datatype` 中使用 。

{{< raw >}}<a id="Xer2.24"></a>{{< /raw >}}**练习 2.24** `[*]`\
下面是用 `define-datatype` 表示的二叉树 : 

```scheme
(define-datatype bintree bintree?
                 (leaf-node
                  (num integer?))
                 (interior-node
                  (key symbol?)
                  (left bintree?)
                  (right bintree?)))
```

实现 `bintree-to-list` 以操作二叉树 , 如此 `(bintree-to-list (interior-node `
`'a (leaf-node 3) (leaf-node 4)))` 应返回列表

```scheme
(interior-node
 a
 (leaf-node 3)
 (leaf-node 4))
```

{{< raw >}}<a id="Xer2.25"></a>{{< /raw >}}**练习 2.25** `[**]`\
用 `cases` 写出 `max-interior` : 它以一个含至少一个非叶节点的整数二叉树 ( 如同上题 )
作为实参并返回叶节点标签之和最大的内部节点所对应的标签 。

```scheme
> (define tree-1 (interior-node 'foo (leaf-node 2) (leaf-node 3)))
> (define tree-2 (interior-node 'bar (leaf-node -1) tree-1))
> (define tree-3 (interior-node 'baz tree-2 (leaf-node 1)))
> (max-interior tree-2)
foo
> (max-interior tree-3)
baz
```

最后一次调用 `max-interior` 也可能返回 `foo`，因为 `foo` 和 `baz` 的叶节点之和都为 $5$ 。

{{< raw >}}<a id="Xer2.26"></a>{{< /raw >}}**练习 2.26** `[**]`\
下面是[练习 1.33](/posts/5cce8df/#Xer1.33) 的另一种版本 。考虑下述树的定义 : 

$\qquad\begin{aligned}[t]
\RedBlueTree    &::={} \RedBlueSubtree \\
\RedBlueSubtree &::={} \texttt{(red-node \RedBlueSubtree  \RedBlueSubtree)} \\
                &::={} \texttt{(blue-node $\{\RedBlueSubtree\}^*$)} \\
                &::={} \texttt{(leaf-node \Int)}
\end{aligned}$

用 `define-datatype` 写出等价的定义并用所得到的接口写一个过程 : 它取一棵树并生成
形状相同的另一棵树 , 但是把每片叶子的值改为从当前叶子节点到树根之间红色节点的数目 。

## 2.5 抽象语法及其表示 {#S2.5}

语法通常声明的是递归数据类型的某一具体表示 : 后者使用前者生成的字符串或值 。这样的
表示叫做**具体语法 concrete syntax** 或**外在 external** 表示 。

考虑[定义 1.1.8](/posts/5cce8df/#Def1.1.8) 提到的所有 $\texttt{lambda}$ 演算表达式构成的集 :  这里用的就是 $\texttt{lambda}$ 演算表达式
的具体语法 。我们也可以用其他具体的语法来表示 $\texttt{lambda}$ 演算表达式 , 比如可以用

$\qquad \begin{aligned}[t]
\LcExp   ::={}& 
\Iden \\ ::={}&
\texttt{(proc \Iden  => \LcExp)} \\::={}&
\texttt{\LcExp  (\LcExp)}
\end{aligned}$

来将 $\texttt{lambda}$ 演算表达式定义为另一种不同的字符串集合 。

为了处理这样的数据 , 我们需要将其转换为**内在 internal** 表示 。`define-datatype` 形式便提供了一种简洁的方式来定义这样的内在表示 ,
这样的表示称作 **abstract syntax** 。
抽象语法中像括号这样的终结符不需要被储存 , 因为它们不传递信息 。
另一方面我们想确保数据结构可允许我们
判断其所代表的 $\texttt{lambda}$ 演算表达式并提取出各个部分 。
[第 46 页提到的 `lc-exp`](#S2.4-lc-exp)
数据类型可允许我们轻松实现上述需求 。

将内在表示可视化为**抽象语法树 abstract syntax tree** 是很有效的 。[图 2.2](#G2.2) 是一棵抽象语法树 , 
对应于 `lc-exp` 类型下的 $\texttt{lambda}$ 演算表达式 `(lambda (x) (f (f x)))` 。抽象语法树中
每个非叶节点都被标注了对应的派生式的名称 , 每个枝干上都被标注了对应的非终结符的出现 , 
每个叶节点则对应于终结符 。

下方为图 2.2 —— `(lambda (x) (f (f x)))` 所对应的抽象语法树 。
$
%\qquad \begin{xy}0;p+/r1cm/:p+/d1cm/::
%(0,0)*+<10pt,0pt>[F-][=code]\texttt{lambda-exp}="t";
%"t";p+/r2cm/:p+/d2cm/::
%(-1,+1)*[code]{\texttt{x}\vphantom{\tt fg}}="t0";{\ar"t0"|{\texttt{bound-var}\vphantom{\tt fg}}};
%(+1,+1)*[code]{\texttt{app-exp}\vphantom{\tt fg}}="t1";{\ar"t1"|{\texttt{body}}};
%"t1";p+/r2cm/:p+/d2cm/::
%(-1,+1)*[code]{\texttt{var-exp}\vphantom{\tt fg}}="t10";{\ar"t10"|{\texttt{rator}}};
%(+1,+1)*[code]{\texttt{app-exp}\vphantom{\tt fg}}="t11";{\ar"t11"|{\texttt{rand}}};
%"t10";p+/r2cm/:p+/d2cm/::
%(+0,+1)*[code]{\texttt{f}\vphantom{\tt fg}}="t100";{\ar"t100"|{\texttt{var}}};
%"t11";p+/r2cm/:p+/d2cm/::
%(-1,+1)*[code]{\texttt{var-exp}\vphantom{\tt fg}}="t110";
%{\ar"t110"|{\texttt{rator}}};
%(+1,+1)*[code]{\texttt{var-exp}\vphantom{\tt fg}}="t111";{
%\ar"t111"|{\texttt{rand}}};
%"t110";p+/r2cm/:p+/d2cm/::
%(+0,+1)*[code]{\texttt{f}\vphantom{\tt fg}}="t1100";{\ar"t1100"|{\texttt{var}}};
%"t111";p+/r2cm/:p+/d2cm/::
%(+0,+1)*[code]{\texttt{x}\vphantom{\tt fg}}="t1110";{\ar"t1110"|{\texttt{var}}};
%\end{xy}
$ {{< raw >}}<a id="G2.2"></a>{{< /raw >}}

{{< svg-file src="EOPL/Gph2.2.svg" >}}

要为某种具体语法设计抽象语法 , 我们就必须为具体语法中的每个派生式及各个派生式中的
每个非终结符的出现都起一个名字 。使用 `define-datatype` 可以很轻松地声明抽象语法 ,
我们为每个非终结符都使用一次 `define-datatype` , 为每个派生式都添加一个变体 。

我们可以用下述记法精确总结概括[图 2.2](#G2.2) 中的结构 :{{< raw >}}<a id="S2.5-lc-exp"></a>{{< /raw >}}

$\qquad \begin{aligned}[t]
\LcExp ::={}& \Iden \\
            & \bbox[border:.4pt solid black]{\texttt{var-exp (var)}} \\[+3pt]
       ::={}& \texttt{(lambda (\Iden) \LcExp)} \\
            & \bbox[border:.4pt solid black]{\texttt{lambda-exp (bound-var body)}}\\[+3pt]
       ::={}& \texttt{(\LcExp  \LcExp)} \\
            & \bbox[border:.4pt solid black]{\texttt{app-exp (rator rand)}}
\end{aligned}$

这样的记法可同时指明具体语法和抽象语法 , 本书将会始终采用 。

既然已经区分了具体语法 ( 主要供人使用 ) 以及抽象语法 ( 主要供计算机使用 ) , 现在看看
如何在这两种语法之间进行转换 。

假如具体语法是个字符串集合 , 那么想要推导出对应的抽象语法树将会非常麻烦 。这样的
任务叫做**解析 parsing** , 其由**解析器 parser** 完成 。由于写解析器通常会很麻烦 , 因此最好
用一种叫**解析器生成器 parser generator** 的工具来实现此目标 ; 解析器生成器以一套语法
作为输入并输出一个解析器 。由于语法是交由工具处理的 , 它们必须以某种机器可识别的
方式来书写 。有很多现成的解析器生成器 。

如果具体语法以列表集合的形式给出 , 那么解析过程就可被大幅简化 。
比如将[本节开头的$\texttt{lambda}$ 演算表达式](#S2.5)声明为列表集合 ,
就如同前面 [第 47 页提到的 `define-datatype`](#S2.4-define-datatype) 一样 。
在这种情形下 Scheme 的 `read` 流程会自动把字符串解析为列表和符号 ,
如此将这些列表结构解析为抽象语法树就容易得多了 ,
正如下面的 `parse-expression` 一样 。

```scheme {data-open=true}
;; parse-expression : SchemeVal → LcExp
(define parse-expression
  (lambda (datum)
    (cond
     [(symbol? datum) (var-exp datum)]
     [(pair? datum)
      (if (eqv? (car datum) 'lambda)
          (lambda-exp
           (car (cadr datum))
           (parse-expression (caddr datum)))
          (app-exp
           (parse-expression (car datum))
           (parse-expression (cadr datum))))]
     [else (report-invalid-concrete-syntax datum)])))
```

通常我们很容易将抽象语法树重新转换为列表 - 符号表示 。如果这样做了那么 Scheme 的
打印流程就会将其展示为列表形式的具体语法 。这一切是通过 `unparse-lc-exp` 实现的 : 

```scheme {data-open=true}
;; unparse-lc-exp : LcExp → SchemeVal
(define unparse-lc-exp
  (lambda (exp)
    (cases lc-exp exp
           (var-exp (var) var)
           (lambda-exp (bound-var body)
                       (list 'lambda (list bound-var)
                             (unparse-lc-exp body)))
           (app-exp (rator rand)
                    (list
                     (unparse-lc-exp rator) (unparse-lc-exp rand))))))
```

{{< raw >}}<a id="Xer2.27"></a>{{< /raw >}}**练习 2.27** `[*]`\
画出下面 lambda 演算表达式的抽象语法树：

```scheme
((lambda (a) (a b)) c)

(lambda (x)
  (lambda (y)
    ((lambda (x)
       (x y))
     x)))
```

{{< raw >}}<a id="Xer2.28"></a>{{< /raw >}}**练习 2.28** `[*]`\
写出逆向解析器以将 `lc-exp` 的抽象语法转换为符合[本节第二个语法 ( 第 52 页 )](#S2.5-lc-exp) 的字符串 。

{{< raw >}}<a id="Xer2.29"></a>{{< /raw >}}**练习 2.29** `[*]`\
若[克莱尼星号或加号 ( 第 7 页 )](/posts/5cce8df/#S1.1.2-Kleene) 被用于具体语法中 , 构建抽象语法树时最好是使用相应子树
构成的列表 。例如，如果 $\texttt{lambda}$ 演算表达式的语法为

$\qquad \qquad \begin{aligned}[t]
\LcExp ::={}& \Iden \\
            & \bbox[border:.4pt solid black]{\texttt{var-exp (var)}} \\[+3pt]
       ::={}& \texttt{(lambda \(\{(\Iden)^*\}\) \LcExp )} \\
            & \bbox[border:.4pt solid black]{\texttt{lambda-exp (bound-var body)}}\\[+3pt]
       ::={}& \texttt{(\LcExp  \(\{\LcExp\}^*\))} \\
            & \bbox[border:.4pt solid black]{\texttt{app-exp (rator rand)}}
\end{aligned}$

那么字段 `bound-vars` 对应的谓词就可以写作是 `(list-of identifier?)` ,
而 `rands` 对应的谓词则可写作 `(list-of lc-exp?)` 。
以此方式写出该语法的 `define-datatype` 和解析器 。

{{< raw >}}<a id="Xer2.30"></a>{{< /raw >}}**练习 2.30** `[**]`\
刚才定义的 `parse-expression` 过程不是很靠谱 :
它检查不到某些可能的语法错误 —— 如 `(a b c)` ;
并且会在其他表达式终止时给不出恰当的错误信息 —— 如 `(lambda)` 。
请修改它以使之更稳健 , 可以接收任何 $\SExp$ ,
并且可对不表示 $\texttt{lambda}$ 演算表达式的 $\SExp$ 给出恰当的错误信息 。

{{< raw >}}<a id="Xer2.31"></a>{{< /raw >}}**练习 2.31** `[**]`\
有时把具体语法定义为被括号包围的符号和整数序列会很方便 。比如我们可考虑把**前缀列表 prefix list** 定义为

$\qquad \begin{aligned}[t]
\PrefixList ::={}& \texttt{(\PrefixList)}  \\
\PrefixList ::={}& \Int \\
            ::={}& \texttt{- \PrefixList  \PrefixList}
\end{aligned}$

如此 `(- - 3 2 - 4 - 12 7)` 是一个合法的前缀列表 。该表示法也称作**波兰前缀表示法 Polish prefix notation** 以纪念其发明者Jan Łukasiewicz 。写一个解析器以将前缀列表表示法
转换为下述抽象语法 : 

```scheme
(define-datatype prefix-exp prefix-exp?
                 (const-exp
                  (num integer?))
                 (diff-exp
                  (operand1 prefix-exp?)
                  (operand2 prefix-exp?)))
```

使上例与这几个构造器能生成相同抽象语法树 : 

```scheme
(diff-exp
 (diff-exp
  (const-exp 3)
  (const-exp 2))
 (diff-exp
  (const-exp 4)
  (diff-exp
   (const-exp 12)
   (const-exp 7))))
```

提示 : 想想如何写一个过程 : 其取一列表并产生一个 `prefix-exp` 和列表剩余元素所组成的列表 。