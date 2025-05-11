---
title: 总结-DaoFP-范畴论
subtitle:
date: 2025-04-18T21:50:28+12:00
slug: 24198cb
draft: false
author:
  name:
  link:
  email:
  avatar:
description:
keywords:
license:
comment: true
weight: DaoFP1000
tags: 
 - 总结
 - DaoFP
 - 函数式编程
 - Haskell
collections:
 - 翻译-DaoFP
linkToMarkdown: false
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
password:
message:
repost:
  enable: true
  url:
---

$$
{\rm \LaTeX}\text{ Definitions are here.}
\let\ph=\phantom
\let\hph=\hphantom
\let\vph=\vphantom
\let\mrel=\mathrel
\let\mbin=\mathbin
\let\mllap=\mathllap
\let\mrlap=\mathrlap
\let\mclap=\mathclap
\def\verts#1{\lvert#1\rvert}
\def\pila{\vph{fg}}                   % 柱子 , 用于指定盒子的最小高度
% ------------------------------------------------------------------
% 类型运算
\def\cons{\mrel{.}}                   % 对应积类型 concatenation 
\def\ethr{\mrel{|}}                   % 对应和类型 either
\def\etc{{\rm etc}}
% ------------------------------------------------------------------
% + 的值构造器
\def\Lft{{\tt Lft}}                   % 对象 + 构造器 Left
\def\Rht{{\tt Rht}}	                  % 对象 + 构造器 Right
% × 的接口
\def\fst{{\tt fst}}                   % 对象 × - first
\def\snd{{\tt snd}}                   % × - second
% N 的值构造器
\def\zero{0}                          % N - zero
\def\succ{{\tt succ}}                 % N - successor
% ------------------------------------------------------------------
% 打印方法名
\def\id{\textrm{id}}                           % 对象的 id
\def\absurd{{\rm\large\textexclamdown}\hspace{-3px}} % 对象的 absurd
\def\bang{{\rm\small !}}                       % 对象的 bang
\def\dom{\textrm{dom}}                         % 箭头的定义域
\def\img{\textrm{img}}                         % 箭头的值域
\def\src{\textrm{src}}                         % 箭头的源头
\def\tar{\textrm{tar}}                         % 箭头的目标
\def\ID{\textrm{ID}}                           % 范畴的 ID
\def\Obj{\textrm{Obj}}                         % 范畴的对象
\def\Arr{\textrm{Arr}}                         % 范畴的箭头
\def\circH{\mbin{\overline\circ}}              % 自然变换的横复合
\def\circV{\mbin{\circ}}                       % 自然变换的纵复合
% 打印常/变量名
\newcommand{\obj}[1][c]{{\sf #1}}              % 对象
\def\obji{\obj[0]}                             % 对象 : 始
\def\objt{\obj[1]}                             % 对象 : 终
\newcommand{\cat}[1][C]{\mathcal{#1}}          % 范畴 : C(默认)
\def\catCat{\cat[Cat]}                         % 范畴 : Cat
\def\catSet{\cat[Set]}                         % 范畴 : Set
\def\catHask{\cat[Hask]}                       % 范畴 : Hask — 笛卡尔闭范畴 (+/× 都在里面)
\newcommand{\catcirc}[1][\cat]{                % 范畴 C 中的复合运算
  \catbfunc[#1]{\circ}}
\newcommand{\catufunc}[2][\cat]{{              % 范畴 C 中的函子 : 一元
  \overset{\mclap{\small\pila #1}}{#2}}}
\newcommand{\catbfunc}[2][\cat]{\mbin{{        % 范畴 C 中的函子 : 二元
  \overset{\mclap{\small\pila #1}}{#2}}}}
\newcommand{\catdiag}[1][\cat]{                % 范畴 C 中的函子 : 对角
  \catufunc[#1]{\Delta}}
\newcommand{\cathom}[1][\cat]{\mbin{{          % 范畴 C 中的函子 : Hom
  \xrightarrow{\small #1}}}}
\newcommand{\catplus}[1][\cat]{                % 范畴 C 中的函子 : +
  \catbfunc[#1]{+}}
\newcommand{\cattimes}[1][\cat]{               % 范畴 C 中的函子 : ×
  \catbfunc[#1]{\times}}
\newcommand{\catotimes}[1][\cat]{              % 范畴 C 中的张量积
  \catbfunc[#1]{\otimes}}
\newcommand{\list}[1]{                         % 范畴 Hask 中的函子 List
	\texttt{[\(#1\)]}}
\def\nil{{\tt []}}                             % 范畴 Hask 中的函子 List 的值构造器 nil
\def\concat{\mbin{:}}                          % 范畴 Hask 中的函子 List 的值构造器 concat
\def\yoneda{{\Large\raise{-1px}{               % 米田嵌入
  \hspace{-1px}\smash{よ}\vph{(fgh)}\hspace{-1px}}}}
\def\yoda{                                     % 尤达嵌入
  \texttt{尤}}
\def\limit#1{{\rm lim}}                        % 极限
\def\colimit#1{{\rm colim}}                    % 余极限
% 用方法求值
\def\getid#1{{}_{:#1}\id}                      % 获取对象的 id
\def\getabsurd#1{{}_{:#1}\absurd}              % 获取对象的 absurd
\def\getbang#1{{}_{:#1}\bang}                  % 获取对象的 bang
\def\getdom#1{#1\textrm{ dom}}                 % 获取箭头的定义域
\def\getimg#1{#1\textrm{ img}}                 % 获取箭头的值域
\def\getsrc#1{#1~\src}                         % 获取箭头的源头
\def\gettar#1{#1~\tar}                         % 获取箭头的目标
\def\getrst#1undr#2{{}^{\vph{fg}}_{:#2}{#1}}   % 获取箭头在对象下的限制 
\def\getID#1{{}_{:#1}\ID}                      % 获取范畴的 ID
\def\getObj#1{#1~\Obj}                         % 获取范畴的对象
\def\getArr#1{#1~\Arr}                         % 获取范畴的箭头
\def\getop#1{#1^{\rm op}}                      % 获取范畴的对偶
\def\getlimit#1{#1\textrm{ lim}}               % 获取极限
\def\getcolimit#1{#1\textrm{ colim}}           % 获取余极限
$$

<!-- ## 章节 00

与普通数学教材有所不同的是 , 我们将采用同\
Python / Java 里 `<对象>.<方法>` 的记法 。例 :

| 记法名称 | 普通记法  | 本书记法 |
| -- | -- | -- |
| 范畴 $\cat$ 的所有箭头 | $\Arr~\cat$ | $\getArr\cat$ |
| 范畴 $\cat$ 的所有对象 | $\Obj~\cat$ | $\getObj\cat$ |
| 映射 $f$ 应用于 $x$ | $f(x)$   | $xf$ |
| 映射 $f$ 的定义域 | $\dom~f$ | $\getdom f$ |
| 映射 $f$ 的值域 | $\img~f$ | $\getimg f$ |
| 映射 $f$ 在对象 $\obj[x]$ 下的限制 | $f\!\upharpoonright_\obj[x]$ | $\getrst{f}undr{\obj[x]}$ |
| 对象 $\obj[x]$ 的恒等映射 | $\id_{\obj[x]}$ | $\getid{\obj[x]}$ |
| 元素 $x$ , $y$ 构成的有序对 | $(x,y)$  | $(x \cons y)$ | -->

{{< raw >}}<div style="page-break-after: always"></div>{{< /raw >}}

## 章节 01 - 03

范畴由对象以及对象间的箭头构成 。本文将会\
着重分析余积闭范畴 $\cat$ 。 现在先给出下述定义 :

- **$\obji$ 为始对象**当且仅当对任意 $\cat$ 中对象 $\obj$\
都有且仅有唯一的箭头 $\getabsurd{\obj}: \obji\cathom \obj$

- **$\objt$ 为终对象**当且仅当对任意 $\cat$ 中对象 $\obj$\
都有且仅有唯一的箭头 $\getbang{\obj}: \obj \cathom \objt$

> [!note]
>
> 其他范畴中始 / 终对象未必存在 。

我们假设范畴 $\cat$ 中含有始对象 $\obj[0]$ 以及终对象 $\obj[1]$ 。\
因此根据上面的的信息我们不难得出下述内容 :

- 形如 $\obji\cathom\obji$ 的箭头\
仅有一个 , 即 $\getid\obji$

- 形如 $\objt\cathom\objt$ 的箭头\
仅有一个 , 即 $\getid\objt$

对任意 $\cat$ 中对象 $\obj[a]$ , $\obj[a_1]$ , $\obj[a_2]$ , $\etc$ , $\obj[b]$ , $\obj[b_1]$ , $\obj[b_2]$ , $\etc$ \
及任意 $\cat$ 中映射 $\phi$ 我们规定下述性质始终成立 : 。

- $\phi$ 为 $\obj[b]$ 的**元素**当且仅当\
$\phi: \obj[a]\cathom\obj[b]$

- $\phi$ 为 $\obj[a]$ 的**全局元素**当且仅当\
$\phi: \objt\cathom\obj[a]$

- $\phi$ 不存在可由\
$\phi: \obj[b]\cathom \obji$ 得出

> [!note]
>
> 其他范畴中上一条断言未必成立 。

另外我们规定

- $\obj[a]\cathom\obj[b]$ 为所有\
从 $\obj[a]$ 射向 $\obj[b]$ 的箭头构成的集

> [!note]
>
> 上述断言仅对**局部小范畴成立**\
> 其他范畴里 $\obj[a]\cathom\obj[b]$ 未必为集合 。
  
{{< raw >}}<div style="page-break-after: always"></div>{{< /raw >}}

范畴 $\cat$ 中某些特定的箭头可进行复合运算 : 

- 对任意 $\cat$ 中对象 $\obj[c_1]$ , $\obj[c_2]$ , $\obj[c_3]$ 我们都会有\
$\begin{aligned}[t]
\cathom &: (\obj[c_1]\cathom\obj[c_2])\cattimes[\catSet](\obj[c_2]\cathom\obj[c_3])\cathom[\catSet](\obj[c_1]\cathom\obj[c_3]) \\
\cathom &: (\quad~\phi_1~\quad\,\cons\,\quad~\phi_2~\quad) \longmapsto ~\phi_1\catcirc \phi_2
\end{aligned}$

假如现在还知道箭头 $f_1$ , $\phi_1$ , $g_1$ 分别属于\
$\obj[a_2]\cathom\obj[a_1]$ , $\obj[a_1]\cathom\obj[b_1]$ , $\obj[b_1]\cathom\obj[b_2]$ , 那么便可得知

- $(f_1\catcirc\phi_1)\catcirc g_1 = f_1\catcirc(\phi_1\catcirc g_1)$\
说明箭头复合运算有结合律

另外如果固定住 $\catcirc$ 的单侧实参我们便可获得\
两个新的函数 : 

$$
\begin{array}{ll}
\getabsurd{\obj}A  \\
\getbang{\obj}A
\end{array}
$$